# 🚀 API Principal Node

Documentação da pasta `api/` (Express + TypeScript) do monorepo PixelOracle-API. Entrada recomendada: [[Guia Ponta a Ponta da API]].

## 🎯 Papel no sistema
- Única porta HTTP pública para o frontend (`:3000`, prefixo `/api`).
- Valida o pitch, pede o dashboard ao [[ML Service FastAPI]] e usa a Gemini **só para redigir** — com validador de saída.
- **Não** acessa PostgreSQL diretamente.

## 🏗️ Organização em camadas

```
api/src/
├── index.ts                         # Bootstrap + DI manual
├── presentation/                    # Rotas, controller, validação, erros
├── application/
│   ├── use-cases/                   # GenerateFullReportUseCase
│   ├── narrative/                   # fallbackNarrative (sem Gemini)
│   └── validation/                  # highlightValidator (pós-Gemini)
├── domain/                          # Entidades (FullReport, GamePitch) + ports
├── services/                        # Adaptadores: ML HTTP + Gemini
└── infrastructure/prompts/          # dashboardNarrativePrompt
```

Padrão: **ports & adapters** ([[Por que Ports and Adapters]]). O caso de uso depende de `IMlService` e `IInsightsService`; o fallback mora na camada de aplicação, não no adaptador da Gemini.

## 🛣️ Rotas

| Rota | Controller | Dependências |
|------|------------|--------------|
| `GET /api/health` | `health` | `mlService.health()` |
| `POST /api/report` | `fullReport` | `parseGamePitch` + `GenerateFullReportUseCase` |
## ✅ Validação do pitch (`parseGamePitch`)
- Obrigatórios: `titulo`, `descricao` (≥ 10), `genres` (≥ 1). Opcionais: `price_brl`, `top_k`.
- Aliases `name`, `ideia`, `description`. Campos antigos do formulário são ignorados.

## 🧠 Caso de uso principal

```
execute(pitch)
  1. analysis = mlService.analyze(pitch)                 // POST /analyze
  2. raw = insightsService.narrateDashboard(pitch, analysis)
     narrativa = validateNarrative(analysis, raw)        // troca frases com jogo fora da evidência
     catch → validateNarrative(analysis, buildFallbackNarrative(...), "fallback")
  3. return { jogo, dashboard, narrativa, cobertura, evidencias }
```

## ♊ Narrativa e validador
- `GeminiService` — prompt `dashboardNarrativePrompt.ts`, `responseMimeType: application/json`, temperatura 0,3; devolve `{resumo, destaques[{id, frase}]}`.
- `FallbackInsightsService` — sem chave: usa as frases determinísticas do ml-service.
- `highlightValidator.ts` — para cada destaque, se a frase da Gemini citar um título conhecido (similares, jogos-exemplo de qualquer destaque, reviews de referência) que **não** está nos `jogos_exemplo` daquele destaque, a frase é substituída e o motivo vai para `narrativa.validacao`. Destaques sem frase da Gemini também recebem a frase-modelo (`fonte: "modelo"`).

Ver [[Por que Gemini so redige]].

## 🧯 Middleware de erro
`presentation/middleware/errorHandler.ts`: erros Axios do ml-service propagam status (ex.: 503 sem banco); falhas da Gemini no report não derrubam a requisição (fallback).

## 🔐 Variáveis de ambiente

| Variável | Default / nota |
|----------|----------------|
| `PORT` | `3000` |
| `ML_SERVICE_URL` | `http://localhost:8000` |
| `GEMINI_API_KEY` | Sem chave = fallback |
| `GEMINI_MODEL` | `gemini-2.5-flash` |

## ⚠️ Armadilhas conhecidas
- Timeout de 60 s para o ml-service: a 1ª chamada carrega o modelo de embedding e pode ser lenta.
- Frontend ainda no contrato antigo quebra no parse — ver [[Guia de Atualizacao do Frontend]].

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[ML Service FastAPI]], [[Contratos de API]], [[Motor de Triangulacao]], [[Por que Gemini so redige]]
