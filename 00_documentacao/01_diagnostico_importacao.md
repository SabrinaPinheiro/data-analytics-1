# Diagnostico inicial de importacao

Data da analise: 2026-06-28

## Objetivo

Preparar a importacao do dump `01_origem/corporativo.sql` no projeto Supabase conectado, sem executar o restore.

## Projeto Supabase identificado

- Nome: `data-analytics-1`
- Project ID/ref: `bkymasiwmcnwpxptpuow`
- Regiao: `sa-east-1`
- Status: `ACTIVE_HEALTHY`
- Host do banco: `db.bkymasiwmcnwpxptpuow.supabase.co`
- Engine informada pelo Supabase: PostgreSQL `17`

## Versao do PostgreSQL

Consulta executada:

```sql
select version();
```

Resultado:

```text
PostgreSQL 17.6 on aarch64-unknown-linux-gnu, compiled by gcc (GCC) 15.2.0, 64-bit
```

Conclusao: a versao do banco Supabase e compativel com o dump, que foi criado a partir de PostgreSQL 17.3 usando `pg_dump` 17.3.

## Formato do arquivo de origem

O arquivo `01_origem/corporativo.sql` possui assinatura `PGDMP` e foi validado com `pg_restore -l`.

Metadados relevantes:

```text
Format: CUSTOM
Compression: gzip
Database original: corporativo_v1
Dumped from database version: 17.3
Dumped by pg_dump version: 17.3
TOC Entries: 177
```

Conclusao: apesar da extensao `.sql`, o arquivo e um dump PostgreSQL em formato custom. Ele deve ser restaurado com `pg_restore`, nao com `psql` nem via SQL Editor do Supabase.

## Verificacao de schemas e objetos existentes

Schemas avaliados:

- `geral`
- `vendas`
- `stage`
- `data_warehouse`
- `public`

Resultado da verificacao:

| Schema | Existe? | Objetos encontrados |
| --- | --- | --- |
| `data_warehouse` | Nao | 0 |
| `geral` | Nao | 0 |
| `stage` | Nao | 0 |
| `vendas` | Nao | 0 |
| `public` | Sim | 0 |

Tambem foi verificada a colisao direta com os objetos esperados do dump, incluindo tabelas, sequencias e views. Resultado: nenhuma colisao encontrada.

## Avaliacao de seguranca para restore

O banco esta seguro para receber o restore do dump `corporativo.sql`, considerando os criterios abaixo:

- O Supabase esta em PostgreSQL 17.6, compativel com o dump gerado em PostgreSQL 17.3.
- Os schemas `geral`, `vendas`, `stage` e `data_warehouse` ainda nao existem.
- O schema `public` existe, como esperado em Supabase, mas nao possui objetos detectados na verificacao realizada.
- Nao ha colisao com `public.view_dt_mart_vendas`, que sera criada pelo dump.
- O restore ainda deve ser executado sem `--clean`, para evitar qualquer acao destrutiva desnecessaria.
- O restore deve usar `--no-owner` e `--no-privileges` para evitar conflitos com roles/owners do ambiente original.

Observacao de seguranca Supabase: apos o restore, sera necessario revisar exposicao de schemas e RLS, especialmente porque o dump cria uma view em `public`, schema que pode ser exposto pela Data API.

## Comando preparado para restore

Nao executar sem confirmacao previa.

PowerShell:

```powershell
& 'C:\Program Files\PostgreSQL\17\bin\pg_restore.exe' `
  --dbname "<SUPABASE_DATABASE_URL>" `
  --no-owner `
  --no-privileges `
  --single-transaction `
  --verbose `
  "C:\Apps\data-analytics-1\01_origem\corporativo.sql"
```

Substituir `<SUPABASE_DATABASE_URL>` pela connection string do banco Supabase, preferencialmente no formato:

```text
postgresql://postgres.<project-ref>:<password>@aws-0-sa-east-1.pooler.supabase.com:6543/postgres
```

ou pela connection string direta equivalente fornecida pelo painel do Supabase.

## Proxima etapa recomendada

Executar o restore apenas apos confirmacao explicita, monitorar o log do `pg_restore` e, em seguida, validar:

- schemas criados;
- quantidade de tabelas, views e sequencias;
- contagem de linhas por tabela;
- constraints e foreign keys;
- funcionamento da view `public.view_dt_mart_vendas`;
- configuracao de acesso/RLS para uso com Power BI e Data API.
