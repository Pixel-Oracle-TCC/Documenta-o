# 🔁 Temas Recorrentes e Métricas

Como o Pixel Oracle transforma milhares de aspectos soltos extraídos pelo [[Extração de Aspectos (ABSA)]] em **temas recorrentes** ("história profunda com personagens carismáticos", "bugs que travam o progresso") e decide quais deles valem destaque no dashboard.

## 🧩 Taxonomia híbrida
- **15 categorias fixas** (`gameplay`, `controles`, `dificuldade`, `historia`, `personagens`, `graficos`, `audio`, `performance`, `bugs`, `conteudo`, `progressao`, `preco`, `multiplayer`, `suporte_dev`, `outros`) — comparáveis entre gêneros e validáveis no gold standard.
- **Subtemas descobertos por clusterização** dentro de cada par (categoria, polaridade) — capturam o que é específico ("roguelite com run curta demais").

## 🧠 Clusterização
- Cada aspecto tem um `resumo` em português; embedding com `paraphrase-multilingual-MiniLM-L12-v2` (384 dims, reviews em PT e EN caem no mesmo espaço).
- `AgglomerativeClustering` (cosseno, linkage *average*, `distance_threshold=0.35`); clusters com menos de 5 itens viram ruído.
- Acima de 6000 pontos: clusteriza uma amostra e atribui o resto ao centróide mais próximo.
- Qualidade: **silhouette (cosseno)** registrada em `/experiments/` (experimento `temas`).
- **Nome do tema:** a Gemini recebe só os resumos representativos e devolve um rótulo curto; se falhar, usa o resumo mais central (`rotulo_fonte = "representante"`).

## 📐 Métricas de recorrência (`theme_stats`)
Calculadas só sobre reviews com `'proporcional' = ANY(fatias)` e jogos `elegivel_metricas` ([[Coleta de Reviews]]). Por gênero e tema:
- **Prevalência** = reviews com o tema / reviews do gênero, com **intervalo de Wilson 95%** (estável com amostras pequenas).
- **Jogos distintos** — um tema só vira destaque se aparecer em ≥ 2 jogos (evita que um jogo barulhento domine).
- **Votos úteis somados** — peso da comunidade.
- **Associação com recomendação:** qui-quadrado 2×2 (tema × recomendada) + **log-odds** com correção Haldane-Anscombe.
- **Correção de múltiplas comparações:** Benjamini-Hochberg; `significativo` quando q < 0,05.
- **Contexto:** % das menções feitas com menos de 2h de jogo e % em early access (vira alerta no dashboard).
- **Evidência:** 3 jogos-exemplo e 3 citações literais mais úteis (ver [[Awards da Steam e Recusa de Scraping]] para o critério de ranking).

## 📁 Código
- `ml-service/themes/clustering.py`, `themes/naming.py`, `scripts/cluster_themes.py`.
- `ml-service/absa/significance.py`, `scripts/compute_theme_stats.py`.

---
*Relacionado:* [[Extração de Aspectos (ABSA)]], [[Drivers de Recomendacao]], [[Motor de Triangulacao]], [[Schema do Banco de Dados]], [[Por que MiniLM e pgvector]]
