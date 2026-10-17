# 🐘 PostgreSQL no Cursor e VS Code

Como o squad inspeciona o banco do Pixel Oracle pela extensão **PostgreSQL** da Microsoft, sem sair do editor. Serve para Cursor e VS Code (a UI é a mesma).

## 📦 Extensão
1. Abra Extensions (`Cmd+Shift+X`).
2. Busque **PostgreSQL** (publicador **Microsoft**, id `ms-ossdata.vscode-pgsql` / no Cursor aparece como `microsoft.cursor-pgsql`).
3. Instale e recarregue a janela se pedir.

Não use Adminer/pgAdmin se a extensão já atender — um só caminho evita confusão no squad.

## 🔌 Criar a conexão (Pixel Oracle)
1. Clique no ícone do **elefante** na barra lateral esquerda (PostgreSQL).
2. **Add Connection** (`+`).
3. Em Parameters, preencha:

| Campo | Valor |
|---|---|
| Server name / Host | `localhost` |
| Port | `5433` (não use 5432 — é outro Postgres, se houver) |
| Authentication | Password |
| User name | `pixel` |
| Password | `pixel` (marque Save password) |
| Database name | `pixeloracle` |
| Connection name | `Pixel Oracle` (opcional) |

4. Em Advanced, se der erro de SSL: **SSL mode = disable** (banco local no Docker).
5. **Connect**.

O container precisa estar no ar: `docker compose up -d db` ([[Como Rodar o Projeto]]).

## 👀 Ver tabelas e dados
1. Expanda **Pixel Oracle → Databases → pixeloracle → Schemas → public → Tables**.
2. Botão direito numa tabela (ex.: `reviews`, `games`) → **Select Top 1000**.
3. O resultado abre em grade; dá para ordenar e filtrar pela UI.

Tabelas principais e colunas abaixo (fonte: migrations `001`–`007` + [[Schema do Banco de Dados]]). Reviews **não** têm coluna de autor ([[Conformidade Steam e LGPD]]).

Na extensão: botão direito na tabela → **Show Table Definition** (ou no `psql`: `\d nome_da_tabela`).

## 📋 Colunas por tabela

### `games`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `steam_appid` | INT UNIQUE | AppID Steam |
| `titulo` | TEXT NOT NULL | |
| `descricao` | TEXT | |
| `preco_brl` | NUMERIC | |
| `release_year` | INT | |
| `release_date` | DATE | |
| `rank_popularidade` | INT | 1 = mais vendido no ranking |
| `raw_metadata` | JSONB | |
| `embedding` | VECTOR(384) | busca vetorial |
| `total_positive` / `total_negative` / `total_reviews_steam` | INT | agregados Steam |
| `review_score` | INT | score 1–9 |
| `review_score_desc` | TEXT | ex.: "Very Positive" |
| `reviews_collected_at` | TIMESTAMP | fim da coleta de reviews |
| `na_amostra` | BOOLEAN | flag amostra estratificada |
| `elegivel_metricas` | BOOLEAN | fora se poucas reviews Steam |
| `created_at` | TIMESTAMP | |

### `apps_ignorados`
| Coluna | Tipo | Notas |
|---|---|---|
| `steam_appid` | INT PK | |
| `titulo` | TEXT | |
| `motivo` | TEXT NOT NULL | software, DLC, fora Store BR… |
| `created_at` | TIMESTAMP | |

### `genres` / `categories` / `developers` / `publishers` / `languages`
| Tabela | Colunas |
|---|---|
| `genres` | `id`, `nome` (UNIQUE) |
| `categories` | `id`, `steam_category_id` (UNIQUE), `nome` |
| `developers` | `id`, `nome` (UNIQUE) |
| `publishers` | `id`, `nome` (UNIQUE) |
| `languages` | `id`, `nome` (UNIQUE) |

### Junções N:N
| Tabela | Colunas (PK composta) |
|---|---|
| `game_genres` | `game_id` → games, `genre_id` → genres |
| `game_categories` | `game_id`, `category_id` |
| `game_developers` | `game_id`, `developer_id` |
| `game_publishers` | `game_id`, `publisher_id` |
| `game_languages` | `game_id`, `language_id` |

### `reviews`
Sem coluna de autor / `steamid`. Dedup por `recommendationid`.

| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `game_id` | INT FK → games | |
| `recommendationid` | TEXT UNIQUE | chave de dedup |
| `texto` | TEXT | |
| `recomendada` | BOOLEAN | |
| `votos_uteis` | INT | votes_up |
| `votes_funny` | INT | |
| `weighted_vote_score` | NUMERIC | |
| `comment_count` | INT | |
| `playtime_at_review` | INT | minutos; não identifica pessoa |
| `language` | TEXT | |
| `timestamp_created` | BIGINT | Unix |
| `early_access` | BOOLEAN | |
| `steam_purchase` | BOOLEAN | |
| `received_for_free` | BOOLEAN | |
| `developer_response` | TEXT | |
| `timestamp_dev_responded` | BIGINT | |
| `fatias` | TEXT[] | `proporcional`, `util_positiva`, `util_negativa` |
| `tipo_amostra` | TEXT | legado; preferir `fatias` |
| `absa_processed_at` | TIMESTAMP | |
| `absa_model` | TEXT | |
| `created_at` | TIMESTAMP | |

