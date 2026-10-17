# 🏅 Awards da Steam e Recusa de Scraping

Registro da decisão sobre **quais sinais de relevância** usar para escolher as reviews citadas como referência no dashboard. A ideia inicial era priorizar reviews com mais *likes* e mais *awards* (prêmios de Pontos Steam).

## 🔎 O que a API oferece
O endpoint oficial `IUserReviewsService/GetAppReviews` (e o legado `appreviews`) devolve por review:
- `votes_up` (útil), `votes_funny` (engraçado), `weighted_vote_score` (score de utilidade ponderado pela Valve), `comment_count`.
- `developer_response` e `timestamp_dev_responded` quando o estúdio respondeu.
- Filtro `filter=all` + `day_range=0` = **"mais úteis de todos os tempos"**.

**Awards não aparecem em nenhum endpoint documentado.** Só existem no HTML da loja.

## ❌ Alternativa descartada: scraping do HTML
- Viola os Termos de Uso da Steam (acesso automatizado fora das APIs), além de ser frágil a mudanças de layout.
- O mesmo princípio de [[Conformidade Steam e LGPD]]: só dados obtidos pelos canais oficiais e sem identificar autores.

## ✅ O que fazemos em vez disso
- Ranking das citações: `(é_meme, -weighted_vote_score, -votes_up)` — reviews em que `votes_funny > votes_up` vão para o fim, para não citar piada como evidência.
- `developer_response` é exibido junto à review de referência, como exemplo de como o estúdio lidou com a crítica.
- Coleta das reviews mais úteis de todos os tempos (positivas e negativas separadas) + uma fatia recente. Ver [[Coleta de Reviews]].

---
*Relacionado:* [[Coleta de Reviews]], [[Conformidade Steam e LGPD]], [[Motor de Triangulacao]], [[Catalogo de Decisoes Tecnologicas]]
