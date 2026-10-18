# ROADMAP — Super Base CNPJ

> Sequência de fases; cada fase = um plan em `plans/NNN-titulo`.

| # | Fase | Status | Plan | Entregável chave |
|---|---|---|---|---|
| 1 | Fundação | active | `001-fase1-fundacao` | pipeline raw→silver + current |
| 2 | Enriquecimento | planned | `002-enriquecimento` | silvers externas desacopladas |
| 3 | Gold + Views | planned | `003-gold-views` | 1 linha/CNPJ + views filtros |
| 4 | API | planned | `004-api` | HTTP read-only + openapi.yaml |
| 5 | Scoring | planned | `005-scoring` | score batch mensal auditável |
| 6 | Operação | planned | `006-operacao` | SLO/backup/runbooks |

Dependências: 2,3 dependem de 1; 4,5 de 3; 6 de 1..5.
