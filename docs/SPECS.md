# Super Base CNPJ — SPECS (Projeto Inteiro)

> Fonte única AGENTS.md (§1-§13). SDD Fase 1 detalhado em `docs/SDD.md`. Este doc cobre projeto inteiro: Fase 1..N, descomponível em plans.

## Visão
Base CNPJ RFB snapshot mensal + enriquecimento + gold/API/scoring para prospecção e enriquecimento de sistemas comerciais. Motor DuckDB+Parquet, sem SQL Server. Swarm VPS.

## Fases
| Fase | Nome | Entregável | Depends |
|---|---|---|---|
| 1 | Fundação | Pipeline raw→silver + current + events + Docker/cron semanal (SDD §2-§14) | — |
| 2 | Enriquecimento | Silvers externas (cada fonte = tabela própria), sem mix na Receita; merge só no gold com prioridade explícita; coleta incremental versionada | 1 |
| 3 | Gold + Views | Desnormalizada por CNPJ (1 linha/CNPJ), views por uso (prospecção por filtros, enriquecimento por CNPJ), sem API ainda | 1,2 |
| 4 | API | HTTP read-only sobre gold (DuckDB), paginação/filtros por UF/CNAE/porte/situação, rate-limit, `openapi.yaml` | 3 |
| 5 | Scoring | Score por CNPJ (ex: propensão/qualidade base), batch mensal, colunas derivadas no gold, auditável | 3 |
| 6 | Operação | Observabilidade, SLO, backup/restore, reprocess, runbooks |

Fora de Fase 1 = Fase 2..6. Postgres/ClickHouse/Redis só em fase futura se gargalo real (ponytail).

## Invariantes (de AGENTS.md/SDD §3)
- raw imutável; silver tipado + fonte/ref_mes/coletado_em; cada mês `ref=AAAA-MM/` + `current` só após validação.
- Fontes externas nunca misturadas na tabela Receita.
- Sem SQL Server nunca; Postgres/ClickHouse/Redis só com gargalo provado.
- `CGO_ENABLED=0`, DuckDB CLI via exec; `gofmt`+`go vet` limpos; reexec idempotente.

## Arquitetura pastas (projeto inteiro)
```
DATA_DIR=/data/cnpj (env)
├── raw/ref=AAAA-MM/            # ZIPs imutáveis + _manifest.json
├── silver/
│   ├── ref=AAAA-MM/            # receitas silver (SDD §5-§6)
│   ├── receita/                # alias p/ silver/ref corrente
│   └── <fonte>/ref=AAAA-MM/   # ex: silver/receita_federal/..., silver/ibge/..., silver/anatel/...
├── gold/ref=AAAA-MM/           # desnormalizada por CNPJ + _stats.json
├── current -> silver/receita/ref=AAAA-MM  # Fase1; gold tem current_gold em Fase3
├── state.json                  # {ultimo_ref, ultimo_check, gold_ref, ...}
└── tmp/                        # por etapa, limpo ao fim
```
Gold: `gold/ref=AAAA-MM/cnpj.parquet` (uma linha/CNPJ), particionado por `uf` ou `cnae_fiscal_principal` prefix se >15GB. Views: `gold/views/*.sql`.

## Config (env, cresce por fase)
Ver AGENTS.md §5 para Fase1. Fase2+ adiciona `ENRICH_SOURCES`, `ENRICH_PRIORITY`, `GOLD_PARTITION_BY`. Fase4+ `API_PORT, API_RATE_LIMIT`.

## Critérios de aceite por fase
- F1: `current` promovido só após validação (§7.3 SDD); reexec não duplica.
- F2: cada fonte externa em silver própria, com `ref_mes` e versionamento; diff mostra origem por coluna no gold.
- F3: gold = 1 linha/CNPJ, filtros UF/CNAE/porte/situação em <2s sobre Parquet 50M linhas (p95, local DuckDB).
- F4: `GET /cnpj/{cnpj}` + `POST /search` com filtros, `openapi.yaml` validado, rate-limit.
- F5: scoring determinístico, reproduzível, versionado.

## Riscos (herda SDD §13)
Mesmos + deriva de fonte externa, PII/GDPR em scoring, divergência gold vs silver.

## Mapa specs→plans
| Spec seção | Plan |
|---|---|
| Fase1 §2-§14 | `plans/001-fase1-fundacao` |
| Fase2 | `plans/002-enriquecimento` |
| Fase3 | `plans/003-gold-views` |
| Fase4 | `plans/004-api` |
| Fase5 | `plans/005-scoring` |
| Fase6 | `plans/006-operacao` |
