# 🧪 Popular e Verificar o Banco

Pipeline offline completo para Postgres + pgvector, do schema até os drivers do RF. Pré-requisito: Docker Desktop **ligado** e serviço `db` no ar.

## ▶️ 1. Subir o Postgres

Na raiz `PixelOracle-API`:

```bash
docker compose up -d db
docker compose ps   # db deve estar healthy
```

No **primeiro** start (volume vazio), o Compose aplica todas as migrations de `ml-service/db/migrations/`. Em volume já existente:

```bash
cd ml-service && source .venv/bin/activate
python3 scripts/apply_schema.py     # idempotente
```

## 🌱 2. `ml-service/.env` (scripts no host)

```env
DATABASE_URL=postgresql://pixel:pixel@localhost:5433/pixeloracle
EMBEDDING_MODEL=all-MiniLM-L6-v2
ASPECT_EMBEDDING_MODEL=paraphrase-multilingual-MiniLM-L12-v2
STEAM_API_KEY=...
GEMINI_API_KEY=...
GEMINI_MODEL=gemini-2.5-flash
GEMINI_THINKING_BUDGET=0
```

Credenciais do Compose: `pixel` / `pixel` / `pixeloracle`, porta **5433** no host.

## 📦 3. Popular (ordem)

Recebeu um dump do grupo? Pule esta seção: [[Compartilhar o Banco com o Grupo]].

```bash
cd ml-service && source .venv/bin/activate

python3 scripts/apply_schema.py
python3 scripts/bootstrap_games_from_json.py
python3 scripts/collect_top_games.py --limit 100
python3 scripts/enrich_games.py
python3 scripts/backfill_embeddings.py
python3 scripts/select_sample.py --per-genre 8 --total 80
python3 scripts/collect_reviews.py --limit-games 0
python3 scripts/run_absa.py --limit 200
python3 scripts/cluster_themes.py
python3 scripts/compute_theme_stats.py
python3 ml/train_drivers.py
```

No terminal zsh, cole os comandos sem comentário `#` na mesma linha (ou `setopt interactivecomments`).

Todos são idempotentes. Re-rodar `cluster_themes.py` recria os temas → rode de novo os dois passos seguintes.

| Script | Tabela | Nota |
|--------|--------|------|
| `bootstrap_games_from_json.py` | `games`, `genres` | — |
| `collect_top_games.py` | `rank_popularidade`, `apps_ignorados` | [[Catalogo pelos Mais Vendidos]] |
| `enrich_games.py` | `release_date`, categorias, estúdios, idiomas | appdetails Store |
| `select_sample.py` | `games.na_amostra` | estratificado por gênero |
| `backfill_embeddings.py` | `games.embedding` | [[Retrieval e Busca Vetorial]] |
| `collect_reviews.py` | `reviews`, `review_snapshots` | [[Coleta de Reviews]] |
| `run_absa.py` | `review_aspects` | [[Extração de Aspectos (ABSA)]] |
| `cluster_themes.py` | `themes` | [[Temas Recorrentes e Metricas]] |
| `compute_theme_stats.py` | `theme_stats` | [[Temas Recorrentes e Metricas]] |
| `ml/train_drivers.py` | `genre_drivers` | [[Drivers de Recomendacao]] |

## ✅ 4. Verificar

```bash
curl -s http://localhost:8000/health | jq .contagens
```

Ou direto no banco:

```bash
docker compose exec db psql -U pixel -d pixeloracle -c "
  SELECT
    (SELECT COUNT(*) FROM games WHERE embedding IS NOT NULL) AS jogos_com_vetor,
    (SELECT COUNT(*) FROM reviews) AS reviews,
    (SELECT COUNT(*) FROM reviews WHERE absa_processed_at IS NULL) AS absa_pendente,
    (SELECT COUNT(*) FROM review_aspects) AS aspectos,
    (SELECT COUNT(*) FROM themes) AS temas,
    (SELECT COUNT(*) FROM theme_stats) AS stats,
    (SELECT COUNT(*) FROM genre_drivers) AS drivers;
"
```

Conformidade: `\d reviews` não deve listar nenhuma coluna de autor ([[Conformidade Steam e LGPD]]).

## ⚠️ Armadilhas
- Docker Desktop fechado → `Cannot connect to the Docker daemon`.
- Scripts no host usam `localhost:5433`; o container ml usa hostname `db`.
- Cota diária da Gemini esgotada → `run_absa.py` para e grava o parcial; continue no dia seguinte ou troque `GEMINI_MODEL`.

---
*Relacionado:* [[Como Rodar o Projeto]], [[Schema do Banco de Dados]], [[Catalogo pelos Mais Vendidos]], [[Coleta de Reviews]], [[Proximos Passos do Pipeline]], [[Por que MiniLM e pgvector]]
