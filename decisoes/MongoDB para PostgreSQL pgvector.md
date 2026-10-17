# 🗄️ MongoDB → PostgreSQL + pgvector

Por que o Pixel Oracle usa **PostgreSQL + pgvector** como único banco, com alternativas avaliadas e âncoras no código.

## 🎯 Contexto
O modelo de dados é relacional: jogos ↔ gêneros (N:N), reviews → jogos, aspectos → reviews, aspectos → temas, métricas por (gênero, tema), drivers por gênero. Além disso, jogos e aspectos têm embeddings para busca vetorial. Ver [[Schema do Banco de Dados]].

## ✅ Escolha
- **PostgreSQL** como banco relacional principal.
- Extensão **`pgvector`** para embeddings (`VECTOR(384)`) e k-NN por cosseno.
- Índice **`ivfflat`** em `games.embedding` (após backfill) com `vector_cosine_ops`.
- Imagem Docker: `pgvector/pgvector:pg16` no `docker-compose.yml` (serviço `db`).

## ❌ Alternativas descartadas
- **MongoDB Atlas Vector Search** — modelo de documentos não acomoda bem as relações N:N; implementação anterior estava instável.
- **MySQL VECTOR** — ecossistema/maturidade menos conveniente no prazo do TCC.

## 📌 Pontos principais no código
| Peça | Path |
|------|------|
| Schema SQL | `ml-service/db/migrations/001_schema.sql`, `002_reviews_focus.sql` |
| Engine/session | `ml-service/db/connection.py` (`DATABASE_URL`, `init_engine`, `session_scope`) |
| ORM | `ml-service/models/orm.py` |
| Query vetorial | `ml-service/retrieval/similar_games.py` |
| Compose | `docker-compose.yml` → monta `db/migrations/` em `docker-entrypoint-initdb.d` |
| Seed | `scripts/bootstrap_games_from_json.py` (UPSERT por `steam_appid`) |

## 🛠️ Implicações para o squad
- A [[API Principal Node]] **não** acessa o banco: só HTTP no ml-service ([[Por que dois servicos e um banco]]).
- Scripts de coleta/processamento são **idempotentes** (UPSERT, `absa_processed_at`, `reviews_collected_at`).
- Detalhes de embedding: [[Por que MiniLM e pgvector]].

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Schema do Banco de Dados]], [[Retrieval e Busca Vetorial]], [[Arquitetura de Microsservicos]], [[Por que MiniLM e pgvector]]
