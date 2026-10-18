# Task Plan: Fase 3 — Gold desnormalizada + Views

## Goal
Tabela gold desnormalizada 1 linha/CNPJ (silver Receita + silvers enriquecimento) com views para prospecção por filtros e enriquecimento por CNPJ.

## Next Step
Desenhar schema gold (colunas, chaves, partição) e validar p95 <2s sobre Parquet 50M linhas.

## Current Phase
Phase 1

## Phases
### Phase 1: Schema gold
- [ ] `gold/ref=AAAA-MM/cnpj.parquet` — 1 linha/CNPJ, colunas derivadas, prioridade por fonte
- [ ] Partição (`uf` ou `cnae` prefix se >15GB) + zstd
- **Status:** pending

### Phase 2: Build gold
- [ ] DuckDB SQL: JOIN silvers → gold (idempotente, `tmp`+`rename`)
- [ ] `gold/views/*.sql` — prospecção (UF/CNAE/porte/situação) + enriquecimento por CNPJ
- **Status:** pending

### Phase 3: Validação
- [ ] Contagens vs silver, unicidade CNPJ, benchmark p95
- **Status:** pending

### Phase 4: Entrega
- [ ] `go vet/test`, docs AGENTS/SPECS/ROADMAP atualizados
- **Status:** pending

## Key Questions
1. Partição gold por UF ou CNAE? → medir tamanho Fase1. 2. Views mínimas Fase 3?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| Gold só após Fase 2 (precisa prioridades) | SPECS Fase 3 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Spec projeto inteiro: `docs/SPECS.md` + `docs/ROADMAP.md` + `docs/SDD.md` (Fase 1 detalhada).
- Depende de 001 + 002.
