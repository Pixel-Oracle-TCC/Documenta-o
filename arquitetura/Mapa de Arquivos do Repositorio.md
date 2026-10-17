# 📁 Mapa de Arquivos do Repositorio

Inventário do monorepo **PixelOracle-API** para o squad achar rápido "onde fica o quê". Fluxos e responsabilidades: [[Guia Ponta a Ponta da API]].

## 📦 Raiz

| Arquivo | Propósito |
|---------|-----------|
| `CONTRATOS_API.md` | Contrato request/response v3 (espelho de [[Contratos de API]]) |
| `docker-compose.yml` | Sobe `db` (pgvector, aplica migrations no 1º boot) + `ml-service` + `api` |
| `README.md` / `DOCKER.md` | Setup operacional e ordem do pipeline de reviews |
| `experiments/` | JSONs de experimentos (temas, drivers, gold standard) |
| `.cursor/rules/*.mdc` | Regras de estilo, contratos, conformidade, documentação do vault |

## 🟦 `api/` — Express / TypeScript

| Path | Propósito |
|------|-----------|
| `src/index.ts` | Bootstrap, env, DI manual, listen |
| `src/presentation/routes/index.ts` | Monta `/health` e `/report` |
| `src/presentation/controllers/reportController.ts` | Handlers HTTP finos |
| `src/presentation/validation/parseGamePitch.ts` | Valida `titulo`, `descricao`, `genres` |
| `src/presentation/middleware/errorHandler.ts` | Traduz erros ML/Gemini/HTTP |
| `src/application/use-cases/GenerateFullReportUseCase.ts` | `/analyze` → narrativa → validador |
| `src/application/narrative/fallbackNarrative.ts` | Narrativa sem Gemini |
| `src/application/validation/highlightValidator.ts` | Bloqueia jogo citado fora da evidência |
| `src/domain/entities/FullReport.ts` | Tipos do dashboard, narrativa e relatório |
| `src/domain/entities/GamePitch.ts` | Tipo do pitch |
| `src/domain/ports/IMlService.ts` / `IInsightsService.ts` | Ports |
| `src/services/mlService.ts` | Axios → `/health`, `/analyze` |
| `src/services/geminiService.ts` | Redação Gemini + `FallbackInsightsService` |
| `src/infrastructure/prompts/dashboardNarrativePrompt.ts` | Prompt de redação |

Detalhe narrativo: [[API Principal Node]].

## 🟩 `ml-service/` — FastAPI / Python

### Núcleo HTTP
| Path | Propósito |
|------|-----------|
| `main.py` | App FastAPI + lifespan (só DB) |
| `presentation/routes.py` | `/`, `/health`, `/retrieve-similar`, `/analyze` |
| `presentation/schemas.py` | `GamePitchInput`, `RetrieveSimilarInput` |
| `application/analyze.py` | `AnalyzePitchUseCase`, `RetrieveSimilarUseCase` |
| `triangulation/engine.py` | Monta destaques, alertas, frases-modelo |

### Dados e pipeline de reviews
| Path | Propósito |
|------|-----------|
| `db/connection.py` | SQLAlchemy + `DATABASE_URL` |
| `db/migrations/001_schema.sql`, `002_reviews_focus.sql` | Schema ([[Schema do Banco de Dados]]) |
| `models/orm.py` | ORM (Review **sem** autor) |
| `collection/steam_client.py`, `catalog.py` | Metadados de jogos, top sellers e regras do catálogo |
| `collection/reviews_client.py` | GetAppReviews sanitizado |
| `embeddings/encoder.py` | MiniLM (jogos) + multilíngue (aspectos) |
| `retrieval/similar_games.py` | Query `<=>` pgvector |
| `retrieval/review_repository.py` | Leitura de temas, stats, drivers, reviews de referência |
| `absa/gemini_client.py`, `taxonomy.py`, `extractor.py` | ABSA multi-aspecto |
| `absa/significance.py` | Wilson, qui-quadrado, BH, citações |
| `absa/validation.py` | Kappa multi-rótulo e F1 |
| `themes/clustering.py`, `naming.py` | Temas recorrentes |
| `ml/drivers.py`, `train_drivers.py` | RF de drivers |
| `ml/experiments.py` | Log de experimentos |

### Scripts
| Path | Propósito |
|------|-----------|
| `scripts/apply_schema.py` | Aplica migrations |
| `scripts/bootstrap_games_from_json.py` / `backfill_embeddings.py` | Seed de jogos e vetores |
| `scripts/collect_reviews.py` | Coleta de reviews |
| `scripts/run_absa.py` | ABSA |
| `scripts/cluster_themes.py` / `compute_theme_stats.py` | Temas e métricas |
| `scripts/export_gold_standard.py` / `evaluate_gold_standard.py` | Validação |
| `scripts/test_pipeline.py` | Testes sintéticos |

Detalhe narrativo: [[ML Service FastAPI]].

## 🧭 Como usar este mapa
1. "O frontend quebrou no report" → [[Contratos de API]] + `FullReport.ts`.
2. "Dashboard vazio" → `/health` → `contagens`; rodar o pipeline em [[Popular e Verificar o Banco]].
3. "Tema com nome estranho" → `themes/naming.py` + `rotulo_fonte` em `themes`.
4. "A Gemini citou jogo errado" → `highlightValidator.ts` + `narrativa.validacao`.
5. "Posso guardar steamid?" → **não** — [[Conformidade Steam e LGPD]].

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[API Principal Node]], [[ML Service FastAPI]], [[Arquitetura de Microsservicos]]
