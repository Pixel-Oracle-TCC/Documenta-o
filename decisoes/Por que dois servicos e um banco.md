# 🔀 Por que dois serviços e um banco

Por que separamos `api` (:3000) e `ml-service` (:8000), e por que **só o ml-service** fala com o PostgreSQL.

## 🎯 Objetivos
1. Frontend Angular consome **uma** base URL estável.
2. Dependências pesadas de ML (sklearn, sentence-transformers) não incham a API pública.
3. Um único dono do schema/dados evita drift (dois ORMs escrevendo `reviews`).

## ✅ Desenho atual

```
Frontend ──► api (Express) ──► ml-service (FastAPI) ──► Postgres+pgvector
                 │                    │
                 └── Gemini redação   ├── embeddings/ + retrieval/
                 └── validador        ├── triangulation/ (/analyze)
                                      └── pipeline offline (coleta, ABSA, temas, RF)
```

**Onde no código / ops:**
- Orquestração: `docker-compose.yml` (`db` → `ml-service` → `api`, healthchecks encadeados)
- API **não** tem driver `pg`/Prisma — só `ML_SERVICE_URL`
- ml-service: `DATABASE_URL` em `db/connection.py`; endpoints que precisam de vetor checam `get_database_url()`

## 📌 Consequências práticas
| Situação | Comportamento |
|----------|---------------|
| DB fora / sem `DATABASE_URL` | `/health` responde 200 com `database_ok: false`; `/analyze` e `/retrieve-similar` 503 |
| Gemini fora | `/api/report` ainda retorna dashboard + narrativa fallback |
| ml-service fora | `/api/health` e demais rotas da API falham (dependência crítica) |

Sem banco não há dashboard possível: todo o conteúdo vem de reviews persistidas.

## ❌ Alternativas descartadas
| Alternativa | Motivo |
|-------------|--------|
| Frontend → ml-service direto | Expõe porta interna; CORS/auth misturados com ML; contrato duplicado |
| API Node acessa Postgres | Duplicaria ORM/schema; risco de bypassar regras de conformidade na coleta |
| Três bancos (API + ML + vector) | Complexidade operacional demais para o TCC |

## 🛠️ Regra do squad
- Precisa de dado persistido (jogo, review, insight)? Implemente no **ml-service** (+ migration SQL).
- Precisa expor algo ao Angular? Passe pela **api** (`/api/...`), atualize [[Contratos de API]].

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Arquitetura de Microsservicos]], [[MongoDB para PostgreSQL pgvector]], [[Guia Ponta a Ponta da API]]
