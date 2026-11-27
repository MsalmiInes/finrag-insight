# Backlog Technique MoSCoW — FinRAG-Insight
## User Stories & Critères d'Acceptation

---

## 1. MUST HAVE (Indispensables pour la Livraison Finale S7)

| ID | User Story Technique | Critères d'Acceptation (DoD) | Séance | Responsable |
| :-: | :--- | :--- | :-: | :-: |
| **US-01** | **Ingestion & Chunking des DEU**<br>En tant que Data Engineer, je veux découper les PDF financiers en chunks avec métadonnées pour un indexage précis. | - Script Python asynchrone avec gestion des erreurs.<br>- Métadonnées : `doc_id`, `societe`, `exercice`, `page_number`, `chunk_id`.<br>- Tableaux conservés en un seul chunk (titre + exercice + unité). | S2 | Ines |
| **US-02** | **Indexation Vectorielle Qdrant (HNSW)**<br>En tant que système RAG, je veux insérer vecteurs denses et sparse dans Qdrant pour accélérer la recherche. | - Qdrant démarré via Docker Compose.<br>- Index HNSW, distance Cosine.<br>- Insertion < 50 ms par batch de 64 chunks. | S2 | Ines |
| **US-03** | **Retrieval Hybride & Reranking**<br>En tant qu'analyste, je veux fusionner dense + BM25 (RRF) puis reranker pour maximiser la pertinence. | - Top-20 hybride, puis Top-5 via `bge-reranker-base`.<br>- Recall@5 ≥ 80 % sur le Golden Dataset. | S3 | Ines |
| **US-04** | **API REST FastAPI & Pydantic v2**<br>En tant qu'application cliente, je veux interroger l'API via des endpoints typés et validés. | - Schémas `QueryRequest` / `QueryResponse` stricts.<br>- HTTP 422 si question vide ou hors limites.<br>- Swagger/OpenAPI généré. | S4 | Chiheb |
| **US-05** | **Réponse en Streaming SSE avec citations**<br>En tant qu'analyste, je veux la réponse mot par mot avec société, exercice et page citées. | - Endpoint `/api/v1/chat/stream` (SSE).<br>- Payload avec chunks sources et scores.<br>- TTFT < 800 ms. | S4/S5 | Chiheb + Semah |
| **US-06** | **Interface Web React avec Source Drawer**<br>En tant qu'analyste, je veux voir la réponse en streaming et inspecter le texte exact des sources. | - États idle / loading / error gérés.<br>- Tiroir affichant le chunk cité et sa page.<br>- Aucun rechargement de page. | S5 | Semah |
| **US-07** | **Qualité & CI/CD**<br>En tant que Lead Developer, je veux que chaque commit lance tests et linter. | - 4 tests Pytest (mocks LLM et Qdrant).<br>- Ruff et GitHub Actions verts avant tout merge.<br>- Couverture ≥ 80 %. | S6 | Semah |
| **US-08** | **Conteneurisation de Production**<br>En tant qu'ingénieur DevOps, je veux déployer l'ensemble via Docker Compose sécurisé. | - Dockerfile multi-stage, utilisateur non-root.<br>- Secrets uniquement via variables d'environnement.<br>- Healthchecks sur chaque service. | S7 | Semah |
| **US-09** | **Golden Dataset d'évaluation**<br>En tant que Data Scientist, je veux un jeu de questions-réponses de référence pour mesurer le système. | - 30 à 50 Q/R avec document et page de référence.<br>- Mélange factuel (tableaux), textuel, multi-sources. | S3 | Ines |

---

## 2. SHOULD HAVE

| ID | User Story Technique | Critères d'Acceptation | Séance |
| :-: | :--- | :--- | :-: |
| **US-10** | **Évaluation continue Ragas**<br>Calcul automatique de Faithfulness et Answer Relevance. | - Pipeline exécutable en une commande.<br>- Faithfulness ≥ 0.85, Answer Relevance ≥ 0.80. | S3 |
| **US-11** | **Tracing Langfuse / OpenTelemetry**<br>Tracer chaque étape de la chaîne RAG. | - Latence par étape (embedding, recherche, rerank, LLM).<br>- Tokens consommés et coût estimé. | S3/S4 |
| **US-12** | **Cache sémantique Redis**<br>Réutiliser les réponses à des questions proches. | - Similarité cosinus > 0.95 servie en < 50 ms.<br>- Invalidation configurable. | S4 |

---

## 3. COULD HAVE

| ID | User Story Technique | Critères d'Acceptation |
| :-: | :--- | :--- |
| **US-13** | **Manifestes Kubernetes** | Deployments, Services, ConfigMaps, Ingress et HPA. |
| **US-14** | **Support multimodal (graphiques de rapports)** | Ingestion de graphiques et schémas via un modèle de vision. |

---

## 4. WON'T HAVE (Exclus du périmètre)

- Entraînement ou fine-tuning d'un LLM from scratch.
- Facturation SaaS (Stripe) et gestion d'abonnements.
- Application mobile native (le web responsive suffit).
- Conseil en investissement : l'outil extrait et cite, il ne recommande pas d'achat ou de vente.
