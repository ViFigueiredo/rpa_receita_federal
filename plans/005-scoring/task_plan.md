# Task Plan: Fase 5 — Scoring

## Goal
Scoring batch mensal por CNPJ, determinístico e versionado, colunas no gold, auditável.

## Next Step
Definir features e fórmula v1 com Vi (o que pontua, pesos, audit).

## Current Phase
Phase 1

## Phases
### Phase 1: Spec scoring
- [ ] Features, pesos v1, saída (coluna + tabela audit), versionamento
- **Status:** pending

### Phase 2: Implementação
- [ ] `internal/scoring` Go + DuckDB, batch sobre gold/ref
- **Status:** pending

### Phase 3: Validação
- [ ] Reprodutível (mesmo input → mesmo score), `go test` com casos
- **Status:** pending

### Phase 4: Entrega
- [ ] Docs + `go vet/test`
- **Status:** pending

## Key Questions
1. Features v1? 2. Saída só coluna gold ou tabela scoring separada?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| Scoring no gold mas batch separado, auditável | SPECS Fase 5 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Spec projeto inteiro: `docs/SPECS.md` + `docs/ROADMAP.md` + `docs/SDD.md` (Fase 1 detalhada).
- Depende de gold (Fase 3).
