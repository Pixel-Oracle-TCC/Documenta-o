# 🔬 Resumo do Foco da Pesquisa

Visão de uma página do Pixel Oracle para quem precisa entender o projeto rápido: qual pergunta a pesquisa responde, o que o sistema faz, o que coleta (e o que deliberadamente não coleta), quais métodos usa e em que estado está. Os detalhes de cada parte ficam nas notas linkadas.

## ❓ Pergunta de pesquisa
É possível dar ao desenvolvedor indie, **antes do lançamento**, uma mentoria fundamentada em evidência verificável — o que jogadores elogiam e reclamam de forma recorrente em jogos parecidos com o dele — sem previsões inventadas e sem alucinação de IA generativa?

- **Hipótese:** reviews públicas da Steam, analisadas por aspecto e agregadas por gênero e tema, revelam padrões recorrentes e mensuráveis de satisfação e insatisfação, e esses padrões explicam parte da decisão de recomendar um jogo.
- **Não é objetivo:** prever vendas ou "chance de sucesso" — vendas não são estimáveis com confiança a partir de dados públicos ([[Mudanca de Foco do Projeto]]).
- **Desafio central:** fundamentar cada afirmação em dado rastreável ([[Desafio Tecnico da Pesquisa]]).

## 🎮 O que o sistema faz
O desenvolvedor envia um **pitch** (título, descrição, gêneros) e recebe um **dashboard de evidências**:
1. **Jogos similares** ao pitch, por busca semântica ([[Retrieval e Busca Vetorial]]).
2. **Destaques:** elogios e reclamações recorrentes nesses jogos e no gênero, cada um com prevalência, jogos-exemplo e citações literais ([[Temas Recorrentes e Metricas]]).
3. **Fatores decisivos:** quais temas mais aumentam ou reduzem a chance de uma review recomendar o jogo ([[Drivers de Recomendacao]]).
4. **Alertas:** reclamações concentradas nas primeiras horas de jogo, em acesso antecipado, ou nicho com avaliação baixa.
5. **Reviews de referência:** as mais úteis de todos os tempos, com resposta do desenvolvedor quando houver.

A Gemini só **redige** o texto final a partir dessas evidências, e um validador descarta frases que citem jogos fora da evidência ([[Por que Gemini so redige]], [[Motor de Triangulacao]]).

## 📥 O que coleta
Só dados públicos de **jogos** e **reviews**, só pela API oficial da Steam:
- **Catálogo:** jogos na ordem dos mais vendidos, a partir do top 100 semanal oficial da Steam somado em 52 semanas ([[Catalogo pelos Mais Vendidos]]); metadados do `appdetails` (descrição, preço em reais, ano, gêneros).
- **Reviews por jogo:** até 40 positivas e 40 negativas mais úteis de todos os tempos e 20 recentes, em português e inglês ([[Coleta de Reviews]]).
- **Campos da review:** texto, recomendou ou não, votos "útil" e "engraçado", horas jogadas no momento da review, idioma, data, acesso antecipado, compra na Steam e resposta do desenvolvedor.
- **Resumo público do jogo:** total de reviews positivas e negativas e a avaliação da Steam ("Muito positivas" etc.).

## 🚫 O que não coleta
- **Nenhum dado de autor:** sem `steamid`, perfil, biblioteca, conquistas ou amigos — Termos da Steam Web API (cláusula 2) e LGPD ([[Conformidade Steam e LGPD]]).
- **Nenhuma estimativa de vendas ou de donos:** a estimativa Boxleiter e o SteamSpy foram removidos do projeto.
- **Nenhum scraping de HTML:** Awards da Steam, busca da Store e SteamDB ficam de fora ([[Awards da Steam e Recusa de Scraping]]).
- **Histórico de preços:** a API oficial só informa o preço atual; sem fonte oficial, ficou fora do escopo.

## 🔄 Pipeline
Coleta do catálogo → embeddings dos jogos → coleta de reviews → extração de aspectos pela Gemini → clusterização em temas → métricas por gênero → Random Forest de drivers → dashboard. Comandos em [[Popular e Verificar o Banco]].

## 📐 Métodos
| Etapa | Método |
|---|---|
| Similaridade | Embeddings `all-MiniLM-L6-v2` + cosseno no `pgvector` |
| Aspectos | ABSA com Gemini (JSON, temperatura 0), 15 categorias fixas, trecho literal verificado em código |
| Temas | `AgglomerativeClustering` sobre resumos em embedding multilíngue; qualidade por silhouette |
| Recorrência | Prevalência com intervalo de Wilson 95%, mínimo de 2 jogos distintos |
| Associação com recomendação | Qui-quadrado 2×2, log-odds (Haldane-Anscombe), correção de Benjamini-Hochberg |
| Fatores decisivos | Random Forest sobre `voted_up` + permutation importance em hold-out ([[Por que Random Forest]]) |
| Validação da ABSA | Gold standard anotado por dois humanos: Kappa entre anotadores e F1 da Gemini |

Cada execução registra parâmetros e métricas em `/experiments/` para reprodutibilidade ([[Modelagem e ML]]).

## 🏗️ Arquitetura
Frontend Angular → API Node/Express (validação, Gemini, validador) → ml-service FastAPI (busca, motor de triangulação) → PostgreSQL + `pgvector`, tudo em Docker ([[Arquitetura de Microsservicos]], [[Schema do Banco de Dados]]).

## 📊 Estado atual
- **Banco:** 125 jogos (110 da base inicial + 15 mais vendidos), 492 reviews de 12 jogos, 625 aspectos extraídos (98,6% com trecho literal).
- **Pendente:** coletar reviews do restante do catálogo, terminar a ABSA (limitada pela cota gratuita da Gemini), gerar temas, métricas e drivers, rodar o fluxo ponta a ponta e fazer a anotação humana do gold standard ([[Proximos Passos do Pipeline]]).

## ⚠️ Limitações
- Mede **associação** nas reviews, não causalidade.
- Reviews da Steam são autosselecionadas: refletem quem escolheu avaliar, não todos os jogadores.
- A qualidade dos temas depende da extração da Gemini — por isso o gold standard humano.
- O catálogo cobre os mais vendidos do último ano no Brasil; jogos de nicho fora do top 100 semanal só entram pela base inicial.

---
*Relacionado:* [[Projeto Pixel Oracle]], [[Mudanca de Foco do Projeto]], [[Desafio Tecnico da Pesquisa]], [[Conformidade Steam e LGPD]], [[Motor de Triangulacao]], [[Proximos Passos do Pipeline]]
