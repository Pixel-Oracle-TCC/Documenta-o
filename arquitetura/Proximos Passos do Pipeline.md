# 🛤️ Proximos Passos do Pipeline

Status operacional do pipeline de evidências de reviews. Atualizar esta nota a cada marco.

## ✅ Já concluído
- Schema com migrations 001 + 002 aplicado (idempotente) — [[Schema do Banco de Dados]].
- 110 jogos com embeddings — [[Retrieval e Busca Vetorial]].
- Coleta via `GetAppReviews` funcionando: 492 reviews de 12 jogos, sem autor, com resumo da Steam por jogo — [[Coleta de Reviews]].
- ABSA multi-aspecto rodando com trechos literais (~99% literais na amostra) — [[Extração de Aspectos (ABSA)]].
- Código de temas, métricas, drivers, `/analyze`, API Node v3 e validador implementados; 11 testes sintéticos passando.

## ⚠️ Limite atual: cota gratuita da Gemini
- Free tier: **20 requisições/dia por modelo**. Com lotes de 10 reviews, isso é ~200 reviews/dia por modelo.
- `gemini-2.5-flash` esgotou a cota diária no primeiro dia de testes; parte da amostra foi processada com `gemini-3.8-flash` (registrado em `reviews.absa_model`).
- O script para sozinho quando a cota diária acaba, grava o parcial e deixa o resto pendente.
- **Decisão pendente do grupo:** fixar um único modelo para todo o ABSA (comparabilidade metodológica) e/ou ativar billing. Se fixar, reprocessar com `run_absa.py --reprocess`.

## 📋 Fila (ordem)

| # | Passo | Como | Depende de |
|---|--------|------|------------|
| 1 | Completar ABSA da amostra | `python3 scripts/run_absa.py --limit 200` (por dia, no free tier) | Cota Gemini |
| 2 | Temas | `python3 scripts/cluster_themes.py` | ABSA |
| 3 | Métricas por gênero | `python3 scripts/compute_theme_stats.py` | Temas |
| 4 | Drivers do RF | `python3 ml/train_drivers.py` | Temas (n ≥ 150 por modelo) |
| 5 | Teste ponta a ponta | `/analyze` e `/api/report` | 2–4 |
| 6 | Ampliar coleta | `collect_reviews.py --limit-games 0` | Cota/tempo |
| 7 | Gold standard | `export_gold_standard.py` → anotação manual → `evaluate_gold_standard.py` | ABSA |
| 8 | Frontend v3 | [[Guia de Atualizacao do Frontend]] | Contrato estável |

---
*Relacionado:* [[Como Rodar o Projeto]], [[Popular e Verificar o Banco]], [[Coleta de Reviews]], [[Extração de Aspectos (ABSA)]], [[Guia Ponta a Ponta da API]]
