# 🎨 Design System PixelOracle

Identidade visual dark do front Angular: fundo `#080c14`, acentos blue `#38bdf8` e purple `#818cf8`, tipografia **SF Pro / system** (Display + Text + Mono). Tokens globais em `src/styles/_tokens.scss`.

## 🎯 Princípios
- Linguagem visual dark, glow blue/purple, pills, cards com borda sutil.
- Copy centrada em **evidência de reviews reais**; nenhum score ou chance de sucesso.
- Narrativa legível + evidência auditável (métricas, citações) — duas camadas no [[Fluxo Analisar Pitch]].
- Empty states honestos quando não houver reviews analisadas.

## 🎨 Tokens essenciais
- Fundo: `--bg-base` `#080c14`, `--bg-card` `#111827`, `--bg-surface` `#0d1526`
- Acentos: `--blue` `#38bdf8`, `--purple` `#818cf8`, `--teal` `#2dd4bf`
- Texto: `--text-primary` `#f1f5f9`, `--text-secondary` `#94a3b8`
- CTA: gradiente `135deg` blue → purple, texto branco, glow azul

## 🚦 Cores do dashboard
- Elogio recorrente → verde
- Alerta (`primeiras_horas`, `early_access`, `nicho_exigente`) → âmbar
- Reclamação recorrente → vermelho

## 🧩 Componentes compartilhados
`HighlightCard`, `DecisiveFactorList`, `ReferenceReviewCard`, `EvidencePanel`, `EmptyState`, `LoadingState`, `ApiBanner`, `SiteHeader`.

---
*Relacionado:* [[Arquitetura do Frontend]], [[Fluxo Analisar Pitch]], [[Guia de Atualizacao do Frontend]]
