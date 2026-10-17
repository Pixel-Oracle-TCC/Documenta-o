# 🎯 Drivers de Recomendação

Reuso do Random Forest após o [[Mudanca de Foco do Projeto]]: em vez de prever vendas, o modelo explica **quais temas mais pesam para uma review ser positiva ou negativa** (`voted_up`) — uma *key driver analysis*. O rótulo é real (o próprio jogador marcou recomendo/não recomendo), sem estimativa.

## 🧮 Matriz de features
Uma linha por review da fatia **`proporcional`** (não as úteis 50/50 — ver [[Coleta de Reviews]]), já processada pelo [[Extração de Aspectos (ABSA)]]:
- **Presença de tema** (0/1) para cada tema de [[Temas Recorrentes e Metricas]] com suporte mínimo de 10 reviews.
- **Faixa de playtime no momento da review:** `<2h`, `2–10h`, `10–50h`, `>50h` (metadado da review, não do usuário).
- **Early access** (review escrita durante acesso antecipado).
- **Alvo:** `recomendada` (`voted_up`).

## 🌲 Modelo e validação
- `RandomForestClassifier(n_estimators=300, min_samples_leaf=5, class_weight="balanced")`.
- `StratifiedKFold(5)` com `f1_macro` e `roc_auc` — métricas guardadas junto aos drivers.
- **Importância:** `permutation_importance` (roc_auc, 10 repetições) num hold-out de 25%. Não usamos a importância de impureza do RF, que favorece features com muitos valores.
- **Direção:** comparamos a taxa de recomendação com e sem a feature (`taxa_recomendacao_com` / `sem`) → `aumenta` ou `reduz` a chance de recomendar.
- **Mínimos:** n ≥ 150 reviews e classe minoritária ≥ 20; gêneros abaixo disso usam o **modelo global** (`genre_id NULL`).

## 📊 Como aparece no dashboard
Seção `fatores_decisivos`: "Reviews que mencionam *bugs que travam o progresso* recomendam 38% vs 81% sem o tema". Importância e taxas vêm do modelo; a Gemini só redige (ver [[Por que Gemini so redige]]).

## 📁 Código
- `ml-service/ml/drivers.py` — `build_driver_matrix`, `train_drivers`.
- `ml-service/ml/train_drivers.py` — treina global + por gênero, grava em `genre_drivers`, registra o experimento `drivers_rf` em `/experiments/`.

## ⚠️ Limitações
- Mede **associação** dentro das reviews, não causalidade.
- Depende da qualidade do ABSA: tema não extraído = feature ausente.

---
*Relacionado:* [[Por que Random Forest]], [[Modelagem e ML]], [[Temas Recorrentes e Metricas]], [[Mudanca de Foco do Projeto]], [[Schema do Banco de Dados]]
