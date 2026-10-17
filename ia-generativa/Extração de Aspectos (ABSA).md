# 🧪 Extração de Aspectos (ABSA)

Pipeline de Aspect-Based Sentiment Analysis sobre reviews públicas da Steam. Cada review pode gerar **vários aspectos**, cada um com trecho literal que serve de evidência. Agregação sempre por **gênero/tema**, nunca por indivíduo — ver [[Conformidade Steam e LGPD]].

## 🎯 Saída por aspecto
`{categoria, sentimento, trecho, resumo}`:
- `categoria` — uma das 15 da taxonomia fixa (ver [[Temas Recorrentes e Metricas]]).
- `sentimento` — `positivo`, `negativo` ou `neutro` (neutros não entram nos temas, que são por polaridade).
- `trecho` — cópia **literal** da review; checado no código (`trecho_literal`). Só trechos literais viram citação no dashboard.
- `resumo` — paráfrase curta em português, usada para clusterizar.

## 🛠️ Implementação
- `absa/gemini_client.py` — REST da Gemini com `responseMimeType: application/json`, temperatura 0, retries.
- **Raciocínio desligado** (`GEMINI_THINKING_BUDGET=0`): num lote real de 10 reviews o tempo caiu de 37 s para 10,5 s, com 32 aspectos (31 literais) contra 38 com raciocínio. O valor fica gravado em `reviews.absa_model` (`gemini-2.5-flash@thinking=0`) para comparar no gold standard.
- `absa/extractor.py` — lotes de 10 reviews com IDs locais no prompt (o `recommendationid` não vai para a Gemini), máximo 5 aspectos por review, deduplicação e normalização.
- `scripts/run_absa.py` — processa reviews com `absa_processed_at` nulo; textos < 20 caracteres são marcados `curta-demais`; lote que falha fica pendente para a próxima execução.
- Resultado na tabela `review_aspects` (ver [[Schema do Banco de Dados]]).

## ✅ Validação (gold standard)
- `scripts/export_gold_standard.py` — amostra estratificada (~200) em CSVs para dois anotadores, formato `gameplay:positivo; historia:negativo` ou `nenhum`.
- `scripts/evaluate_gold_standard.py`:
  - **Kappa multi-rótulo** entre anotadores: cada célula (review × categoria) recebe `positivo/negativo/neutro/misto/ausente` (`misto` quando a mesma categoria aparece com sentimentos diferentes); também Kappa só de presença.
  - **F1 da Gemini** contra o consenso, sobre pares (categoria, sentimento) — micro e macro por categoria.
  - Resultado gravado em `absa_gold_standard` e em `/experiments/`.

## 📋 Status
- Código implementado e testado com dados sintéticos.
- Anotação humana do gold standard: trabalho manual do grupo (pendente).

---
*Relacionado:* [[Coleta de Reviews]], [[Temas Recorrentes e Metricas]], [[Motor de Triangulacao]], [[Por que Gemini so redige]], [[Conformidade Steam e LGPD]]
