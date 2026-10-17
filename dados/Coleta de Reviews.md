# 📥 Coleta de Reviews

Coleta de reviews públicas pela API oficial da Steam, com exclusão obrigatória de identificadores de autor. É a matéria-prima de todo o dashboard ([[Extração de Aspectos (ABSA)]] → [[Temas Recorrentes e Metricas]]).

## 🔌 Endpoint
- Principal: `IUserReviewsService/GetAppReviews/v1`.
- Fallback para `store.steampowered.com/appreviews` se o serviço responder 400/403/404/405.
- Retry com backoff em HTTP 429; pausa de 1,5 s entre páginas. **Não paralelizar.**

## 🎯 Escopo de jogos
- Só `games.na_amostra = TRUE` (`scripts/select_sample.py`, estratificado por gênero).
- Mínimo de **200** `total_reviews_steam` (senão `elegivel_metricas = FALSE`).
- Ordem de coleta: **round-robin por gênero** (mais vendidos primeiro dentro do gênero), para não enviesar o banco para AAA se a coleta/ABSA parar no meio.

## 📦 Fatias por jogo
Uma review, várias fatias (`reviews.fatias TEXT[]`). **Não há dedup entre fatias** — se uma recente também é útil, fica nas duas.

| Fatia | Default | Fonte |
|---|---|---|
| `proporcional` | **50 por idioma** (PT-BR + EN ≈ 100) | `filter=recent`, sem filtro de tipo |
| `util_positiva` | 25 | Mais úteis de todos os tempos, recomendadas |
| `util_negativa` | 25 | Mais úteis de todos os tempos, não recomendadas |

Antes de contar a cota: descarta texto < 20 chars, só pontuação e ASCII art; pagina mais se a fatia esvaziar. Near-dup por texto normalizado dentro da fatia+idioma.

### ⚠️ Limite da proporcional
`filter=recent` cobre as **últimas semanas**, não a vida do jogo (onde review bombing e patches pesam). `timestamp_created` é gravado; no TCC a proporcional é descrita como “recente”, não “ao longo da vida”.

## 📐 Onde cada fatia entra
- **Métricas e RF** (`theme_stats`, `train_drivers`, `load_scope`): só `'proporcional' = ANY(fatias)` e `elegivel_metricas`.
- **Citações / reviews de referência**: preferem fatias `util_*`.

## 📸 Snapshot e review bombing
A cada jogo: `review_snapshots` com totais oficiais e, em paralelo, totais com `filter_offtopic_activity` ligado/desligado (`*_unfiltered`). Campos da review: `steam_purchase`, `received_for_free`, `timestamp_created`, `playtime_at_review`.

## 🛡️ Conformidade
- `sanitize_review` descarta o bloco `author` inteiro (incl. `steamid`).
- Dedup por `recommendationid`. Ver [[Conformidade Steam e LGPD]].

## 🔁 Retomada e reprodutibilidade
- Commit por jogo; `reviews_collected_at` marca o checkpoint (`--force` recoleta).
- Parâmetros da execução vão para `/experiments/collect_reviews_*.json`.

---
*Relacionado:* [[Conformidade Steam e LGPD]], [[Schema do Banco de Dados]], [[Extração de Aspectos (ABSA)]], [[Temas Recorrentes e Metricas]], [[Drivers de Recomendacao]], [[Popular e Verificar o Banco]]
