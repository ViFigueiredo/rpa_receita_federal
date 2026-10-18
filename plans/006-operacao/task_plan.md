# Task Plan: Fase 6 — Operação (SLO/backup/runbooks)

## Goal
Operação: observabilidade, SLO, backup/restore, reprocess, runbooks para Swarm.

## Next Step
Definir SLO (disponibilidade, atraso ref) e estratégia backup (o que, onde, retenção).

## Current Phase
Phase 1

## Phases
### Phase 1: Observabilidade
- [ ] Métricas/logs por etapa + alertas (quando, onde, quem)
- **Status:** pending

### Phase 2: Backup/Restore & Reprocess
- [ ] Backup `raw`/`silver`/`gold` + restore testado, reprocess por ref
- **Status:** pending

### Phase 3: Runbooks
- [ ] `docs/runbooks/*.md` + Docker Swarm `replicas/deploy`
- **Status:** pending

### Phase 4: Entrega
- [ ] `go vet/test`, AGENTS atualizado
- **Status:** pending

## Key Questions
1. Backup: só raw ou silver/gold também? 2. Retenção?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| Operação por último (depende de todas) | SPECS Fase 6 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Spec projeto inteiro: `docs/SPECS.md` + `docs/ROADMAP.md` + `docs/SDD.md` (Fase 1 detalhada).
- Fecha o ciclo.
