# 🏗️ Arquitetura de Microsserviços

Visão atual do Pixel Oracle após a [[Mudanca de Foco do Projeto]]: API Node orquestra e redige; ml-service concentra busca de similares, pipeline de reviews e montagem do dashboard; Postgres + pgvector persiste as evidências.

**Onboarding do squad:** comece por [[Guia Ponta a Ponta da API]].

## 🧩 Serviços
- **`pixel-oracle-api` (Express)** — `POST /api/report`, `GET /api/health`. Não acessa o banco. Detalhe: [[API Principal Node]].
- **`pixel-oracle-ml-service` (FastAPI)** — `POST /analyze`, `POST /retrieve-similar`, `GET /health`. Detalhe: [[ML Service FastAPI]].
- **`db` (PostgreSQL + pgvector)** — games, reviews, review_aspects, themes, theme_stats, genre_drivers. Ver [[Schema do Banco de Dados]].

## 🔄 Fluxo do relatório (`/api/report`)
1. API recebe o pitch.
2. Chama `/analyze`: similares + temas recorrentes + drivers do RF + reviews de referência ([[Motor de Triangulacao]]).
3. Gemini **redige** resumo e frases; validador remove citações de jogos fora da evidência.
4. Se a Gemini falhar, usa as frases determinísticas do ml-service.

## 🧪 Pipeline offline (fora do request)
[[Coleta de Reviews]] → [[Extração de Aspectos (ABSA)]] → [[Temas Recorrentes e Metricas]] → [[Drivers de Recomendacao]]. Roda por scripts no ml-service; o request só lê o resultado.

## 📁 Contratos e mapa
- [[Contratos de API]] / `CONTRATOS_API.md`
- [[Mapa de Arquivos do Repositorio]]

## 🐳 Compose
`db` (healthy) → `ml-service` (healthy em `/health`) → `api`.

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[Motor de Triangulacao]], [[MongoDB para PostgreSQL pgvector]], [[Mudanca de Foco do Projeto]]