### `review_snapshots`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `game_id` | INT FK → games | |
| `total_positive` / `total_negative` / `total_reviews` | INT | com filtro offtopic |
| `review_score` / `review_score_desc` | INT / TEXT | |
| `total_positive_unfiltered` / `total_negative_unfiltered` / `total_reviews_unfiltered` | INT | sem filtro offtopic |
| `collected_at` | TIMESTAMP | |

### `themes`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `categoria` | TEXT | |
| `polaridade` | TEXT | `positivo` / `negativo` |
| `rotulo` | TEXT | |
| `tamanho` | INT | |
| `centroide` | VECTOR(384) | |
| `rotulo_fonte` | TEXT | `gemini` / `representante` |
| `created_at` | TIMESTAMP | |

### `review_aspects`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `review_id` | INT FK → reviews | |
| `categoria` | TEXT | |
| `sentimento` | TEXT | `positivo` / `negativo` / `neutro` |
| `trecho` | TEXT | citação |
| `trecho_literal` | BOOLEAN | só literal vira citação |
| `resumo` | TEXT | |
| `embedding` | VECTOR(384) | |
| `theme_id` | INT FK → themes | |
| `created_at` | TIMESTAMP | |

### `theme_stats`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `genre_id` / `theme_id` | INT FK | UNIQUE (genre, theme) |
| `reviews_com_tema` / `total_reviews` | INT | |
| `prevalencia` | NUMERIC | |
| `wilson_low` / `wilson_high` | NUMERIC | |
| `jogos_distintos` / `total_jogos` | INT | |
| `votos_uteis` | INT | |
| `pct_recomendadas_com_tema` | NUMERIC | |
| `p_value` / `p_value_ajustado` | NUMERIC | BH por gênero |
| `log_odds` | NUMERIC | |
| `significativo` | BOOLEAN | |
| `jogos_exemplo` / `citacoes` / `contexto` | JSONB | |
| `updated_at` | TIMESTAMP | |

### `genre_drivers`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `genre_id` | INT FK (NULL = global) | |
| `feature` / `rotulo` / `tipo` | TEXT | tipo: `tema` / `contexto` |
| `importancia` / `importancia_std` | NUMERIC | |
| `direcao` | TEXT | `aumenta` / `reduz` |
| `taxa_recomendacao_com` / `taxa_recomendacao_sem` | NUMERIC | |
| `suporte` | INT | |
| `modelo_f1` / `modelo_roc_auc` | NUMERIC | |
| `n_amostras` | INT | |
| `updated_at` | TIMESTAMP | |

### `absa_gold_standard`
| Coluna | Tipo | Notas |
|---|---|---|
| `id` | SERIAL PK | |
| `review_id` | INT UNIQUE FK → reviews | |
| `anotador1` / `anotador2` / `consenso` / `gemini` | JSONB | listas `{categoria, sentimento}` |
| `updated_at` | TIMESTAMP | |

## ✍️ Rodar SQL
1. Botão direito no banco `pixeloracle` → **New Query** (ou paleta: “PostgreSQL: New Query”).
2. Escreva o SQL e execute com `Cmd+Shift+E` ou o botão ▶.

Consultas úteis:

```sql
-- Progresso da coleta de reviews
SELECT
  count(*) FILTER (WHERE reviews_collected_at IS NOT NULL) AS coletados,
  count(*) AS total_jogos,
  (SELECT count(*) FROM reviews) AS reviews
FROM games;

-- Reviews recentes com nome do jogo
SELECT g.titulo, left(r.texto, 120) AS trecho, r.recomendada,
       r.votos_uteis, r.language, r.fatias
FROM reviews r
JOIN games g ON g.id = r.game_id
ORDER BY r.id DESC
LIMIT 50;

-- Fatias da amostra
SELECT unnest(fatias) AS fatia, count(*) FROM reviews GROUP BY 1;
```

Mais queries de pipeline: [[Popular e Verificar o Banco]], [[Coleta de Reviews]].

## ⚠️ Problemas comuns
| Sintoma | Causa | O que fazer |
|---|---|---|
| Connection refused | Docker/db parado | `docker compose up -d db` |
| Conectou em banco vazio | Porta 5432 errada | Use **5433** |
| SSL / certificate error | Cliente exige SSL | Advanced → SSL mode `disable` |
| `relation "reviews" does not exist` | Migrations não aplicadas | `cd ml-service && python scripts/apply_schema.py` |
| Senha rejeitada | Credencial errada | user/senha/db = `pixel` / `pixel` / `pixeloracle` |

## 🖥️ Alternativa no terminal
Se a extensão falhar:

```bash
docker exec -it pixel-oracle-db psql -U pixel -d pixeloracle
```

Dentro do `psql`: `\dt` lista tabelas, `\d reviews` mostra colunas, `\q` sai.

---
*Relacionado:* [[Como Rodar o Projeto]], [[Schema do Banco de Dados]], [[Popular e Verificar o Banco]], [[Coleta de Reviews]], [[Conformidade Steam e LGPD]]
