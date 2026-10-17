# ⚖️ Conformidade Steam e LGPD

Decisão permanente: o Pixel Oracle **não** perfila usuários individuais da Steam. Esta nota liga a regra jurídica/ToS aos **pontos exatos do código** que o squad deve preservar em todo PR.

## ❌ O que foi descartado
- Persistir `author.steamid` (ou qualquer identificador de autor) vindo de `GetAppReviews`/`appreviews`.
- Chamar `GetPlayerSummaries`, `GetOwnedGames`, `GetPlayerAchievements`, `GetFriendList`, `GetPlayerBans` ou endpoints de `ISteamUser`/`IPlayerService` sobre terceiros.
- Personas de público-alvo baseadas em perfil individual.

## 📜 Fundamento
- **Termos de Uso da Steam Web API, Cláusula 2:** dados de um end user só a pedido do próprio usuário.
- **LGPD (Lei 13.709/2018):** perfilamento de indivíduo identificável sem base legal/consentimento.

## ✅ Alternativa adotada
- Escopo: jogos (`appdetails`, top sellers semanal), reviews públicas **sem autor**. Só a API oficial da Steam.
- Deduplicação por `recommendationid`.
- Insights via agregação por **gênero/tema** (`theme_stats`); citações exibidas mostram só trecho, votos e horas jogadas — nunca autor.
- Sem scraping de HTML (ex.: awards) — ver [[Awards da Steam e Recusa de Scraping]].

## 📌 Pontos principais no código (não regredir)

### Sanitização na ingestão
`ml-service/collection/reviews_client.py` → `sanitize_review`:
- Exige `recommendationid`.
- Monta dict **sem** chave `author`; só `playtime_at_review` é copiado (metadado da review), **steamid nunca**.
- Coberto por teste: `scripts/test_pipeline.py` → `test_sanitize_review_drops_author_identity`.

### Persistência
- Tabelas `reviews` e `review_aspects` (`001_schema.sql`, `002_reviews_focus.sql`, `models/orm.py`): **nenhuma** coluna de autor — comentário explícito na migration.
- UPSERT por `recommendationid` em `scripts/collect_reviews.py` (atualiza votos e resposta do dev, nunca cria linha duplicada).

### Clientes permitidos
- `collection/steam_client.py` — metadados de **app**.
- `collection/steam_client.py` — top sellers semanal (agregado por jogo) e `appdetails`; nenhum endpoint de perfil.

### Produto
- O dashboard lê `theme_stats`, `genre_drivers` e reviews sem autor, nunca perfis: `retrieval/review_repository.py`.
- Drivers do RF usam `playtime_at_review` (metadado da review) em faixas, não dado do jogador.

## 🛠️ Checklist de PR (squad)
- [ ] Nova coluna em `reviews`? Confirmar que não identifica autor.
- [ ] Novo endpoint Steam? Só dados de jogo/agregados.
- [ ] Feature de “persona”? Redirecionar para temas por gênero (`theme_stats`).

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Mudanca de Foco do Projeto]], [[Coleta de Reviews]], [[Schema do Banco de Dados]], [[Extração de Aspectos (ABSA)]]
