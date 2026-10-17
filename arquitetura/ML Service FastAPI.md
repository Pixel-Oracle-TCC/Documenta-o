# 🐍 ML Service FastAPI

Documentação da pasta `ml-service/` — busca de similares, pipeline offline de reviews e montagem do dashboard de evidências. Entrada do squad: [[Guia Ponta a Ponta da API]].

## 🎯 Papel no sistema
- Serviço interno (`:8000`) consumido pela [[API Principal Node]].
- Concentra: embeddings + pgvector, coleta Steam (metadados, top sellers, reviews), ABSA, temas, métricas de recorrência, RF de drivers e o [[Motor de Triangulacao]].
- Único serviço que fala com o **PostgreSQL + pgvector**.
- Não carrega modelo no boot: os drivers do RF ficam pré-calculados no banco.

## 🏗️ Organização

```
ml-service/
├── main.py                 # FastAPI + lifespan (só conexão com o banco)
├── presentation/           # Rotas HTTP + schemas Pydantic
├── application/            # Casos de uso: analyze.py (analyze + retrieve-similar)
├── db/                     # connection + migrations 001/002
├── models/                 # ORM SQLAlchemy
├── collection/             # Steam: appdetails, top sellers, reviews (GetAppReviews)
├── embeddings/             # MiniLM (jogos) + multilíngue (aspectos)
├── retrieval/              # similares + leitura de temas/stats/drivers/reviews
├── absa/                   # Gemini client, taxonomia, extração, Kappa/F1, significância
├── themes/                 # clusterização e nomeação de temas
├── ml/                     # drivers.py, train_drivers.py, experiments.py
├── triangulation/          # engine do dashboard
└── scripts/                # pipeline offline
```

## 🚀 Boot (`main.py`)
1. Se `DATABASE_URL` → `init_engine()` (SQLAlchemy + tipo Vector).
2. Sem DB o serviço sobe; `/health` responde 200 e `/analyze` devolve 503.

## 🛣️ Endpoints

| Método | Path | Precisa DB? | Caso de uso |
|--------|------|-------------|-------------|
| `GET` | `/` e `/health` | Não | Status + `database_ok` + `contagens` das 6 tabelas do pipeline |
| `POST` | `/retrieve-similar` | **Sim** | `RetrieveSimilarUseCase` |
| `POST` | `/analyze` | **Sim** | `AnalyzePitchUseCase` |

Schemas: `presentation/schemas.py` — `GamePitchInput` (campos extras ignorados) e `RetrieveSimilarInput`.

## 🧭 `/analyze` passo a passo
1. Embedding do pitch (título + descrição + gêneros) → jogos similares no pgvector ([[Retrieval e Busca Vetorial]]).
2. Temas nas reviews **dos similares**, calculados na hora (`compute_theme_stats`, mínimo de 3 menções).
3. `theme_stats` **dos gêneros** do pitch (pré-calculado).
4. Drivers do RF dos gêneros, com fallback no modelo global ([[Drivers de Recomendacao]]).
5. Reviews de referência mais úteis de cada similar (positivas e negativas, com resposta do dev).
6. `TriangulationEngine.build` → dashboard; `coverage` → quantas reviews sustentam a análise.

A Gemini não roda aqui em tempo de requisição — só nos scripts offline (ABSA e nome dos temas).

## 📜 Scripts offline (ordem)

| Ordem | Script | Função |
|-------|--------|--------|
| 1 | `scripts/apply_schema.py` | Aplica migrations (idempotente) |
| 2 | `scripts/bootstrap_games_from_json.py` | Seed jogos/gêneros |
| 3 | `scripts/collect_top_games.py` | Catálogo na ordem dos mais vendidos ([[Catalogo pelos Mais Vendidos]]) |
| 4 | `scripts/backfill_embeddings.py` | Embeddings dos jogos |
| 5 | `scripts/collect_reviews.py` | Reviews sem autor ([[Coleta de Reviews]]) |
| 6 | `scripts/run_absa.py` | Aspectos via Gemini ([[Extração de Aspectos (ABSA)]]) |
| 7 | `scripts/cluster_themes.py` | Temas ([[Temas Recorrentes e Metricas]]) |
| 8 | `scripts/compute_theme_stats.py` | Métricas por gênero |
| 9 | `ml/train_drivers.py` | RF de drivers |
| — | `scripts/export_gold_standard.py` / `evaluate_gold_standard.py` | Validação Kappa/F1 |
| — | `scripts/test_pipeline.py` | Testes sintéticos sem rede |

## 🔐 Variáveis de ambiente

| Variável | Uso |
|----------|-----|
| `DATABASE_URL` | Postgres + pgvector |
| `EMBEDDING_MODEL` | Jogos — default `all-MiniLM-L6-v2` |
| `ASPECT_EMBEDDING_MODEL` | Aspectos — default `paraphrase-multilingual-MiniLM-L12-v2` |
| `SIMILAR_GAMES_TOP_K` | Default `5` |
| `STEAM_API_KEY` | Coleta |
| `GEMINI_API_KEY` / `GEMINI_MODEL` / `GEMINI_THINKING_BUDGET` | ABSA e nomeação de temas (offline) |
| `EXPERIMENTS_DIR` | Logs de experimento (default `<repo>/experiments`) |

## ⚠️ Armadilhas conhecidas
- Primeira chamada que carrega `sentence-transformers` é lenta / pesada em memória.
- Sem reviews processadas, o dashboard vem vazio com alerta `dados_insuficientes` (não é bug).
- Re-rodar `cluster_themes.py` recria os temas → é preciso rodar `compute_theme_stats.py` e `train_drivers.py` de novo.

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[API Principal Node]], [[Motor de Triangulacao]], [[Retrieval e Busca Vetorial]], [[Modelagem e ML]], [[Schema do Banco de Dados]]
