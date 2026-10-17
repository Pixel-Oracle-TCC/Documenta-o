# 🧭 Mudança de Foco do Projeto

Registro do escopo atual do Pixel Oracle e das decisões que o definiram, para a seção de metodologia do TCC.

## 🎯 Escopo atual
O Pixel Oracle entrega ao desenvolvedor indie um **dashboard de evidências de reviews reais** de jogos parecidos com o pitch dele:
1. **Jogos similares** ao pitch via [[Retrieval e Busca Vetorial]].
2. **Temas recorrentes** de elogio e reclamação, extraídos por [[Extração de Aspectos (ABSA)]] e agrupados com métricas de recorrência em [[Temas Recorrentes e Metricas]] (ex.: "Muitos jogadores valorizam história profunda com personagens carismáticos, como em Hollow Knight e Hades").
3. **Fatores decisivos** de recomendação: Random Forest sobre `voted_up` das reviews — [[Drivers de Recomendacao]].
4. **Reviews de referência** mais úteis de todos os tempos, com resposta do desenvolvedor quando houver ([[Awards da Steam e Recusa de Scraping]]).

A Gemini só redige o texto a partir dessas evidências, com validador de saída ([[Por que Gemini so redige]]).

## 🗑️ Score de vendas eliminado
Não há previsão de vendas nem de "chance de sucesso": vendas não são estimáveis com confiança a partir dos dados públicos.

## ❌ Alternativa descartada: gerenciamento ágil de projetos
Reformular o projeto para gestão ágil (backlog, sprints, estimativa de esforço) foi **descartado** por sobreposição com o TCC "Zênite" — mesma instituição, mesmo orientador, mesmo problema (RAG + multiagente sobre dados do GitHub). Manter esse foco comprometeria a originalidade.

## ❌ Alternativa descartada: perfilamento de usuários individuais
Cruzar o `steamid` de autores de reviews com perfil individual (biblioteca, conquistas, tempo jogado) para inferir personas foi **descartado permanentemente**:
- Viola a Cláusula 2 dos Termos de Uso da Steam Web API.
- Configura perfilamento de dado pessoal sem base legal (LGPD).

A agregação por **gênero/tema**, nunca por indivíduo, preserva o valor analítico. Ver [[Conformidade Steam e LGPD]].

## 🗄️ Infraestrutura
PostgreSQL + `pgvector` atende ao modelo relacional (jogos, gêneros, reviews, aspectos, temas, métricas, drivers) e à busca vetorial no mesmo banco. Ver [[MongoDB para PostgreSQL pgvector]].

---
*Relacionado:* [[Projeto Pixel Oracle]], [[Drivers de Recomendacao]], [[Temas Recorrentes e Metricas]], [[Motor de Triangulacao]], [[Desafio Tecnico da Pesquisa]], [[Conformidade Steam e LGPD]]
