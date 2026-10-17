# 🗺️ Guia Ponta a Ponta da API

Documento de onboarding do squad: como a API do Pixel Oracle funciona do request do frontend até a resposta final. Leia este arquivo primeiro; em seguida os **porquês** em [[Catalogo de Decisoes Tecnologicas]]; depois aprofunde em [[API Principal Node]], [[ML Service FastAPI]] e [[Contratos de API]].

## 🎯 Visão em uma frase
O **frontend** fala só com a **API principal** (`:3000`). Ela pede ao **ml-service** (`:8000`) o dashboard de evidências — jogos similares, temas recorrentes de elogio/reclamação, fatores decisivos e reviews de referência — e a Gemini apenas **redige** o texto, passando por um validador.

## 🧩 Peças do sistema

```
┌─────────────┐     HTTP      ┌──────────────────┐     HTTP      ┌─────────────────┐
│  Frontend   │ ────────────► │  api (:3000)     │ ────────────► │ ml-service      │
│  (Angular)  │  /api/*       │  Express/TS      │  /analyze     │ (:8000) FastAPI │
└─────────────┘               │  Gemini + valid. │               │ RAG + temas+RF  │
                              └──────────────────┘               └────────┬────────┘
                                                                          │
                                                                   PostgreSQL+pgvector
                                                       (games, reviews, review_aspects,
                                                        themes, theme_stats, genre_drivers)
```

| Peça | Porta | Repo / pasta | Responsabilidade |
|------|-------|--------------|------------------|
| Frontend | — | Pixel-Oracle-FrontEnd | UI; consome só `/api/*` |
| API principal | 3000 | `api/` | Validação do pitch, orquestração, redação Gemini + validador |
| ML service | 8000 | `ml-service/` | Similares, dashboard, pipeline offline de reviews |
| Banco | 5433 (host) | Docker `db` | Persistência relacional + vetores |

## 🛣️ Endpoints

### Públicos (frontend → API)
| Método | Path | O que faz |
|--------|------|-----------|
| `GET` | `/api/health` | Saúde da API + ping no ml-service |
| `POST` | `/api/report` | Dashboard completo + narrativa |

### Internos (API → ml-service)
| Método | Path | Quem chama |
|--------|------|------------|
| `GET` | `/health` | `/api/health` |
| `POST` | `/analyze` | `/api/report` |
| `POST` | `/retrieve-similar` | Debug |

Schemas formais: [[Contratos de API]] e `CONTRATOS_API.md`.

## 🔄 Fluxo principal — `POST /api/report`
1. `parseGamePitch` valida `titulo`, `descricao`, `genres`.
2. `GenerateFullReportUseCase` chama `mlService.analyze(pitch)` → ml-service `/analyze` (passo a passo em [[ML Service FastAPI]]).
3. Com o dashboard pronto, `insightsService.narrateDashboard` pede à Gemini um resumo e uma frase por destaque (amarrada pelo `id`).
4. `validateNarrative` troca pela frase determinística qualquer frase que cite jogo fora dos `jogos_exemplo` daquele destaque, e registra o motivo em `narrativa.validacao`.
5. Sem Gemini (chave ausente ou erro): `buildFallbackNarrative` usa só as frases do ml-service (`fonte: "fallback"`).
6. Resposta: `{ jogo, dashboard, narrativa, cobertura, evidencias }`.

## 🔄 `GET /api/health`
API chama `/health` do ml-service, que sempre devolve 200 com `database_ok` e `contagens` (games, reviews, review_aspects, themes, theme_stats, genre_drivers) — útil para saber se o pipeline já rodou.

## 🗄️ Dados offline (antes do dashboard ficar rico)
Ordem dos scripts em [[ML Service FastAPI]]; comandos prontos em [[Popular e Verificar o Banco]]. Resumo: schema → jogos → embeddings → [[Coleta de Reviews]] → [[Extração de Aspectos (ABSA)]] → [[Temas Recorrentes e Metricas]] → [[Drivers de Recomendacao]].

## 🔐 Variáveis de ambiente (resumo)

| Serviço | Essenciais |
|---------|------------|
| `api/` | `PORT`, `ML_SERVICE_URL`, `GEMINI_API_KEY`, `GEMINI_MODEL` |
| `ml-service/` | `DATABASE_URL`, `EMBEDDING_MODEL`, `ASPECT_EMBEDDING_MODEL`, `STEAM_API_KEY`, `GEMINI_API_KEY`, `GEMINI_MODEL` |

## 🖥️ Frontend
Contrato v3 e especificação de telas: [[Contratos de API]] e [[Guia de Atualizacao do Frontend]].

## 📚 Para onde ir depois
- **Porquês:** [[Catalogo de Decisoes Tecnologicas]]
- **Comentários no código:** [[Trechos Comentados no Codigo]]
- [[API Principal Node]] · [[ML Service FastAPI]] · [[Mapa de Arquivos do Repositorio]]
- Produto/método: [[Mudanca de Foco do Projeto]], [[Motor de Triangulacao]]

---
*Relacionado:* [[Arquitetura de Microsservicos]], [[Catalogo de Decisoes Tecnologicas]], [[Trechos Comentados no Codigo]], [[API Principal Node]], [[ML Service FastAPI]], [[Contratos de API]], [[Projeto Pixel Oracle]]
