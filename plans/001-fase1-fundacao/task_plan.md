# Task Plan: Fase 1 — Fundação Super Base CNPJ

## Goal
Pipeline Fase 1 operando fim-a-fim (descoberta → download → conversão → validação → diff → Docker/cron) validado com dados reais da RFB, idempotente e promovendo `current` só após validação.

## Next Step
Etapa 1 — Descoberta: implementar `internal/discover` + `internal/state` + CLI `discover`, validar contra URLs reais (primária + fallbacks), persistir `state.json`.

## Current Phase
Phase 1

## Phases
### Phase 1: Descoberta & Estado
- [ ] `internal/config` (env defaults)
- [ ] `internal/state` (load/save atomico state.json, symlink current)
- [ ] `internal/discover` (lista HTTP, extrai YYYY-MM, compara vs state, ordem primária→fallbacks)
- [ ] `cmd/cnpj-pipeline` subcomando `discover`
- [ ] Validar contra URLs reais; teste `httptest` para HTML parsing
- **Status:** pending

### Phase 2: Downloader
- [ ] Downloader paralelo 4-6, Range resume, backoff, checksum, idempotente
- [ ] Validar com 1 ZIP real (Cnaes/Motivos pequeno primeiro)
- **Status:** pending

### Phase 3: Conversão CSV→Parquet
- [ ] `internal/convert` via DuckDB CLI (columns explícito, 00000000→NULL, vírgula→ponto)
- [ ] Estabelecimentos PARTITION_BY UF
- [ ] Validar com CSV real de 1 fonte
- **Status:** pending

### Phase 4: Silver + Validação
- [ ] Layout silver completo (tipos, zstd, fonte/ref_mes/coletado_em)
- [ ] Validação pré-promoção (contagens, nulos%, unicidade) + _stats.json
- **Status:** pending

### Phase 5: Diff & Promoção
- [ ] _eventos_mudanca.parquet (ANTI/FULL JOIN)
- [ ] promote (ln -sfn current)
- **Status:** pending

### Phase 6: Docker & Cron
- [ ] Dockerfile multi-stage (CGO_ENABLED=0, duckdb bin)
- [ ] Job semanal (checagem; pipeline quando mês novo)
- **Status:** pending

### Phase 7: Entrega Fase 1
- [ ] go vet/test limpos, baseline §9 AGENTS.md atualizado
- [ ] Revisão SDD/AGENTS se divergir
- **Status:** pending

## Key Questions
1. URL primária real lista YYYY-MM ou exige parse HTML Nextcloud? → probe real na Phase 1
2. DuckDB binário versão fixa? → definir no Dockerfile Phase 6

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| DuckDB CLI via exec, sem CGO | Build reproduzível, falha isolada, imagem menor (SDD §6) |
| DATA_DIR=/data/cnpj, ref=AAAA-MM/, current symlink | Snapshot imutável + promoção atômica (SDD §5) |
| Thresholds empresas ±3% / estab,socios ±5% | SDD §7.4; override FORCE_PROMOTE=1 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
|       |         |            |

## Notes
- Fonte única regras: AGENTS.md. Arquitetura: docs/SDD.md. Este plan é o roadmap vivo — atualizar status ao avançar.
- Fora escopo Fase 1: gold, fontes externas, API, scoring, Postgres/ClickHouse/Redis.
