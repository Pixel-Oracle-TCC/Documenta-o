# 🏗️ Arquitetura do Frontend

O Angular (`Pixel-Oracle-FrontEnd`) segue **clean architecture** espelhando a [[Arquitetura de Microsservicos|API]]: camadas `domain`, `application`, `infrastructure` e `presentation`. Componentes não chamam `HttpClient` diretamente — passam por use-cases e ports.

## 📁 Camadas
- **domain** — entidades (`GamePitch`, `FullReport` v3 copiado de `api/src/domain/entities/FullReport.ts`) e ports (`IReportRepository`, `IHealthRepository`).
- **application** — use-cases (`GenerateFullReportUseCase`, `CheckApiHealthUseCase`) e mapper de formulário → pitch.
- **infrastructure** — adapters HTTP, normalização da resposta, mapeamento de erros 422/502/503/timeout.
- **presentation** — páginas (`home`, `analisar`), layouts e UI compartilhada: card de destaque (tema + métricas + citações), lista de fatores decisivos, card de review de referência, painel de evidências, empty states.

## 🔌 Dependências
- Injeção via `InjectionToken` (`REPORT_REPOSITORY`, `HEALTH_REPOSITORY`) em `app.config.ts`.
- `environment.apiUrl` aponta para `http://localhost:3000`.
- Timeout do relatório: 180 s (primeira chamada carrega o modelo de embedding).

## 🚫 Limites
- Não chama o ml-service (`8000`) direto — ver [[Contratos de API]].
- Não exibe nenhum dado de autor de review ([[Conformidade Steam e LGPD]]).

---
*Relacionado:* [[Guia de Atualizacao do Frontend]], [[Fluxo Analisar Pitch]], [[Design System PixelOracle]], [[Contratos de API]], [[Guia Ponta a Ponta da API]]
