# ♊ Por que Gemini só redige

Decisão central de IA generativa: a Gemini **não analisa o pitch do zero**; ela **redige** um dashboard que o [[Motor de Triangulacao]] já montou com métricas e citações reais.

## 🎯 Problema
Prompts genéricos ("analise este jogo indie…") produzem texto persuasivo e **não auditável** — alucinação de jogos "similares", métricas inventadas. Ver [[Desafio Tecnico da Pesquisa]].

## ✅ Papéis da Gemini

| Uso | Onde | Papel |
|-----|------|-------|
| **Redação do relatório** (online) | `api/` | Resumo + reescrita da frase de cada destaque |
| **Extração ABSA** (offline) | `ml-service/absa/extractor.py` | Aspectos com trecho literal, validável com Kappa/F1 |
| **Nome dos temas** (offline) | `ml-service/themes/naming.py` | Rótulo curto a partir de resumos representativos |

Em nenhum deles a Gemini decide o que é recorrente ou importante: isso vem das métricas ([[Temas Recorrentes e Metricas]]) e do RF ([[Drivers de Recomendacao]]).

## 📌 Pontos principais no código

### Orquestração com fallback obrigatório
`api/src/application/use-cases/GenerateFullReportUseCase.ts`: `/analyze` primeiro → `narrateDashboard` → `validateNarrative`; se a Gemini falhar, `buildFallbackNarrative` usa as frases determinísticas.

### Prompt restritivo
`api/src/infrastructure/prompts/dashboardNarrativePrompt.ts`: usar só o JSON de evidência, citar só os `jogos_exemplo` de cada destaque, não inventar números; saída `{resumo, destaques[{id, frase}]}`.

### Validador pós-Gemini
`api/src/application/validation/highlightValidator.ts`: frase que cita título conhecido fora dos `jogos_exemplo` daquele destaque volta para a frase-modelo; o motivo fica em `narrativa.validacao` (auditável no TCC).

## ❌ O que foi descartado
| Abordagem | Motivo |
|-----------|--------|
| Gemini gera o relatório só com o pitch | Alucinação; sem rastreio científico |
| Gemini decide quais temas importam | Seleção precisa ser mensurável (prevalência, BH, permutation importance) |
| Confiar só no prompt para não inventar jogos | Prompt não é garantia; validador determinístico checa a saída |
| Falhar o `/api/report` se Gemini cair | Critério de aceite: fallback funcional |

---
*Relacionado:* [[Catalogo de Decisoes Tecnologicas]], [[Motor de Triangulacao]], [[Desafio Tecnico da Pesquisa]], [[API Principal Node]], [[Extração de Aspectos (ABSA)]]
