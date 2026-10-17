# 📜 Contratos de API

Espelho legível dos contratos entre `pixel-oracle-ml-service`, `pixel-oracle-api` e o frontend. Manter sincronizado com `CONTRATOS_API.md` na raiz do repositório PixelOracle-API (lá estão os JSONs completos).

Guia de leitura: [[Guia Ponta a Ponta da API]].

## 👥 Quem produz / quem consome

| Endpoint | Produtor | Consumidor |
|----------|----------|------------|
| `GET /health` | ml-service | api (`/api/health`) |
| `POST /retrieve-similar` | ml-service | debug |
| `POST /analyze` | ml-service | api (`/api/report`) |
| `GET /api/health` | api | frontend |
| `POST /api/report` | api | frontend |

## 🛠️ Pitch de entrada
Obrigatórios: `titulo`, `descricao` (≥ 10 caracteres), `genres[]`. Opcionais: `price_brl`, `top_k`.
Aliases: `name` → titulo; `ideia`/`description` → descricao. Campos extras são ignorados.

```json
{
  "titulo": "Floresta Pixel",
  "descricao": "Metroidvania 2D com crafting e exploração cooperativa local.",
  "genres": ["Indie", "Ação", "Aventura"]
}
```

## 📉 Respostas (resumo)

### `POST /analyze`
- `dashboard.destaques.elogios|reclamacoes[]` — tema, `frase`, `jogos_exemplo`, `metricas` (prevalência + Wilson, jogos distintos, votos úteis, q-value, log-odds, `significativo`), `citacoes` literais, `contexto`. Regras em [[Motor de Triangulacao]].
- `dashboard.fatores_decisivos[]` — saída do [[Drivers de Recomendacao]] (importância, `direcao`, taxas com/sem, métricas do modelo).
- `dashboard.alertas[]` — `dados_insuficientes | primeiras_horas | early_access | nicho_exigente`.
- `dashboard.reviews_referencia[]` — por jogo similar, reviews mais úteis positivas e negativas, com `resposta_desenvolvedor`.
- `dashboard.jogos_similares[]` — com `review_score_desc` e `pct_positivas` reais da Steam.
- `cobertura` — quantas reviews sustentam a análise.

### `POST /api/report`
Mesmo miolo de `/analyze` + `narrativa`: `resumo`, `destaques[{id, frase, fonte: gemini|modelo}]`, `fonte: gemini|fallback`, `validacao{frases_substituidas, motivos}`.

## ⚠️ Versão atual: v3 (2026-10-14)
Contrato de dashboard de evidências ([[Mudanca de Foco do Projeto]]). O frontend deve seguir [[Guia de Atualizacao do Frontend]].

## 🔁 Regra de sincronização
Ao mudar Pydantic (`ml-service/presentation/schemas.py`) ou tipos TS (`api/src/domain/entities/*`), atualizar **este** vault **e** `CONTRATOS_API.md` na mesma tarefa.

---
*Relacionado:* [[Arquitetura de Microsservicos]], [[API Principal Node]], [[ML Service FastAPI]], [[Motor de Triangulacao]], [[Guia Ponta a Ponta da API]], [[Guia de Atualizacao do Frontend]]
