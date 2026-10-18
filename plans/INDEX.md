# plans/INDEX — Super Base CNPJ

> Cada pasta `NNN-titulo/` é um plan rastreável. Fonte specs: `docs/SPECS.md` + `docs/SDD.md` (Fase 1) + `docs/ROADMAP.md`. Qualquer agente abre o `task_plan.md` do plan ativo e sabe onde estamos.

| Plan | Fase | Status | Entrada |
|---|---|---|---|
| `001-fase1-fundacao` | Fase 1 — Fundação | `active` | `plans/001-fase1-fundacao/task_plan.md` |
| `002-enriquecimento` | Fase 2 — Enriquecimento | `planned` | `plans/002-enriquecimento/task_plan.md` |
| `003-gold-views` | Fase 3 — Gold + Views | `planned` | `plans/003-gold-views/task_plan.md` |
| `004-api` | Fase 4 — API | `planned` | `plans/004-api/task_plan.md` |
| `005-scoring` | Fase 5 — Scoring | `planned` | `plans/005-scoring/task_plan.md` |
| `006-operacao` | Fase 6 — Operação | `planned` | `plans/006-operacao/task_plan.md` |

## Como usar (para qualquer agente)
1. Ler `plans/INDEX.md` para achar o plan ativo (`active`).
2. Entrar no `plans/<ativo>/task_plan.md` — `Current Phase` + `Next Step` dizem o que fazer.
3. `progress.md` do plan registra o que já foi feito; `findings.md` registra decisões/evidências.
4. `docs/SPECS.md` = spec projeto inteiro; `docs/SDD.md` = Fase 1 detalhada; `docs/ROADMAP.md` = sequência.

## Criar novo plan
Duplicar template: `cp -r plans/001-fase1-fundacao plans/00N-novo --no-clobber` e editar `task_plan.md`.
