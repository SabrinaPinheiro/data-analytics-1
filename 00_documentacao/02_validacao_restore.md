# Validacao do restore da base corporativa

Data da validacao: 2026-06-28

## Resumo

O restore do dump `01_origem/corporativo.sql` foi executado no projeto Supabase `data-analytics-1` (`bkymasiwmcnwpxptpuow`) com sucesso.

O comando foi executado com as opcoes solicitadas:

- `--no-owner`
- `--no-privileges`
- `--single-transaction`
- `--verbose`

Nao foi utilizado `--clean`.

O `pg_restore` finalizou com codigo `0`, criando schemas, tabelas, views, sequencias, dados, constraints e foreign keys.

## Ambiente de destino

Validacao de conexao antes do restore:

```text
Database: postgres
User: postgres
PostgreSQL: 17.6
```

## Schemas criados

Schemas confirmados apos o restore:

| Schema |
| --- |
| `data_warehouse` |
| `geral` |
| `public` |
| `stage` |
| `vendas` |

Observacao: `public` ja existia no Supabase e recebeu a view `public.view_dt_mart_vendas`.

## Quantidade de objetos por schema

| Schema | Tipo de objeto | Quantidade |
| --- | --- | ---: |
| `data_warehouse` | sequences | 1 |
| `data_warehouse` | tables | 7 |
| `data_warehouse` | views | 1 |
| `geral` | sequences | 9 |
| `geral` | tables | 10 |
| `geral` | views | 2 |
| `public` | views | 1 |
| `stage` | tables | 4 |
| `vendas` | sequences | 5 |
| `vendas` | tables | 5 |
| `vendas` | views | 2 |

Totais:

- Tabelas: 26
- Views: 6
- Sequencias: 15

## Contagem de linhas por tabela

| Schema | Tabela | Linhas |
| --- | --- | ---: |
| `data_warehouse` | `dim_cliente` | 1.000 |
| `data_warehouse` | `dim_forma_pagamento` | 9 |
| `data_warehouse` | `dim_produto` | 233 |
| `data_warehouse` | `dim_tempo` | 12.785 |
| `data_warehouse` | `dim_vendedor` | 24 |
| `data_warehouse` | `fato_vendas` | 25.391 |
| `data_warehouse` | `fato_vendas_detalhes` | 35.752 |
| `geral` | `bairro` | 42.158 |
| `geral` | `cidade` | 5.568 |
| `geral` | `contato` | 28.304 |
| `geral` | `endereco` | 15.962 |
| `geral` | `estado` | 27 |
| `geral` | `pessoa` | 15.962 |
| `geral` | `pessoa_fisica` | 14.155 |
| `geral` | `pessoa_juridica` | 1.807 |
| `geral` | `responsavel_juridico` | 928 |
| `geral` | `tipo_contato` | 5 |
| `stage` | `forma_pagamento` | 9 |
| `stage` | `nota_fiscal` | 25.391 |
| `stage` | `pessoa_fisica` | 14.155 |
| `stage` | `pessoa_juridica` | 1.807 |
| `vendas` | `categoria` | 9 |
| `vendas` | `forma_pagamento` | 9 |
| `vendas` | `item_nota_fiscal` | 35.752 |
| `vendas` | `nota_fiscal` | 25.391 |
| `vendas` | `produto` | 233 |

## Constraints e foreign keys

Resumo por schema:

| Schema | Tipo | Total | Validadas | Nao validadas |
| --- | --- | ---: | ---: | ---: |
| `data_warehouse` | foreign_key | 8 | 8 | 0 |
| `data_warehouse` | primary_key | 7 | 7 | 0 |
| `geral` | foreign_key | 10 | 9 | 1 |
| `geral` | primary_key | 10 | 10 | 0 |
| `geral` | unique | 2 | 2 | 0 |
| `stage` | primary_key | 4 | 4 | 0 |
| `vendas` | foreign_key | 7 | 0 | 7 |
| `vendas` | primary_key | 5 | 5 | 0 |

Foreign keys importadas como `NOT VALID`, conforme definicao original do dump:

| Schema | Tabela | Foreign key |
| --- | --- | --- |
| `geral` | `endereco` | `endereco_id_pessoa_fkey` |
| `vendas` | `item_nota_fiscal` | `item_nota_fiscal_id_nota_fiscal_fkey` |
| `vendas` | `item_nota_fiscal` | `item_nota_fiscal_id_produto_fkey` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_cliente_fkey` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_forma_pagto_fkey` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_vendedor_fkey` |
| `vendas` | `produto` | `produto_id_categoria_fkey` |
| `vendas` | `produto` | `produto_id_fornecedor_fkey` |

Observacao: essas FKs existem no banco, mas foram restauradas com `NOT VALID`, exatamente como estavam no dump. A validacao dessas constraints pode ser uma etapa posterior, caso o projeto exija integridade referencial formalmente validada.

## Validacao das views

Views encontradas:

| Schema | View |
| --- | --- |
| `data_warehouse` | `view_dt_mart_vendas` |
| `geral` | `vi_lista_pessoa_fisica` |
| `geral` | `vi_lista_pessoas` |
| `public` | `view_dt_mart_vendas` |
| `vendas` | `vi_lista_cliente` |
| `vendas` | `vi_relatorio_gestao` |

Teste obrigatorio da view publica:

```sql
select count(*) as view_rows
from public.view_dt_mart_vendas;
```

Resultado:

```text
view_rows = 35752
```

Tambem foi executado:

```sql
select *
from public.view_dt_mart_vendas
limit 5;
```

Resultado: a query retornou linhas com dados de venda, produto, cliente, vendedor, tempo, valores e localidade. Portanto, a view `public.view_dt_mart_vendas` esta funcional.

## Conclusao

O restore da base corporativa foi concluido e validado com sucesso no Supabase.

Pontos de atencao para as proximas etapas:

- revisar exposicao do schema `public`, pois a view `public.view_dt_mart_vendas` pode ser acessivel pela Data API dependendo das configuracoes do projeto;
- avaliar se as foreign keys `NOT VALID` devem ser validadas posteriormente com `ALTER TABLE ... VALIDATE CONSTRAINT`;
- documentar o modelo relacional e o modelo dimensional a partir dos schemas restaurados;
- preparar consultas de Data Mart e conexao com Power BI.
