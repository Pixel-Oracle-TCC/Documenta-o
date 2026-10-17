# 🤖 Modelagem e ML

Núcleo estatístico do Pixel Oracle depois do [[Mudanca de Foco do Projeto]]: nenhum modelo prevê sucesso comercial. Os métodos quantitativos servem para **medir e explicar padrões em reviews reais**.

## 🧮 Métodos em uso
- **Busca vetorial** (MiniLM + pgvector, cosseno) para achar jogos similares ao pitch → [[Retrieval e Busca Vetorial]].
- **ABSA via Gemini** com validação Kappa / F1 → [[Extração de Aspectos (ABSA)]].
- **Clusterização hierárquica** dos aspectos em temas + silhouette → [[Temas Recorrentes e Metricas]].
- **Estatística de recorrência:** Wilson, qui-quadrado, log-odds, Benjamini-Hochberg → [[Temas Recorrentes e Metricas]].
- **Random Forest de drivers de recomendação** (`voted_up`) + permutation importance → [[Drivers de Recomendacao]].

## 🧠 Por que Random Forest
Justificativa da família de modelos e o histórico do uso anterior: [[Por que Random Forest]].

## 🧪 Rastreabilidade
Todo treino/clusterização/avaliação grava um JSON em `<repo>/experiments/` (`ml/experiments.py`, configurável por `EXPERIMENTS_DIR`): `temas`, `drivers_rf`, `gold_standard`.

---
*Relacionado:* [[Mudanca de Foco do Projeto]], [[Por que Random Forest]], [[Drivers de Recomendacao]], [[Temas Recorrentes e Metricas]], [[Motor de Triangulacao]], [[Catalogo de Decisoes Tecnologicas]]
