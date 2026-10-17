# 🔍 Retrieval e Busca Vetorial

Busca de jogos similares ao pitch via embeddings e `pgvector`. Os similares definem o escopo mais específico do dashboard: temas das reviews deles, reviews de referência e contexto de nota na Steam ([[Motor de Triangulacao]]).

## 🧠 Modelo
- `sentence-transformers` / `all-MiniLM-L6-v2` (384 dimensões).
- Texto indexado: título + descrição + gêneros.
- Decisão completa: [[Por que MiniLM e pgvector]].

## 🛠️ Operação
- Backfill: `scripts/backfill_embeddings.py`.
- Query: `ORDER BY embedding <=> :query LIMIT k` em `retrieval/similar_games.py`; similaridade = 1 − distância.
- Cada similar volta com `review_score_desc`, `total_reviews_steam` e `pct_positivas` (resumo real da Steam salvo na [[Coleta de Reviews]]).
- Endpoints: `POST /retrieve-similar` (debug) e uso interno em `POST /analyze`.
- Índice `ivfflat` criado **após** popular embeddings.

---
*Relacionado:* [[Schema do Banco de Dados]], [[Motor de Triangulacao]], [[Por que MiniLM e pgvector]], [[Coleta de Reviews]], [[Catalogo de Decisoes Tecnologicas]]
