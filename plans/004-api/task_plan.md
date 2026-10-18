# Task Plan: Fase 4 — API HTTP read-only

## Goal
API HTTP read-only sobre gold: `GET /cnpj/{cnpj}` e `POST /search` com filtros/paginação/rate-limit e `openapi.yaml` validado.

## Next Step
Definir contrato OpenAPI (filtros, paginação, erros) e aprovar antes de código.

## Current Phase
Phase 1

## Phases
### Phase 1: Contrato
- [ ] `openapi.yaml` — filtros UF/CNAE/porte/situação, paginação, erros
- **Status:** pending

### Phase 2: Implementação
- [ ] Go HTTP (`net/http` stdlib), DuckDB queries parametrizadas, rate-limit
- **Status:** pending

### Phase 3: Teste & Docs
- [ ] `go test` + `openapi.yaml` lint/validate, `docs/API.md`
- **Status:** pending

### Phase 4: Entrega
- [ ] Docker, `go vet/test`, AGENTS §9
- **Status:** pending

## Key Questions
1. Auth da API (nenhum, apikey, JWT)? 2. Rate-limit por IP/chave?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| API só após gold (Fase 3) | SPECS Fase 4 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Spec projeto inteiro: `docs/SPECS.md` + `docs/ROADMAP.md` + `docs/SDD.md` (Fase 1 detalhada).
- Sem gold não há API.
