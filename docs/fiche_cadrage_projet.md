# Fiche de Cadrage d'Ingénierie — Projet Fédérateur AI-BD_BI
## Spécification du Moteur RAG / Data Intelligent

---

### 1. Fiche d'Identité du Projet

- **Nom du Projet :** `FinRAG-Insight`
- **Membres de l'Équipe :**
  1. Msalmi Ines — Rôle : Lead Data / Retrieval (ingestion, chunking, embeddings, Qdrant, hybrid search, reranker, évaluation Ragas)
  2. Chiheb Aouadi — Rôle : Lead Backend / Frontend / Infra (FastAPI, SSE, interface React, Docker, CI/CD)
- **Lien du Dépôt GitHub :** `https://github.com/<ton-compte>/finrag-insight`
- **Branche de travail principale :** `main` (protégée)

---

### 2. Cadrage du Problème Métier ("Problem-First")

#### 2.1 Persona Cible & Contexte Opérationnel
- **Utilisateur visé :** Analyste financier buy-side dans un fonds d'investissement, qui compare plusieurs sociétés cotées du CAC 40 pendant la saison des résultats.
- **Point de douleur majeur (*Pain point*) :** Pour chaque société, il doit parcourir un document d'enregistrement universel de 300 à 400 pages afin de retrouver l'endettement net, les engagements hors-bilan, les covenants et les risques déclarés. Comparer 5 sociétés représente plusieurs heures de lecture manuelle, sans garantie de citer la bonne page.
- **Limite des approches classiques (Pourquoi l'IA générative ?) :**
  - Synonymes financiers : « dette nette », « endettement financier net » et « net debt » désignent la même notion, Ctrl+F ou Elasticsearch seul les manque.
  - Tableaux multi-lignes : le chiffre est séparé de son libellé et de son exercice, ce qui casse le contexte.
  - Questions sémantiques croisées (ex. « Quels risques ont augmenté entre 2023 et 2024 ? ») : impossibles par mots-clés.
  - Besoin de traçabilité : chaque réponse doit citer le document et la page source.

#### 2.2 Proposition de Valeur
Fournir une interface de recherche augmentée en streaming, capable d'extraire, synthétiser et citer avec précision de page les engagements financiers des sociétés du corpus, avec un score de fidélité Ragas supérieur ou égal à 0.85.

---

### 3. Données & Volumétrie

- **Format des données sources :** `[x] PDF multi-colonnes` `[ ] JSON/CSV` `[ ] Tables relationnelles SQL` `[ ] Markdown` `[ ] Images/Scans`
- **Source :** Documents d'enregistrement universels (DEU/URD) publics, en français, téléchargés sur les sites « Investisseurs » des sociétés.
- **Volumétrie initiale (PoC S1-S3) :** 8 à 10 sociétés x 2 à 3 exercices, soit environ 20 à 30 documents / ~25 000 chunks.
- **Volumétrie cible en production :** 10 000 documents / 2 000 000 chunks.
- **Fréquence de mise à jour des données :** `[ ] Batch quotidien` `[ ] Temps réel / Streaming` `[x] Statique` (ré-ingestion en batch à chaque nouvelle publication annuelle)
- **Golden Dataset :** 30 à 50 questions-réponses annotées (question, réponse attendue, document et page de référence), couvrant questions factuelles (tableaux), textuelles et multi-sources.

---

### 4. Architecture Technique & Composants RAG

- **Stratégie de Chunking :** `[ ] Recursive Character` `[ ] Semantic Chunking` `[x] Parent-Document / Hierarchical`
  - *Taille de chunk visée :* 512 tokens | *Overlap :* 64 tokens
  - Les tableaux sont conservés en un seul chunk avec leur titre et leur exercice, pour éviter de séparer chiffres et libellés.
- **Modèle d'Embeddings :** `BAAI/bge-m3` (Dimension : 1024), multilingue, adapté au français.
- **Recherche hybride :** vecteurs denses + vecteurs sparse BM25, fusionnés par RRF (Reciprocal Rank Fusion). Top-20 initial.
- **Base Vectorielle retenue :** `[x] Qdrant` `[ ] Milvus` `[ ] pgvector`
  - *Justification du choix :* Qdrant gère nativement les vecteurs denses et sparse dans la même collection (hybrid search), filtre efficacement par métadonnées (société, exercice, page), propose un index HNSW et se lance en une commande via Docker Compose.
- **Reranker (Cross-Encoder) :** `BAAI/bge-reranker-base`, qui réduit le Top-20 au Top-5.
- **LLM de Génération :** LLM via API (Mistral ou Claude), derrière une interface abstraite pour pouvoir changer de modèle. Un LLM local type vLLM est écarté (pas de GPU garanti).
- **Backend / Frontend :** FastAPI async + Pydantic v2, streaming SSE, interface React (Next.js), PostgreSQL pour l'historique.

---

### 5. Métriques Quantitatives & SLAs

| Métrique | Définition / Outil | Objectif Cible (KPI) |
| :--- | :--- | :---: |
| **Recall@10 (Retrieval)** | Présence du document de vérité dans le top-10 | $\ge 85\%$ |
| **Recall@5 (après reranking)** | Présence du document de vérité dans le top-5 | $\ge 80\%$ |
| **Faithfulness (Ragas)** | Absence d'hallucinations non sourcées | $\ge 0.85$ / 1.00 |
| **Answer Relevance (Ragas)** | Adéquation directe à la question posée | $\ge 0.80$ / 1.00 |
| **Latence First-Token (TTFT)** | Temps d'affichage du premier token en streaming | $< 800$ ms |
| **Latence E2E P95** | Temps total de complétion de la requête | $< 3.5$ s |

Toutes les mesures sont calculées sur le Golden Dataset (aucune affirmation « à l'œil »).

---

### 6. Gestion des Risques & Garde-fous

- **Prévention des Hallucinations :** Injection systématique de la directive : *"Si l'information n'est pas présente dans les extraits ci-dessus, répondez explicitement que le corpus ne permet pas de conclure."* Citation obligatoire du document et de la page.
- **Erreurs sur les chiffres (tableaux mal extraits) :** Vérification manuelle d'un échantillon de tableaux après ingestion, conservation de l'exercice et de l'unité dans les métadonnées.
- **Protection des Secrets :** Utilisation exclusive de variables d'environnement via `.env` (aucun token dans le code source, `.env` dans le `.gitignore`).
- **Isolation Réseau :** Base vectorielle et base PostgreSQL confinées dans un réseau Docker interne privé.
- **Latence :** Le reranker tournant sur CPU peut menacer le SLA de 800 ms ; la latence sera mesurée par étape (embedding, recherche, reranking, LLM) dès la S3.
