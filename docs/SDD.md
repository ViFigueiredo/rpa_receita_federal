# Super Base CNPJ — SDD (Fase 1)

> Status: draft — aguardando execução por etapas. Stack: Go + DuckDB CLI + Parquet zstd. Sem Postgres/ClickHouse/Redis nesta fase.

## 1 Visão
Base CNPJ Receita Federal, snapshot mensal completo, enriquecimento futuro desacoplado. Consumo: prospecção e enriquecimento. Host: VPS Docker Swarm.

## 2 Escopo Fase 1 — só isto
1. Descoberta mês mais recente + diff vs `state.json`
2. Downloader Go: paralelo 4-6, resume Range, retry backoff, checksum ZIP, idempotente
3. CSV→Parquet via DuckDB CLI (`read_csv` latin-1, `;`, sem header, `zstd`)
4. Silver tipado, `fonte/ref_mes/coletado_em`, estabelecimentos particionado por UF
5. Validação pré-promoção: contagem vs mês anterior, nulos%, unicidade CNPJ — só então atualiza `current`
6. Tabela eventos de mudança entre meses
7. Docker + job semanal (checagem; pipeline quando mês novo)

Fora escopo: API, gold desnormalizado, fontes externas, scoring.

## 3 Não reabrir
- DuckDB sobre Parquet. Sem SQL Server.
- Sem Postgres/ClickHouse/Redis nesta fase.
- `raw` imutável; `silver` tipado; gold futuro merge com prioridade explícita; fontes externas = silver própria.
- Partição `ref=AAAA-MM/`, ponteiro `current` só após validação.

## 4 Fonte de dados
- Primária decidida: `https://dadosabertos.rfb.gov.br/CNPJ/`
- Fallback: `https://arquivos.receitafederal.gov.br/index.php/s/YggdBLfdninEJX9`
- Fallback 2: `dados.gov.br/dataset/cadastro-nacional-da-pessoa-juridica-cnpj`
- Layout oficial: `https://www.gov.br/receitafederal/dados/cnpj-metadados.pdf` (NOVO LAYOUT, 6 pág)
- Arquivos: `Empresas[0-9].zip`, `Estabelecimentos[0-9].zip`, `Socios[0-9].zip`, `Simples.zip`, `Cnaes.zip`, `Motivos.zip`, `Municipios.zip`, `Naturezas.zip`, `Paises.zip`, `Qualificacoes.zip`
- Formato: CSV sem header, `;`, `ISO-8859-1`, datas `AAAAMMDD` (`00000000` = NULL), decimais vírgula, CPF sócio mascarado

### 4.1 Ordem colunas (do PDF — `columns={}` explícito, sem auto_detect)
**Empresas (7):** `cnpj_basico, razao_social, natureza_juridica, qualificacao_responsavel, capital_social, porte_empresa, ente_federativo`
**Estabelecimentos (30):** `cnpj_basico, cnpj_ordem, cnpj_dv, identificador_matriz_filial, nome_fantasia, situacao_cadastral, data_situacao_cadastral, motivo_situacao_cadastral, nome_cidade_exterior, pais, data_inicio_atividade, cnae_fiscal_principal, cnae_fiscal_secundaria, tipo_logradouro, logradouro, numero, complemento, bairro, cep, uf, municipio, ddd1, telefone1, ddd2, telefone2, ddd_fax, fax, correio_eletronico, situacao_especial, data_situacao_especial`
**Sócios (11):** `cnpj_basico, identificador_socio, nome_socio, cpf_cnpj_socio_mascarado, qualificacao_socio, data_entrada_sociedade, pais, representante_legal, nome_representante, qualificacao_representante, faixa_etaria`
**Simples (6+):** `cnpj_basico, opcao_simples, data_opcao_simples, data_exclusao_simples, opcao_mei, data_opcao_mei, data_exclusao_mei`
**Domínios (codigo, descricao):** `cnaes, motivos, municipios, naturezas, paises, qualificacoes`

## 5 Arquitetura pastas
```
DATA_DIR=/data/cnpj
├── raw/ref=AAAA-MM/          # ZIPs imutáveis + _manifest.json
├── silver/ref=AAAA-MM/       # Parquet tipado + _stats.json + _eventos_mudanca.parquet
├── current -> silver/ref=AAAA-MM  # symlink atômico
├── state.json                # {ultimo_ref_processado, ultimo_check}
└── tmp/                      # por etapa, apagado ao fim
```
`current` sempre via symlink; cada silver com `fonte, ref_mes, coletado_em`.

