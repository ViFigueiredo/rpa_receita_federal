# Task Plan: Fase 2 — Enriquecimento (fontes externas)

## Goal
Fontes externas de enriquecimento ingestadas como silvers próprias (sem mix na Receita), versionadas por ref_mes, com regras de prioridade explícitas documentadas para o gold.

## Next Step
Definir catálogo de fontes Fase 2 (quais, licença, URL, atualização, PII), aprovar com Vi antes de implementar.

## Current Phase
Phase 1

## Phases
### Phase 1: Catálogo & Contrato
- [ ] Listar fontes candidatas (IBGE, ANATEL, etc.) com licença/URL/frequência/PII
- [ ] Aprovar lista com Vi; registrar em `docs/SPECS.md` Fase 2
- [ ] Definir `silver/<fonte>/ref=AAAA-MM/` + `fonte/ref_mes/coletado_em` + versionamento
- **Status:** pending

### Phase 2: Coletor genérico
- [ ] `internal/enrich/collector` genérico (HTTP + retry + checksum + raw imutável)
- [ ] `internal/enrich/<fonte>` adapters (1 por fonte)
- **Status:** pending

### Phase 3: Silver externa
- [ ] CSV/JSON→Parquet via DuckDB CLI (mesmo padrão SDD §7.3), tipado + zstd
- [ ] Particionamento por fonte (não por UF se não aplicar)
- **Status:** pending

### Phase 4: Regras de prioridade
- [ ] `docs/enrich-priority.md` — por coluna, qual fonte vence no gold
- [ ] Validação cross-fonte (conflitos, nulos, freshness)
- **Status:** pending

### Phase 5: Entrega
- [ ] `go vet/test`, AGENTS.md §9 baseline, docs atualizados
- **Status:** pending

## Key Questions
1. Quais fontes entram na Fase 2 (aprovar)? 2. Prioridade por coluna no gold?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| Cada fonte = silver própria, nunca mix na Receita | SDD §3 + SPECS Fase 2 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Spec projeto inteiro: `docs/SPECS.md` + `docs/ROADMAP.md` + `docs/SDD.md` (Fase 1 detalhada).
- Bloqueado por Fase 1 (precisa silver Receita estável).
