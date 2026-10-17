# 🔎 Por que MiniLM e pgvector

Por que o RAG de jogos similares usa `all-MiniLM-L6-v2` + coluna `VECTOR(384)` no PostgreSQL, e não outro embedding store.

## 🎯 Necessidade
Dado um pitch (título + descrição + gêneros), recuperar os **k jogos históricos mais próximos semanticamente** para alimentar o [[Motor de Triangulacao]] — evidência verificável, não “jogos que a LLM inventou”.

## ✅ Embedding: `sentence-transformers` / `all-MiniLM-L6-v2`
- 384 dimensões — cabe em `VECTOR(384)` do schema.
- Leve o bastante para rodar no mesmo container do ml-service (TCC / free tier).
- Bom custo/benefício para similaridade de texto curto (nome + descrição de jogo).
- Configurável via `EMBEDDING_MODEL` sem hardcode absoluto no retrieve.

**Onde no código:**
- `ml-service/embeddings/encoder.py`
  - `DEFAULT_MODEL = "all-MiniLM-L6-v2"`
  - `EMBEDDING_DIM = 384`
  - `build_text(titulo, descricao, genres)` → string única antes do encode
  - `normalize_embeddings=True` (cosseno coerente com `<=>` do pgvector)
- Backfill: `ml-service/scripts/backfill_embeddings.py`

## ✅ Busca: **pgvector** no mesmo Postgres
- Evita um segundo produto (Pinecone, Qdrant, Atlas Vector) só para k-NN.
- SQL legível e auditável no TCC: `ORDER BY embedding <=> CAST(:query AS vector)`.
- Similaridade exposta como `1 - distância` em `retrieval/similar_games.py`.

**Onde no código:**
- Query: `ml-service/retrieval/similar_games.py` → `SimilarGamesRetriever.retrieve`
- Schema: `ml-service/db/migrations/001_schema.sql` (`embedding VECTOR(384)`)
- ORM: `ml-service/models/orm.py` → `Game.embedding`
- Endpoint: `POST /retrieve-similar` e uso interno em `application/analyze.py`
- Aspectos de review usam outro modelo da mesma família, multilíngue (`paraphrase-multilingual-MiniLM-L12-v2`, também 384d), para PT e EN caírem no mesmo espaço — ver [[Temas Recorrentes e Metricas]]

## ❌ Alternativas descartadas
| Alternativa | Motivo |
|-------------|--------|
| OpenAI / Gemini embeddings só na nuvem | Custo + dependência de rede no caminho crítico; MiniLM roda local |
| Modelos maiores (mpnet, e5-large) | Mais RAM/latência; ganho incerto na amostra atual |
| FAISS em arquivo separado | Mais uma fonte de verdade; divergência do Postgres relacional |
| MongoDB Atlas Vector Search | Ver [[MongoDB para PostgreSQL pgvector]] |

## ⚠️ Cuidados operacionais
- Índice `ivfflat` só **depois** de popular embeddings (comentado no SQL de init).
- Primeira carga do SentenceTransformer é lenta — esperado no primeiro request.
- Sem backfill, `/retrieve-similar` devolve lista vazia (não é “bug de API”).

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Retrieval e Busca Vetorial]], [[MongoDB para PostgreSQL pgvector]], [[Motor de Triangulacao]]
