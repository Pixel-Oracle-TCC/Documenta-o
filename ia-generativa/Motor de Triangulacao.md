# 🧭 Motor de Triangulação

Componente do [[ML Service FastAPI]] que monta o **dashboard de evidências** a partir de três fontes: jogos similares ao pitch ([[Retrieval e Busca Vetorial]]), temas recorrentes com métricas ([[Temas Recorrentes e Metricas]]) e fatores decisivos do RF ([[Drivers de Recomendacao]]). Tudo é montado **antes** de qualquer texto da Gemini. Não existe mais predição de vendas ([[Mudanca de Foco do Projeto]]).

## 🛠️ Entradas e Saídas
- **Entrada:** pitch (`titulo`, `descricao`, `genres`), jogos similares, `theme_stats` dos similares (calculado na hora) e dos gêneros (pré-calculado), drivers, reviews de referência.
- **Saída:** `{destaques: {elogios, reclamacoes}, fatores_decisivos, alertas, reviews_referencia, jogos_similares}` + `cobertura` — contrato em [[Contratos de API]].
- Implementação: `ml-service/triangulation/engine.py`, orquestrado por `application/analyze.py` no endpoint `POST /analyze`.

## 🎯 Regras de seleção dos destaques
- **Dois escopos:** `similares` (reviews dos jogos mais parecidos — mais específico) e `genero` (amostra maior). Similares entram primeiro; gênero completa sem repetir tema.
- Tema só vira destaque se aparece em **≥ 2 jogos distintos** — problema de um jogo só não é padrão do mercado.
- Ordenação por prevalência e depois votos úteis; até 4 por escopo e 6 por polaridade.
- Cada destaque carrega `frase` determinística, `jogos_exemplo`, `metricas` (prevalência + Wilson, jogos distintos, votos úteis, q-value, log-odds) e `citacoes` literais.

## 🚨 Alertas
- `dados_insuficientes` — nada analisado para similares nem gêneros.
- `primeiras_horas` — reclamação concentrada em quem jogou < 2h (≥ 50%).
- `early_access` — reclamação concentrada em reviews de acesso antecipado.
- `nicho_exigente` — mais da metade dos similares com < 70% de reviews positivas na Steam.

## ✍️ Redação e validação
A API Node pede à Gemini um resumo e a reescrita das frases. Um validador (`api/src/application/validation/highlightValidator.ts`) troca pela frase determinística qualquer frase que cite um jogo conhecido fora dos `jogos_exemplo` daquele destaque. Sem Gemini, o fallback usa só as frases do motor. Ver [[Por que Gemini so redige]].

## ⚠️ Decisão descartada
Deixar a Gemini gerar a análise direto do pitch — descartado por risco de alucinação sem fundamentação ([[Desafio Tecnico da Pesquisa]]).

---
*Relacionado:* [[Arquitetura de Microsservicos]], [[Guia Ponta a Ponta da API]], [[Por que Gemini so redige]], [[Temas Recorrentes e Metricas]], [[Drivers de Recomendacao]], [[Retrieval e Busca Vetorial]], [[Catalogo de Decisoes Tecnologicas]]
