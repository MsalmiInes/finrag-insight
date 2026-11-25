# FinRAG-Insight

Moteur RAG hybride pour analystes financiers : interrogation en langage naturel de documents d'enregistrement universels (DEU) du CAC 40, avec réponses en streaming et citations de pages.

## Problème métier

Un analyste buy-side doit parcourir des rapports de 300 à 400 pages pour retrouver endettement net, engagements hors-bilan et risques. Ctrl+F échoue face aux synonymes financiers et aux tableaux multi-lignes. FinRAG-Insight extrait, synthétise et cite la page source.

## Architecture (vue initiale)

Client Web (React) -> FastAPI (SSE) -> Embeddings bge-m3 -> Qdrant (dense + sparse, RRF) -> Reranker bge-reranker-base -> LLM -> Réponse streamée avec citations. PostgreSQL pour l'historique, Langfuse pour le tracing.

## Équipe

| Membre | Rôle |
| :--- | :--- |
| Msalmi Ines | Lead Data / Retrieval |
| Chiheb Aouadi | Lead Backend / Frontend / Infra |

## Documentation

- [Fiche de cadrage](docs/fiche_cadrage_projet.md)
- [Backlog MoSCoW](docs/matrice_moscow_backlog.md)

## Démarrage rapide

```bash
python3 -m venv .venv && source .venv/bin/activate
cp .env.example .env   # puis renseigner les valeurs
docker compose up -d   # (à venir en S2)
```

## Corpus

Sources et liste des DEU utilisés : voir `data/raw/README.md` (à compléter avant la S2).
