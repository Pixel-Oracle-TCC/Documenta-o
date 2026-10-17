# ▶️ Como Rodar o Projeto

Guia operacional do monorepo PixelOracle-API para o squad, do zero até o relatório com dados reais. Fluxos de código: [[Guia Ponta a Ponta da API]]. Versão completa no `README.md` do repositório.

## 🧩 O que sobe
| Serviço | Porta (host) | Papel |
|---------|--------------|-------|
| `db` | 5433 | PostgreSQL + pgvector |
| `ml-service` | 8000 | Similares + dashboard (`/analyze`) |
| `api` | 3000 | Borda pública (`/api/*`) |

## ✅ Pré-requisitos
- Docker Desktop **aberto**, Python 3.9+, Node.js 18+.
- Chave Gemini (Google AI Studio); chave Steam Web API opcional.
- **Não commitar** `.env` com chaves.
- Para inspecionar o banco no editor: [[PostgreSQL no Cursor e VS Code]].

## 🛠️ 1. Instalar e configurar

```bash
cd ml-service
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # DATABASE_URL (localhost:5433), GEMINI_API_KEY, STEAM_API_KEY

cd ../api
npm install
cp .env.example .env        # GEMINI_API_KEY, GEMINI_MODEL
```

Variáveis do ml-service: `DATABASE_URL`, `EMBEDDING_MODEL`, `ASPECT_EMBEDDING_MODEL`, `STEAM_API_KEY`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `GEMINI_THINKING_BUDGET=0`. Detalhes em [[ML Service FastAPI]].

## 🚀 2. Subir a stack

**Docker (recomendado):**
```bash
docker compose up --build -d
docker compose ps                         # db, ml-service, api → healthy
curl -s http://localhost:3000/api/health | jq
```

**Local (dev):**
```bash
docker compose up -d db                                   # terminal 1
cd ml-service && uvicorn main:app --reload --port 8000    # terminal 2
cd api && npm run dev                                     # terminal 3
```

Volume do banco já existia? `python scripts/apply_schema.py` (idempotente).

## 📦 3. Popular o banco
Ordem: jogos → embeddings → reviews → ABSA → temas → métricas → drivers. Comandos e verificação em [[Popular e Verificar o Banco]]. Se alguém do grupo já tem os dados, basta importar o dump: [[Compartilhar o Banco com o Grupo]]. Sem esse passo o relatório responde com o alerta `dados_insuficientes`.

> **Cota gratuita da Gemini:** 20 requisições/dia por modelo (~200 reviews/dia no ABSA). API e ml-service com a mesma chave e o mesmo modelo dividem a cota.

## 🧪 4. Testar

```bash
curl -s http://localhost:8000/health | jq .contagens

curl -s -X POST http://localhost:3000/api/report \
  -H 'Content-Type: application/json' \
  -d '{"titulo":"Floresta Pixel","descricao":"Metroidvania 2D com crafting e exploração cooperativa local.","genres":["Indie","Ação","Aventura"]}' | jq

cd ml-service && python scripts/test_pipeline.py   # testes sintéticos
cd api && npx tsc --noEmit                         # tipos da API
```

## ⚠️ Armadilhas
- `Cannot connect to the Docker daemon` → Docker Desktop fechado.
- No Compose, a API usa `ML_SERVICE_URL=http://ml-service:8000`; no host, `localhost:8000`.
- Postgres do projeto na porta **5433** no host.
- Primeiro build do `ml-service` baixa o Torch (CPU forçado no Dockerfile); a 1ª chamada carrega o modelo de embedding (timeout de 120–180 s no cliente).
- `narrativa.fonte: "fallback"` → chave ausente, erro ou cota da Gemini esgotada.

## 🛑 Parar
```bash
docker compose down        # mantém volume do Postgres
docker compose down -v     # apaga o banco também
```

---
*Relacionado:* [[Guia Ponta a Ponta da API]], [[Popular e Verificar o Banco]], [[PostgreSQL no Cursor e VS Code]], [[Arquitetura de Microsservicos]], [[Por que dois servicos e um banco]], [[Proximos Passos do Pipeline]]
