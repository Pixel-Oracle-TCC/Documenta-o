# 🗃️ Schema do Banco de Dados

Schema do Pixel Oracle em PostgreSQL + `pgvector`, alinhado ao pipeline de evidências de reviews (coleta → ABSA → temas → métricas → drivers).

## 🧩 Entidades
- **`games`** — metadados Steam (`steam_appid`, título, descrição, `preco_brl`, `release_year`, `release_date`, totais e `review_score` / `review_score_desc`), `rank_popularidade` ([[Catalogo pelos Mais Vendidos]]), `na_amostra` (flag da amostra estratificada), `embedding VECTOR(384)`, `reviews_collected_at`.
- **`apps_ignorados`** — apps do ranking que não entram no catálogo (software, DLC, sem página na Store BR).
- **`genres` / `game_genres`** — N:N jogos ↔ gêneros.
- **`categories` / `game_categories`** — N:N com `steam_category_id` (single-player, co-op, conquistas…).
- **`developers` / `game_developers`**, **`publishers` / `game_publishers`** — estúdios (entidades comerciais, não pessoas).
- **`languages` / `game_languages`** — idiomas suportados (HTML da Store limpo).
- **`reviews`** — texto + metadados (`steam_purchase`, `received_for_free`, `timestamp_created`, `playtime_at_review`…); dedup por `recommendationid`; **sem coluna de autor**. `fatias TEXT[]` acumula pertença (`proporcional`, `util_positiva`, `util_negativa`) — ver [[Coleta de Reviews]].
- **`review_snapshots`** — totais oficiais no tempo + `*_unfiltered` (sem filtro offtopic), para review bombing.
- **`review_aspects`**, **`themes`**, **`theme_stats`**, **`genre_drivers`**, **`absa_gold_standard`** — ver [[Extração de Aspectos (ABSA)]], [[Temas Recorrentes e Metricas]], [[Drivers de Recomendacao]].

## 🔍 Busca vetorial
- Índice `ivfflat` em `games.embedding` com `vector_cosine_ops`; modelo `all-MiniLM-L6-v2` — ver [[Retrieval e Busca Vetorial]].
- Aspectos usam `paraphrase-multilingual-MiniLM-L12-v2` (também 384 dims).

## 📁 Fonte no repositório
- Migrations `001`–`007` em `ml-service/db/migrations/` (`006` metadados/amostra/snapshots; `007` fatias, `received_for_free`, elegível métricas).
- Lista coluna a coluna (para inspecionar na extensão): [[PostgreSQL no Cursor e VS Code]].
- Aplicação: automática no primeiro `docker compose up` (volume vazio) ou `python scripts/apply_schema.py`.
- Não há coluna de vendas nem de donos estimados (ver [[Mudanca de Foco do Projeto]]).

---
*Relacionado:* [[MongoDB para PostgreSQL pgvector]], [[Conformidade Steam e LGPD]], [[Coleta de Reviews]], [[Catalogo pelos Mais Vendidos]], [[Popular e Verificar o Banco]], [[PostgreSQL no Cursor e VS Code]]
