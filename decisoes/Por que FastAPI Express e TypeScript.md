# ⚙️ Por que FastAPI, Express e TypeScript

Justificativa da stack dos dois serviços HTTP e onde cada escolha se materializa no código.

## 🎯 Problema que a stack resolve
Precisamos de (1) um serviço **bom em ML/dados** (Python) e (2) uma borda HTTP **estável para o frontend** (contrato tipado, CORS, orquestração com Gemini). Um monólito único misturaria ciclos de release e dependências pesadas (torch/sklearn) com a API pública.

## ✅ Escolhas

### ml-service → **Python + FastAPI + Pydantic**
- Ecossistema nativo de `scikit-learn`, `pandas`, `sentence-transformers`, `scipy`, SQLAlchemy.
- FastAPI gera validação automática via Pydantic (`presentation/schemas.py` → `GamePitchInput`).
- OpenAPI “de graça” ajuda o squad a inspecionar contratos em `/docs` do Uvicorn.

**Onde no código:**
- App e lifespan (conexão com o banco): `ml-service/main.py`
- Rotas: `ml-service/presentation/routes.py` (`/analyze`, `/retrieve-similar`, `/health`)
- Validação de entrada: `ml-service/presentation/schemas.py`

### API principal → **Node.js + Express + TypeScript**
- Mesmo ecossistema do frontend Angular do squad (TS).
- Express é fino e suficiente: duas rotas públicas, sem necessidade de Nest neste estágio.
- TypeScript força o contrato do relatório (`FullReport`, `EvidenceDashboard`, `Highlight`) perto do frontend.

**Onde no código:**
- Bootstrap/DI: `api/src/index.ts`
- Rotas: `api/src/presentation/routes/index.ts`
- Tipos do contrato: `api/src/domain/entities/FullReport.ts`
- Cliente HTTP do ML: `api/src/services/mlService.ts` (axios)

## ❌ Alternativas descartadas
| Alternativa | Por que não |
|-------------|-------------|
| Tudo em Python (FastAPI público) | Frontend/squad Node; Gemini já integrado na API TS; misturar CORS/produtos com deps de ML |
| Tudo em Node (ML em JS) | Random Forest + embeddings + scipy são de primeira classe em Python; portar seria retrabalho sem ganho científico |
| NestJS / Django | Overhead de framework para poucas rotas / serviço focado; prazo de TCC favorece clareza sobre “enterprise boilerplate” |
| Flask em vez de FastAPI | Menos validação/OpenAPI nativos; Pydantic + FastAPI encaixa melhor com contratos do TCC |

## 🛠️ Implicação prática para o squad
- Código de **coleta, ABSA, temas, métricas, RF, SQL** → só no `ml-service/`.
- Código de **orquestração HTTP pública e redação** → só no `api/`.
- Não duplicar lógica de análise nos dois lados: a API **orquestra** `/analyze` e só cuida de redação + validação.

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Por que dois servicos e um banco]], [[API Principal Node]], [[ML Service FastAPI]]
