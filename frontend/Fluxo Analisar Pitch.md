# 🧭 Fluxo Analisar Pitch

Fluxo principal do produto no Angular: o desenvolvedor descreve o jogo e recebe, via `POST /api/report`, o que jogadores de jogos parecidos mais elogiam e mais criticam — com métricas, jogos-exemplo e trechos reais de reviews. Especificação de tela detalhada em [[Guia de Atualizacao do Frontend]].

## 🛣️ Telas
- `/` — landing com o posicionamento "evidência de reviews reais" e CTA para analisar.
- `/analisar` — formulário + resultado na mesma rota (signals).

## 🛠️ Entrada (pitch)
Obrigatórios: `titulo`, `descricao` (≥ 10), `genres` (≥ 1). Opcionais: `price_brl`, `top_k`. Contrato em [[Contratos de API]].

## 📉 Saída (relatório)
1. Resumo (`narrativa.resumo`) + badge de `fonte` (`gemini` | `fallback`) + `cobertura` (quantas reviews sustentam a análise).
2. Alertas (`dashboard.alertas`) em banners; `dados_insuficientes` vira empty state.
3. Cards de **elogios** e **reclamações** recorrentes: frase narrada (casada por `id`), tema, prevalência com intervalo de Wilson, jogos distintos, votos úteis, jogos-exemplo e citações expansíveis.
4. **Fatores decisivos** do RF: tema, direção (aumenta/reduz), taxa de recomendação com vs sem.
5. **Reviews de referência** por jogo similar, com resposta do desenvolvedor quando houver.
6. **Jogos similares** com similaridade e nota da Steam.

## ⚠️ Erros tratados
- `422` — validação do pitch.
- `502`/`503` — ML offline ou sem banco.
- Timeout / rede — banner e mensagem amigável (timeout de 120–180 s).
- Health check em `GET /api/health` ao abrir a página.

---
*Relacionado:* [[Arquitetura do Frontend]], [[Guia de Atualizacao do Frontend]], [[Contratos de API]], [[Motor de Triangulacao]], [[Por que Gemini so redige]]
