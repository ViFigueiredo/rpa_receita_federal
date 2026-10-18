# Findings & Decisions — Fase 1

## Requirements
- 7 etapas SDD §2; sem gold/API/fontes externas/scoring nesta fase
- Motor DuckDB+Parquet; sem SQL Server/Postgres/ClickHouse/Redis
- Idempotente, resume Range, zstd, particionado por UF, current só após validação

## Research Findings
- Layout oficial: https://www.gov.br/receitafederal/dados/cnpj-metadados.pdf (NOVO LAYOUT, 6 pág) — colunas §4.1 SDD
- Base primária: https://dadosabertos.rfb.gov.br/CNPJ/ — fallback Nextcloud YggdBLfdninEJX9
- CSV: sem header, ; , ISO-8859-1, 00000000=NULL, decimal vírgula, CPF mascarado

## Technical Decisions
| Decision | Rationale |
|----------|-----------|
| DuckDB CLI (exec) | Sem CGO, imagem menor, falha isolada |
| CGO_ENABLED=0 | Swarm enxuto |
| PARTITION_BY uf para estabelecimentos | SDD §5/§6 |

## Issues Encountered
| Issue | Resolution |
|-------|------------|
|       |            |

## Resources
- docs/SDD.md — SDD canônico
- AGENTS.md — regras mínimas
- https://www.gov.br/receitafederal/dados/cnpj-metadados.pdf
- https://dadosabertos.rfb.gov.br/CNPJ/

## Visual/Browser Findings
-
