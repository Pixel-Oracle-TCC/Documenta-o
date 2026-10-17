# 🌲 Por que Random Forest

Por que o modelo supervisionado do projeto é um `RandomForestClassifier` (scikit-learn), usado para explicar o que leva uma review a recomendar ou não um jogo ([[Drivers de Recomendacao]]).

## ✅ Por que Random Forest
- Features binárias/categóricas (presença de tema, faixa de playtime, early access) sem necessidade de normalização.
- Captura interações (ex.: "bugs" pesa mais em reviews de quem jogou pouco) sem especificá-las à mão.
- `class_weight="balanced"` lida com o desbalanceamento natural (a maioria das reviews é positiva).
- Combinado com `permutation_importance`, gera um ranking de drivers explicável na monografia.
- Rótulo real (`voted_up`), sem estimativa — ao contrário de vendas ([[Mudanca de Foco do Projeto]]).

## ❌ Alternativas descartadas
| Alternativa | Motivo |
|-------------|--------|
| Rede neural profunda | Amostra pequena por gênero; pouco interpretável |
| Regressão logística pura | Útil como baseline, mas não captura interações entre temas |
| Importância por impureza (MDI) | Enviesada para features com mais valores; trocada por permutation importance |
| Gradient boosting | Ganho marginal esperado não compensa mais hiperparâmetros para justificar |

## 📌 Código
`ml-service/ml/drivers.py` e `ml-service/ml/train_drivers.py`.

---
*Relacionado:* [[Drivers de Recomendacao]], [[Modelagem e ML]], [[Catalogo de Decisoes Tecnologicas]]
