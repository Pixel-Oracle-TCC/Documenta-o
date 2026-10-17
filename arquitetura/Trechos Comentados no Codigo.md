# 💬 Trechos Comentados no Codigo

Índice dos arquivos em que o squad colocou comentários de **funcionamento + porquê**. Use junto com [[Catalogo de Decisoes Tecnologicas]] e [[Mapa de Arquivos do Repositorio]].

## 🎯 Objetivo
Quem abrir o arquivo no IDE deve entender a decisão sem precisar perguntar no chat — especialmente conformidade, métricas e o papel da Gemini.

## 🟦 API Node (`api/`)

| Arquivo | O que os comentários cobrem |
|---------|-----------------------------|
| `src/index.ts` | DI manual; Gemini opcional; API não acessa Postgres |
| `src/application/use-cases/GenerateFullReportUseCase.ts` | Ordem analyze → narrativa → validador; fallback |
| `src/application/validation/highlightValidator.ts` | Por que checar jogos citados fora da evidência |
| `src/services/geminiService.ts` | Dois adapters da mesma port; JSON mode |
| `src/infrastructure/prompts/dashboardNarrativePrompt.ts` | Regras de redação sem invenção |

## 🟩 ml-service (`ml-service/`)

| Arquivo | O que os comentários cobrem |
|---------|-----------------------------|
| `application/analyze.py` | Fluxo do `/analyze` em 6 passos |
| `triangulation/engine.py` | Dois escopos; mínimo de 2 jogos distintos; alertas |
| `collection/reviews_client.py` | **Não persistir autor**; fatias úteis/recentes; fallback do endpoint |
| `absa/extractor.py` | Multi-aspecto; trecho literal; IDs locais no prompt |
| `absa/gemini_client.py` | Temperatura 0; orçamento de raciocínio |
| `absa/significance.py` | Wilson, BH, ranking de citações sem memes |
| `themes/clustering.py` | Cosseno/average; amostragem acima de 6000 |
| `ml/drivers.py` | Permutation importance em vez de MDI; mínimos de amostra |
| `db/migrations/002_reviews_focus.sql` | Tabelas sem autor; recriação condicional do gold standard |

## 🧭 Como contribuir
1. Ao tocar em trecho crítico (conformidade, métricas, prompt, engine), **mantenha ou melhore** o comentário de *porquê*.
2. Não comente o óbvio. Comente a **decisão**.
3. Se a decisão for nova, também atualize uma nota em `/decisoes/` ([[Catalogo de Decisoes Tecnologicas]]).

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[Catalogo de Decisoes Tecnologicas]], [[Mapa de Arquivos do Repositorio]], [[Por que Gemini so redige]], [[Conformidade Steam e LGPD]]
