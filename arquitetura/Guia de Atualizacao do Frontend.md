# 🖥️ Guia de Atualização do Frontend

Documento para o squad Angular (`Pixel-Oracle-FrontEnd`) migrar o front para o **contrato v3**: o score de viabilidade foi eliminado ([[Mudanca de Foco do Projeto]]) e o relatório passou a ser um **dashboard de evidências de reviews reais**.

**Repos:** API em `PixelOracle-API` (porta `3000`). O front **não** chama o ml-service (`8000`) diretamente.

Contrato formal: [[Contratos de API]] / `CONTRATOS_API.md` no repo da API. Tipos canônicos: `api/src/domain/entities/FullReport.ts` (copiar para o domínio do front).

---

## 🎯 O que o front comunica
- **Elogios e reclamações recorrentes** em jogos parecidos, com métricas.
- Temas com jogos-exemplo e **citações literais** de reviews.
- **Fatores decisivos**: o que mais pesa para uma review recomendar ou não.
- **Reviews de referência** mais úteis, com resposta do desenvolvedor.
- Nenhum número de sucesso ou vendas.

**Frase-guia:** *"Descreva seu jogo. Veja o que jogadores de jogos parecidos mais elogiam e mais criticam — com números e trechos reais das reviews."*

---

## 🧩 Endpoints

| Método | Path | Uso no front |
|--------|------|----------------|
| `GET` | `/api/health` | Status / banner "API offline" |
| `POST` | `/api/report` | Tela principal — dashboard completo |

---

## 🛠️ Request do pitch

```json
{
  "titulo": "Floresta Pixel",
  "descricao": "Metroidvania 2D com crafting e exploração cooperativa local.",
  "genres": ["Indie", "Ação", "Aventura"]
}
```

- Obrigatórios: `titulo`, `descricao` (≥ 10), `genres` (≥ 1). Opcionais: `price_brl`, `top_k`.
- Campos extras são ignorados pela API; o formulário deve ter só os campos acima.

---

## 🖼️ Telas recomendadas

### 1. Formulário
Título, descrição (textarea), gêneros (multi-select). CTA **"Analisar reviews de jogos parecidos"**. Loading: *"Buscando jogos parecidos e reviews…"* (1ª chamada pode passar de 30 s).

### 2. Resumo
`narrativa.resumo` em destaque. Badge: `fonte === 'gemini'` → "Redigido pela IA"; `'fallback'` → "Texto automático".
Abaixo, `cobertura`: "Baseado em N reviews de jogos similares e M reviews do gênero" — deixa explícito o tamanho da evidência.

### 3. Cards de destaque (elogios e reclamações)
Duas colunas: `dashboard.destaques.elogios` (verde) e `.reclamacoes` (vermelho). Cada card:
- **Frase:** buscar em `narrativa.destaques` pelo mesmo `id`; se não achar, usar `highlight.frase`.
- **Rótulo do tema** (`tema.rotulo`) + chip da categoria.
- **Métricas:** "em X% das reviews" (`prevalencia`, tooltip com `wilson_low`–`wilson_high`), "em N de M jogos" (`jogos_distintos`/`total_jogos`), votos úteis somados. Tag "estatisticamente significativo" se `significativo`.
- **Jogos-exemplo:** chips com `jogos_exemplo[].titulo`.
- **Citações (expandir):** `citacoes[]` — trecho entre aspas, jogo, 👍 votos úteis, horas jogadas.
- Chip `escopo`: "jogos similares" vs "gênero X".

### 4. Fatores decisivos
Lista de `dashboard.fatores_decisivos` ordenada por `importancia`:
- "Reviews que mencionam *{rotulo}* recomendam {taxa_recomendacao_com}% vs {taxa_recomendacao_sem}% sem" com seta ↑ (`aumenta`) / ↓ (`reduz`).
- Rodapé discreto com `modelo.roc_auc` e `n_amostras` (transparência metodológica).

### 5. Alertas
`dashboard.alertas[]` como banners âmbar com `mensagem`. `dados_insuficientes` → empty state honesto em vez de cards vazios.

### 6. Reviews de referência
Por jogo (`reviews_referencia[]`), abas positivas/negativas: texto (até 600 chars), 👍 `votos_uteis`, horas, idioma e, se existir, **bloco "Resposta do desenvolvedor"** (`resposta_desenvolvedor`).

### 7. Jogos similares
Cards com título, similaridade %, `review_score_desc` ("Very Positive"), `pct_positivas`, link `https://store.steampowered.com/app/{steam_appid}`.

---

## 🧹 O que remover do front
- Badge de risco, número `chance_sucesso_estimada_pct`, gráfico de `probabilidades`.
- Tipos `predicao_ml`, `DashboardPoint`, `GenreInsight`, `pontos_*`, `insights_genero`, `aviso`.
- Serviço de `/api/predict`.
- Copy da landing que fale em risco, predição ou chance de sucesso.

---

## ✅ Checklist de aceite

- [ ] Tipos copiados de `FullReport.ts` (v3)
- [ ] Formulário com título, descrição e gêneros
- [ ] Resumo + cobertura
- [ ] Cards de elogios/reclamações com frase narrada (casada por `id`), métricas, jogos-exemplo e citações
- [ ] Fatores decisivos com direção e taxas
- [ ] Alertas e empty state de `dados_insuficientes`
- [ ] Reviews de referência com resposta do desenvolvedor
- [ ] Jogos similares com nota da Steam
- [ ] Nenhum resquício de risco/score na UI ou na landing
- [ ] Erros 422 / 502 / 503 e timeout de 120–180 s

---

## 🧪 Como testar contra a API local

```bash
curl -s -X POST http://localhost:3000/api/report \
  -H 'Content-Type: application/json' \
  -d '{"titulo":"Floresta Pixel","descricao":"Metroidvania 2D com crafting e exploracao cooperativa local.","genres":["Indie","Ação","Aventura"]}' | jq
```

---

## 🗺️ Fora do escopo
- Chamar o ml-service direto.
- Qualquer dado de autor de review ou persona de jogador ([[Conformidade Steam e LGPD]]).

---
*Relacionado:* [[Contratos de API]], [[Mudanca de Foco do Projeto]], [[Motor de Triangulacao]], [[Temas Recorrentes e Metricas]], [[Drivers de Recomendacao]], [[Arquitetura do Frontend]], [[Fluxo Analisar Pitch]], [[Design System PixelOracle]]
