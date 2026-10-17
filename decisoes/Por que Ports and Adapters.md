# 🧱 Por que Ports and Adapters

Por que a API Node (e partes do ml-service) separam **domínio/casos de uso** de **adaptadores** (HTTP, Gemini, axios) — e o que isso muda no dia a dia do squad.

## 🎯 Problema
Se `GenerateFullReportUseCase` importasse diretamente `@google/generative-ai` e axios,:
- testes unitários precisariam de rede/chave;
- trocar Gemini por outro LLM quebraria o caso de uso;
- o fallback “sem chave” viraria `if` espalhado.

## ✅ Padrão adotado (API)
- **Ports** (interfaces): `domain/ports/IMlService.ts`, `IInsightsService.ts`
- **Adapters** (implementações): `services/mlService.ts`, `services/geminiService.ts` (+ `FallbackInsightsService`)
- **Use case**: depende só das ports — `application/use-cases/GenerateFullReportUseCase.ts`
- **Wiring**: composição manual em `api/src/index.ts` (sem Nest/Inversify)

```
Controller → GenerateFullReportUseCase → IMlService / IInsightsService
                                      ↗ MlService (HTTP)
                                      ↗ GeminiService | FallbackInsightsService
```

## 📌 Pontos principais no código
- Troca Gemini ↔ fallback **sem** mudar o use case: só a instância injetada no `index.ts`.
- `analyze` na port `IMlService` espelha o endpoint real do ml-service — o use case não monta URL.
- Fallback (`application/narrative/`) e validador (`application/validation/`) ficam na camada de aplicação, independentes do adaptador da Gemini.
- Presentation (`parseGamePitch`, `reportController`) não contém regra de negócio.

## ✅ Eco no ml-service
- Casos de uso em `application/analyze.py` (`AnalyzePitchUseCase`, `RetrieveSimilarUseCase`).
- Rotas em `presentation/routes.py` só despacham e traduzem HTTPException.
- Acesso ao banco atrás de módulos (`db/connection.py`, `retrieval/review_repository.py`).
- `TriangulationEngine`, `compute_theme_stats` e o validador de narrativa (Node) são puros (dicts in/out) — testáveis em `scripts/test_pipeline.py` **sem** Postgres/Gemini.
## ❌ Alternativas descartadas
| Alternativa | Motivo |
|-------------|--------|
| “Tudo no controller” | Impossível testar fallback/contrato com clareza; vira macarrão |
| Framework DI pesado | Duas rotas; wiring manual é legível para o tamanho do TCC |

## 🛠️ Regra para PRs do squad
Nova integração externa (ex.: outro LLM, cache Redis) → **criar port + adapter**; não importar SDK dentro do use case.

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Por que FastAPI Express e TypeScript]], [[Por que Gemini so redige]], [[API Principal Node]]
