# 🤝 Compartilhar o Banco com o Grupo

Como o squad distribui os dados já processados (jogos, reviews, aspectos, temas, métricas, drivers) sem que cada pessoa precise rodar o pipeline de [[Popular e Verificar o Banco]] — que depende de coleta na Steam e da cota diária da Gemini.

## 📤 Exportar (quem tem os dados)
```bash
./scripts/db-export.sh --csv
```
- Sem `--csv`, gera só `dumps/pixeloracle_<data>.dump`; com ele, também `dumps/csv_<data>/<tabela>.csv`.
- No zsh, não cole comentário `#` na mesma linha do comando: sem `setopt interactivecomments`, o `#` vira argumento e o script recusa.
- `.dump` = formato custom do `pg_dump`, comprimido; inclui schema, extensão `vector` e embeddings.
- CSVs = uma planilha por tabela, **sem** colunas de vetor; útil para análise no Excel/Sheets ou para a monografia.

## 📥 Importar (quem recebe)
```bash
./scripts/db-import.sh dumps/pixeloracle_<data>.dump
docker compose restart ml-service
```
- O `restart` só é necessário se a stack já estava no ar.
- Sobe o container `db` se necessário, **apaga e recria** o banco local e restaura o dump.
- Pede confirmação (`--yes` pula); mostra as contagens no fim.

## 🛡️ Conformidade e segurança
- Nenhuma tabela tem identificador de autor de review ([[Conformidade Steam e LGPD]]) — o dump pode circular no grupo.
- O dump não contém chaves de API (`.env` fica fora do banco).
- `dumps/` está no `.gitignore`: compartilhar pelo Drive do grupo, não pelo git (arquivo binário que muda a cada exportação).

## ⚠️ Decisão descartada
Commitar o dump no repositório — descartado: incha o histórico do git a cada nova exportação e mistura dados com código.

---
*Relacionado:* [[Popular e Verificar o Banco]], [[Como Rodar o Projeto]], [[Schema do Banco de Dados]], [[Conformidade Steam e LGPD]]