## 6 Esquema silver (tipos Parquet/DuckDB)
- **empresas**: `cnpj_basico VARCHAR, razao_social VARCHAR, natureza_juridica USMALLINT, qualificacao_responsavel USMALLINT, capital_social DECIMAL(16,2) NULL, porte_empresa UTINYINT, ente_federativo VARCHAR NULL`
- **estabelecimentos** (particionado por `uf`, ordenado por `cnpj_basico`): `cnpj_basico/ordem/dv VARCHAR, cnpj_completo VARCHAR (basico+ordem+dv), identificador_matriz_filial UTINYINT, situacao_cadastral UTINYINT, data_situacao_cadastral DATE NULL, motivo USMALLINT, pais USMALLINT, data_inicio_atividade DATE NULL, cnae_principal UINTEGER, cnae_secundaria VARCHAR, endereco..., cep VARCHAR, uf VARCHAR, municipio UINTEGER, telefones VARCHAR, email VARCHAR, situacao_especial VARCHAR`
- Conversões: `00000000`→NULL, `capital_social` vírgula→ponto `::DECIMAL`, `cep` sem máscara.
- **socios/simples/domínios**: análogos; `cpf_cnpj_socio_mascarado` nunca é chave.

## 7 Módulos Fase 1
- **7.1 Descoberta**: lista HTTP da base, extrai `YYYY-MM`, max vs `state.json`; tenta primária→fallback→dados.gov.br
- **7.2 Downloader**: 4-6 paralelas, Range resume, backoff 1/2/4/8s jitter max 5, magic bytes + unzip -t + sha256, idempotente
- **7.3 Conversão**: `COPY (SELECT ... FROM read_csv(..., columns={...})) TO ... (FORMAT PARQUET, COMPRESSION zstd)`; `PARTITION_BY uf` para estabelecimentos; apaga CSV após Parquet OK
- **7.4 Validação**: contagens vs anterior (`empresas ±3%`, `estab/socios ±5%`), nulos% delta 10pp, unicidade CNPJ; `_stats.json`; `ln -sfn` só se OK (`FORCE_PROMOTE=1` override)
- **7.5 Diff**: `_eventos_mudanca.parquet` via ANTI/FULL JOIN entre refs; `tipo_evento ENUM`
- **7.6 Orquestração**: binário `cnpj-pipeline` (`discover/download/convert/validate/diff/promote/run`); `run` = pipeline completo; lock `pipeline.lock`

## 8 Docker
Multi-stage `golang:1.22-alpine` (`CGO_ENABLED=0`) + `alpine` com `duckdb` + `unzip`; volume `DATA_DIR`; check `<20GB` aborta cedo; Swarm `replicas 0/1` ou cron host `docker run --rm`.

## 9 Config (env)
`DATA_DIR`, `CNPJ_BASE_URL`, `CNPJ_FALLBACK_URL`, `PARALLELISM=4`, `RETRY_MAX=5`, `FORCE_PROMOTE=0`, `LOG_LEVEL`, `TZ=UTC`.

## 10 Logs e métricas
JSON por etapa (`etapa, ref_mes, duracao_ms, linhas, bytes`); `_stats.json`.

## 11 Idempotência
Artefato existe e válido→skip; writes `tmp`+`rename`; falha não atualiza `current`/`state.json`; reexec retoma do falhado; `.part` preservado para resume.

## 12 Estimativa
ZIPs 4-6GB, CSVs 25-35GB, Parquet 8-12GB, tmp pico ~40GB; tempo 4vCPU/8GB: download 1-3h, conversão 0.5-2h. Mín. 60GB livres.

## 13 Riscos
RFB instável, URL migra, layout muda, `00000000`/vírgula quebra CAST, threshold mal calibrado, disco esgota, reexec corrompe — ver SDD original para mitigações completas.

## 14 Fora deste SDD (futuro)
Gold, fontes externas (silver própria), API, scoring, Postgres/ClickHouse/Redis com gargalo provado.
