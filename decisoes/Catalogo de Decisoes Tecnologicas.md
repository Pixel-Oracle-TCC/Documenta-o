# 🧠 Catalogo de Decisoes Tecnologicas

Índice das **escolhas de stack e arquitetura** do Pixel Oracle, com o *porquê*, o que foi descartado e **onde isso aparece no código**. Objetivo: qualquer pessoa do squad entender o “porquê” sem depender de quem escreveu o módulo.

## 📖 Como usar este catálogo
1. Leia a decisão relacionada à área em que vai trabalhar.
2. Abra o arquivo citado (path do monorepo PixelOracle-API).
3. Se discordar ou quiser trocar a tecnologia, registre a proposta em `/decisoes/` (não troque silenciosamente).

**Onboarding técnico paralelo:** [[Guia Ponta a Ponta da API]].
**Comentários no código (índice):** [[Trechos Comentados no Codigo]].

## 🗂️ Decisões (clique e aprofunde)

| Tema | Nota | Em uma frase |
|------|------|--------------|
| Escopo do produto | [[Mudanca de Foco do Projeto]] | Evidência de reviews reais; não gestão ágil (Zênite) |
| Score de vendas | [[Mudanca de Foco do Projeto]] | Eliminado: vendas não são estimáveis com confiança |
| Banco de dados | [[MongoDB para PostgreSQL pgvector]] | Relacional + vetores no mesmo Postgres |
| Stack dos serviços | [[Por que FastAPI Express e TypeScript]] | Python no ML; Node tipado na borda HTTP pública |
| Modelo estatístico | [[Por que Random Forest]] | RF reaproveitado para drivers de recomendação (`voted_up`) |
| Temas e recorrência | [[Temas Recorrentes e Metricas]] | Taxonomia fixa + clusters; Wilson, qui-quadrado, BH |
| Relevância das citações | [[Awards da Steam e Recusa de Scraping]] | Awards fora da API; ranking por utilidade, sem scraping |
| Embeddings / RAG | [[Por que MiniLM e pgvector]] | 384d leve, busca cosseno nativa no banco |
| LLM (Gemini) | [[Por que Gemini so redige]] | Redação ≠ análise; análise fica no motor de triangulação |
| Arquitetura de código | [[Por que Ports and Adapters]] | Ports na API; casos de uso testáveis; fallback sem Gemini |
| Coleta Steam | [[Conformidade Steam e LGPD]] | Sem `steamid`; dedup por `recommendationid` |
| Separação de serviços | [[Por que dois servicos e um banco]] | Frontend só fala com `:3000`; banco só no ml-service |

## 🧭 Mapa mental rápido

```
Produto (pitch pré-lançamento)
    │
    ├─ Similaridade ─► MiniLM + pgvector ──────────► embeddings/, retrieval/
    ├─ Opinião ──────► ABSA Gemini offline ────────► absa/ + review_aspects
    ├─ Temas ────────► Agglomerative + métricas ───► themes/, absa/significance.py → theme_stats
    ├─ Drivers ──────► RandomForest (voted_up) ────► ml/drivers.py → genre_drivers
    ├─ Consolidação ─► TriangulationEngine ────────► triangulation/engine.py (/analyze)
    └─ Redação ──────► Gemini + validador (Node) ──► geminiService + highlightValidator
```

## ⚠️ Regras de ouro do squad
- **Não** colocar análise “do zero” no prompt da Gemini — ver [[Por que Gemini so redige]].
- **Não** persistir `author.steamid` — ver [[Conformidade Steam e LGPD]] e `collection/reviews_client.py` → `sanitize_review`.
- **Não** reintroduzir previsão de vendas/sucesso — ver [[Mudanca de Foco do Projeto]].
- **Não** fazer scraping do HTML da Steam (awards etc.) — ver [[Awards da Steam e Recusa de Scraping]].
- Mudança de modelo/prompt/engine → registrar em `/experiments/` (regra de rastreabilidade).

---
*Relacionado:* [[Projeto Pixel Oracle]], [[Guia Ponta a Ponta da API]], [[Desafio Tecnico da Pesquisa]], [[Mudanca de Foco do Projeto]]
