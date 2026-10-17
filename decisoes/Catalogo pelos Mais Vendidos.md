# 📈 Catálogo pelos Mais Vendidos

O catálogo de jogos é populado na ordem dos mais vendidos para os menos, para que a [[Coleta de Reviews]] priorize os jogos com mais reviews e mais relevância no mercado. A ordem vem **só da API oficial da Steam**.

## 🛠️ Como funciona
- `scripts/collect_top_games.py` baixa as últimas N semanas (`--weeks`, padrão 52) de `IStoreTopSellersService/GetWeeklyTopSellers`: o top 100 por receita de cada semana no Brasil.
- As semanas viram um ranking único: cada semana dá `101 − posição` pontos (1º = 100, 100º = 1). Empates são decididos pelas semanas no top 100 e depois pela melhor posição. O resultado vai para `games.rank_popularidade` (1 = mais vendido).
- Cada jogo novo é completado com o `appdetails` da Store BR: descrição, preço em reais, ano e gêneros.
- DLCs (já no top sellers), softwares (gêneros 50–60 da Steam) e apps sem página no Brasil vão para `apps_ignorados` ([[Schema do Banco de Dados]]).
- Gêneros são normalizados pelo id da Steam para os nomes já gravados ("Desporto", não "Esportes"), evitando duplicatas no recorte por gênero de [[Temas Recorrentes e Metricas]].
- Retomada: o ranking é regravado a cada execução; jogos já no banco e apps descartados não consultam a Store de novo. `collect_reviews.py` segue a mesma ordem.

## 📊 Alcance
- 52 semanas resultam em cerca de 650 jogos distintos; o histórico vai até 2016 (`--weeks 520` ≈ 10 anos).
- A Store aceita cerca de 200 `appdetails` a cada 5 minutos por IP; rajadas derrubam a conexão, e o cliente espera e recria a sessão antes de tentar de novo. 100 jogos novos levam cerca de 4 minutos.

## ⚖️ Alternativas consideradas
- **SteamSpy (`request=all`, ordenado por donos estimados)**: cobre dezenas de milhares de jogos, mas é fonte de terceiros com estimativas em faixas largas. Descartado em favor da API oficial.
- **Mais jogados (`ISteamChartsService/GetMostPlayedGames`)**: mede jogadores simultâneos, não vendas.
- **Busca da Store filtrada por mais vendidos**: exigiria scraping de HTML, já recusado em [[Awards da Steam e Recusa de Scraping]].

## ⚠️ Limites e conformidade
- O ranking mede **receita** semanal: jogos gratuitos com microtransações (CS2, Dota 2) aparecem no topo. `--skip-free` os deixa de fora.
- Só entram jogos que passaram pelo top 100 semanal do período. Para ir além, aumente `--weeks`.
- A posição serve só para **ordenar a coleta**, nunca como rótulo ou previsão, coerente com o [[Mudanca de Foco do Projeto]].
- Só dados de jogo: o ranking é agregado por jogo e nenhum endpoint de perfil é chamado ([[Conformidade Steam e LGPD]]).

---
*Relacionado:* [[Popular e Verificar o Banco]], [[Coleta de Reviews]], [[Schema do Banco de Dados]], [[Mudanca de Foco do Projeto]], [[Conformidade Steam e LGPD]]
