# Validação das Métricas e KPIs Analíticos

Data da validacao: 2026-06-28

## Escopo e premissas

Esta etapa valida se as metricas basicas e os KPIs definidos em `06_objetivos_analiticos.md` podem ser obtidos a partir do modelo dimensional existente no schema `data_warehouse`.

Foram executadas apenas consultas `SELECT`. Nenhuma estrutura do banco foi alterada, nenhum dado foi modificado e nenhum objeto foi criado.

Ambiente validado:

```text
Database: postgres
Usuario: postgres
PostgreSQL: 17.6
Projeto Supabase: data-analytics-1
```

Observacao operacional: uma primeira tentativa de comparar clientes entre as duas fatos usou `cross join` e foi abortada pelo PostgreSQL por espaco temporario insuficiente. A consulta foi substituida por subconsultas escalares somente leitura. Nao houve alteracao no banco.

## 1. Faturamento total

### Objetivo de negocio

Medir a receita bruta total de vendas no periodo analisado.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`, no grao de nota fiscal.
- `data_warehouse.fato_vendas_detalhes`, no grao de item, apenas para reconciliacao.

### Dimensoes utilizadas

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_produto`, quando a analise estiver no detalhe

### Medidas utilizadas

- `fato_vendas.valor`
- `fato_vendas_detalhes.valor_venda_real`

### SQL de validacao

```sql
select
  count(*) as registros_fato_vendas,
  count(distinct numero_nf) as notas_fiscais,
  sum(valor)::numeric(18,2) as faturamento_fato_vendas
from data_warehouse.fato_vendas;

select
  count(*) as registros_fato_detalhes,
  count(distinct numero_nf) as notas_fiscais,
  sum(valor_venda_real)::numeric(18,2) as faturamento_fato_detalhes
from data_warehouse.fato_vendas_detalhes;

select
  (select sum(valor) from data_warehouse.fato_vendas)::numeric(18,2) as faturamento_cabecalho,
  (select sum(valor_venda_real) from data_warehouse.fato_vendas_detalhes)::numeric(18,2) as faturamento_detalhe,
  (
    (select sum(valor) from data_warehouse.fato_vendas)
    - (select sum(valor_venda_real) from data_warehouse.fato_vendas_detalhes)
  )::numeric(18,2) as diferenca;
```

### Resultado obtido

| Origem | Registros | Notas fiscais | Valor |
| --- | ---: | ---: | ---: |
| `fato_vendas` | 25.391 | 25.391 | 88.562.548,00 |
| `fato_vendas_detalhes` | 35.752 | 25.391 | 88.562.548,00 |
| Diferenca entre fatos | - | - | 0,00 |

### Validacao

KPI validado. O faturamento de cabecalho e o faturamento de detalhe conciliam exatamente.

### Cuidados

Nao somar `fato_vendas.valor` depois de juntar com itens, pois o valor da nota sera repetido por produto.

## 2. Lucro bruto estimado

### Objetivo de negocio

Estimar a rentabilidade bruta das vendas a partir do valor real vendido e do custo dos produtos.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `valor_custo`
- `quantidade`

### SQL de validacao

```sql
select
  count(*) as registros_item,
  sum(valor_venda_real)::numeric(18,2) as faturamento,
  sum(valor_custo * quantidade)::numeric(18,2) as custo_estimado,
  sum(valor_venda_real - valor_custo * quantidade)::numeric(18,2) as lucro_bruto_estimado
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Registros | Faturamento | Custo estimado | Lucro bruto estimado |
| ---: | ---: | ---: | ---: |
| 35.752 | 88.562.548,00 | 65.491.931,00 | 23.070.617,00 |

### Validacao

Validado com ressalvas. O KPI pode ser calculado tecnicamente, mas depende da confirmacao de que `valor_custo` representa custo unitario.

### Cuidados

Se `valor_custo` ja representar custo total da linha, multiplicar por `quantidade` duplicaria o custo. A semantica deve ser confirmada antes de usar como indicador financeiro oficial.

## 3. Margem bruta estimada

### Objetivo de negocio

Medir o percentual do faturamento que permanece apos o custo estimado dos produtos.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `valor_custo`
- `quantidade`

### SQL de validacao

```sql
select
  sum(valor_venda_real)::numeric(18,2) as faturamento,
  sum(valor_venda_real - valor_custo * quantidade)::numeric(18,2) as lucro_bruto_estimado,
  round(
    (sum(valor_venda_real - valor_custo * quantidade)
     / nullif(sum(valor_venda_real), 0) * 100)::numeric,
    2
  ) as margem_percentual
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Faturamento | Lucro bruto estimado | Margem percentual |
| ---: | ---: | ---: |
| 88.562.548,00 | 23.070.617,00 | 26,05% |

### Validacao

Validado com ressalvas pela mesma dependencia semantica do custo.

### Cuidados

Nao calcular margem usando `valor_nota_fiscal` na fato detalhe, pois esse campo esta repetido por item.

## 4. Ticket medio

### Objetivo de negocio

Medir o valor medio de cada venda/nota fiscal.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`

### Medidas utilizadas

- `valor`
- `numero_nf`

### SQL de validacao

```sql
select
  count(*) as registros,
  count(distinct numero_nf) as notas_fiscais,
  sum(valor)::numeric(18,2) as faturamento,
  round((sum(valor) / nullif(count(distinct numero_nf), 0))::numeric, 2) as ticket_medio
from data_warehouse.fato_vendas;
```

### Resultado obtido

| Registros | Notas fiscais | Faturamento | Ticket medio |
| ---: | ---: | ---: | ---: |
| 25.391 | 25.391 | 88.562.548,00 | 3.487,95 |

### Validacao

KPI validado.

### Cuidados

Usar sempre o grao de nota fiscal. No detalhe, a nota pode aparecer mais de uma vez.

## 5. Quantidade vendida

### Objetivo de negocio

Medir o volume fisico vendido.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `quantidade`

### SQL de validacao

```sql
select
  count(*) as registros_item,
  count(distinct numero_nf) as notas_fiscais,
  sum(quantidade) as quantidade_vendida
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Registros de item | Notas fiscais | Quantidade vendida |
| ---: | ---: | ---: |
| 35.752 | 25.391 | 89.527 |

### Validacao

KPI validado.

### Cuidados

Nao usar `count(*)` como quantidade vendida; `count(*)` mede linhas de item, nao unidades.

## 6. Numero de clientes ativos

### Objetivo de negocio

Medir quantos clientes realizaram compras no periodo.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`
- `data_warehouse.fato_vendas_detalhes`, para reconciliacao

### Dimensoes utilizadas

- `dim_cliente`
- `dim_tempo`

### Medidas utilizadas

- `id_cliente`

### SQL de validacao

```sql
select
  (select count(distinct id_cliente) from data_warehouse.fato_vendas) as clientes_ativos_fato_vendas,
  (select count(distinct id_cliente) from data_warehouse.fato_vendas_detalhes) as clientes_ativos_fato_detalhes,
  (select count(*) from data_warehouse.dim_cliente) as clientes_dimensao;
```

### Resultado obtido

| Clientes ativos em `fato_vendas` | Clientes ativos em `fato_vendas_detalhes` | Clientes na dimensao |
| ---: | ---: | ---: |
| 1.000 | 1.000 | 1.000 |

### Validacao

KPI validado.

### Cuidados

Usar `count(distinct id_cliente)`. Contagem simples de linhas mede vendas ou itens, nao clientes.

## 7. Numero de vendas

### Objetivo de negocio

Medir o volume de transacoes comerciais.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`

### Medidas utilizadas

- `numero_nf`

### SQL de validacao

```sql
select
  count(*) as registros,
  count(distinct numero_nf) as numero_de_vendas
from data_warehouse.fato_vendas;
```

### Resultado obtido

| Registros | Numero de vendas |
| ---: | ---: |
| 25.391 | 25.391 |

### Validacao

KPI validado.

### Cuidados

Usar `count(distinct numero_nf)` quando houver qualquer juncao com tabelas de detalhe.

## 8. Itens por venda

### Objetivo de negocio

Avaliar a composicao media das compras, medindo quantas linhas de item existem por nota fiscal.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_tempo`
- `dim_produto`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- linhas de item
- `numero_nf`

### SQL de validacao

```sql
select
  count(*) as itens,
  count(distinct numero_nf) as vendas,
  round((count(*)::numeric / nullif(count(distinct numero_nf), 0)), 2) as itens_por_venda
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Itens | Vendas | Itens por venda |
| ---: | ---: | ---: |
| 35.752 | 25.391 | 1,41 |

### Validacao

KPI validado.

### Cuidados

Este KPI mede linhas de item por venda, nao unidades por venda. Para unidades por venda, usar `sum(quantidade) / count(distinct numero_nf)`.

## 9. Faturamento por vendedor

### Objetivo de negocio

Comparar desempenho comercial individual.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_vendedor`
- `dim_tempo`
- `dim_cliente`
- `dim_forma_pagamento`

### Medidas utilizadas

- `valor`
- `numero_nf`

### SQL de validacao

```sql
select
  v.nome as vendedor,
  count(distinct f.numero_nf) as vendas,
  sum(f.valor)::numeric(18,2) as faturamento
from data_warehouse.fato_vendas f
join data_warehouse.dim_vendedor v
  on v.id = f.id_vendedor
group by v.nome
order by faturamento desc
limit 10;

select
  count(*) as vendedores_com_venda,
  sum(faturamento)::numeric(18,2) as faturamento_total
from (
  select f.id_vendedor, sum(f.valor) as faturamento
  from data_warehouse.fato_vendas f
  group by f.id_vendedor
) x;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Vendedores com venda | 24 |
| Faturamento total conciliado | 88.562.548,00 |
| Maior faturamento | Francisco Carlos da Silva: 7.282.299,00 |
| Segundo maior faturamento | Maria do Socorro Alves: 4.129.952,00 |
| Terceiro maior faturamento | Antonio Alves da Silva: 3.943.474,00 |

### Validacao

KPI validado.

### Cuidados

Usar a fato de cabecalho para evitar duplicar valor da nota em vendas com mais de um item.

## 10. Ticket medio por vendedor

### Objetivo de negocio

Avaliar o valor medio das vendas de cada vendedor.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_vendedor`
- `dim_tempo`

### Medidas utilizadas

- `valor`
- `numero_nf`

### SQL de validacao

```sql
select
  v.nome as vendedor,
  count(distinct f.numero_nf) as vendas,
  sum(f.valor)::numeric(18,2) as faturamento,
  round((sum(f.valor) / nullif(count(distinct f.numero_nf), 0))::numeric, 2) as ticket_medio
from data_warehouse.fato_vendas f
join data_warehouse.dim_vendedor v
  on v.id = f.id_vendedor
group by v.nome
order by ticket_medio desc
limit 10;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Vendedores avaliados | 24 |
| Maior ticket medio | Maria do Socorro Alves: 3.771,65 |
| Segundo maior ticket medio | Ana Paula da Silva Lima: 3.733,35 |
| Terceiro maior ticket medio | Antonia Moreira da Silva: 3.660,31 |

### Validacao

KPI validado.

### Cuidados

Quando cruzar com produto/categoria, calcular ticket medio a partir de notas distintas, nao de linhas de item.

## 11. Clientes por vendedor

### Objetivo de negocio

Avaliar a amplitude da carteira atendida por cada vendedor.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_vendedor`
- `dim_cliente`
- `dim_tempo`

### Medidas utilizadas

- `id_cliente`

### SQL de validacao

```sql
select
  v.nome as vendedor,
  count(distinct f.id_cliente) as clientes_atendidos
from data_warehouse.fato_vendas f
join data_warehouse.dim_vendedor v
  on v.id = f.id_vendedor
group by v.nome
order by clientes_atendidos desc, vendedor
limit 10;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Vendedores avaliados | 24 |
| Maior carteira | Francisco Carlos da Silva: 885 clientes |
| Segunda maior carteira | Antonio Alves da Silva: 672 clientes |
| Terceira maior carteira | Maria do Socorro Alves: 671 clientes |

### Validacao

KPI validado.

### Cuidados

Usar `distinct`, pois um mesmo cliente pode comprar varias vezes com o mesmo vendedor.

## 12. Faturamento por categoria

### Objetivo de negocio

Identificar as categorias que mais contribuem para a receita.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `quantidade`

### SQL de validacao

```sql
select
  p.categoria,
  count(*) as itens,
  sum(d.quantidade) as quantidade,
  sum(d.valor_venda_real)::numeric(18,2) as faturamento
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_produto p
  on p.id = d.id_produto
group by p.categoria
order by faturamento desc;
```

### Resultado obtido

| Categoria | Itens | Quantidade | Faturamento |
| --- | ---: | ---: | ---: |
| INFORMATICA | 11.512 | 28.779 | 29.340.956,00 |
| TV E AUDIO | 3.684 | 9.093 | 22.396.854,00 |
| ELETRODOMESTICOS | 2.916 | 7.303 | 10.440.766,00 |
| PASSAGENS | 3.274 | 8.186 | 9.841.200,00 |
| CELULARES | 4.205 | 10.664 | 8.296.121,00 |
| MOVEIS | 1.303 | 3.299 | 4.920.100,00 |
| GAMES | 3.440 | 8.599 | 2.139.767,00 |
| DVDS | 3.091 | 7.774 | 725.667,00 |
| LIVROS | 2.327 | 5.830 | 461.117,00 |

### Validacao

KPI validado.

### Cuidados

Categoria existe somente via `dim_produto`, portanto a fato correta e a de detalhe.

## 13. Participacao da categoria

### Objetivo de negocio

Medir o peso percentual de cada categoria no faturamento total.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`

### Medidas utilizadas

- `valor_venda_real`

### SQL de validacao

```sql
select
  p.categoria,
  sum(d.valor_venda_real)::numeric(18,2) as faturamento,
  round(
    (sum(d.valor_venda_real) / sum(sum(d.valor_venda_real)) over () * 100)::numeric,
    2
  ) as participacao_percentual
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_produto p
  on p.id = d.id_produto
group by p.categoria
order by faturamento desc;
```

### Resultado obtido

| Categoria | Faturamento | Participacao |
| --- | ---: | ---: |
| INFORMATICA | 29.340.956,00 | 33,13% |
| TV E AUDIO | 22.396.854,00 | 25,29% |
| ELETRODOMESTICOS | 10.440.766,00 | 11,79% |
| PASSAGENS | 9.841.200,00 | 11,11% |
| CELULARES | 8.296.121,00 | 9,37% |
| MOVEIS | 4.920.100,00 | 5,56% |
| GAMES | 2.139.767,00 | 2,42% |
| DVDS | 725.667,00 | 0,82% |
| LIVROS | 461.117,00 | 0,52% |

### Validacao

KPI validado.

### Cuidados

O denominador deve respeitar os filtros do dashboard. Em Power BI, a medida deve controlar o contexto de filtro.

## 14. Faturamento por produto

### Objetivo de negocio

Identificar os produtos que mais geram receita.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `quantidade`

### SQL de validacao

```sql
select
  p.produto,
  p.categoria,
  sum(d.quantidade) as quantidade,
  sum(d.valor_venda_real)::numeric(18,2) as faturamento
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_produto p
  on p.id = d.id_produto
group by p.produto, p.categoria
order by faturamento desc
limit 10;

select count(distinct id_produto) as produtos_com_venda
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Produtos com venda | 226 |
| Produto lider | NOTEBOOK MSFT HIBR: 1.948.800,00 |
| Segundo produto | SMART TV PANASONIC 52: 1.843.800,00 |
| Terceiro produto | LAVADORA GANCA 10L: 1.836.000,00 |

### Validacao

KPI validado.

### Cuidados

Usar `id_produto` como chave. O inventario mostra 233 chaves de produto e 227 nomes distintos, entao nomes podem se repetir.

## 15. Preco medio praticado

### Objetivo de negocio

Medir o valor medio efetivo por unidade vendida.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `quantidade`
- `valor_unitario`, para analises alternativas

### SQL de validacao

```sql
select
  p.categoria,
  sum(d.quantidade) as quantidade,
  sum(d.valor_venda_real)::numeric(18,2) as faturamento,
  round((sum(d.valor_venda_real) / nullif(sum(d.quantidade), 0))::numeric, 2) as preco_medio_praticado
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_produto p
  on p.id = d.id_produto
group by p.categoria
order by preco_medio_praticado desc;

select
  sum(quantidade) as quantidade_total,
  sum(valor_venda_real)::numeric(18,2) as faturamento_total,
  round((sum(valor_venda_real) / nullif(sum(quantidade), 0))::numeric, 2) as preco_medio_geral
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Quantidade total | 89.527 |
| Faturamento total | 88.562.548,00 |
| Preco medio geral | 989,23 |
| Maior preco medio por categoria | TV E AUDIO: 2.463,09 |
| Menor preco medio por categoria | LIVROS: 79,09 |

### Validacao

KPI validado.

### Cuidados

Preferir `sum(valor_venda_real) / sum(quantidade)` a `avg(valor_unitario)` para preservar ponderacao por volume.

## 16. Desconto ou variacao media estimada

### Objetivo de negocio

Avaliar possivel diferenca entre valor referencial e valor real praticado.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda`
- `valor_venda_real`

### SQL de validacao

```sql
select
  count(*) as registros_item,
  sum(valor_venda)::numeric(18,2) as valor_referencial,
  sum(valor_venda_real)::numeric(18,2) as valor_real,
  sum(valor_venda - valor_venda_real)::numeric(18,2) as variacao_absoluta,
  round(
    (sum(valor_venda - valor_venda_real) / nullif(sum(valor_venda), 0) * 100)::numeric,
    2
  ) as variacao_percentual_sobre_referencial
from data_warehouse.fato_vendas_detalhes;
```

### Resultado obtido

| Registros | Valor referencial | Valor real | Variacao absoluta | Variacao percentual |
| ---: | ---: | ---: | ---: | ---: |
| 35.752 | 35.520.358,00 | 88.562.548,00 | -53.042.190,00 | -149,33% |

### Validacao

Necessita ajuste. O KPI pode ser calculado matematicamente, mas o resultado indica que `valor_venda` nao se comporta como base simples de desconto em relacao a `valor_venda_real`.

### Cuidados

Nao recomendar como KPI oficial de desconto antes de validar a semantica dos campos `valor_venda`, `valor_unitario` e `valor_venda_real`. O indicador pode gerar interpretacao incorreta.

## 17. Faturamento por periodo

### Objetivo de negocio

Acompanhar evolucao temporal do faturamento.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_tempo`

### Medidas utilizadas

- `valor`
- `numero_nf`

### SQL de validacao

```sql
select
  t.ano,
  count(distinct f.numero_nf) as vendas,
  sum(f.valor)::numeric(18,2) as faturamento
from data_warehouse.fato_vendas f
join data_warehouse.dim_tempo t
  on t.id = f.id_data_venda
group by t.ano
order by t.ano;

select
  count(distinct t.data) as dias_com_venda,
  min(t.data) as primeira_venda,
  max(t.data) as ultima_venda
from data_warehouse.fato_vendas f
join data_warehouse.dim_tempo t
  on t.id = f.id_data_venda;
```

### Resultado obtido

| Ano | Vendas | Faturamento |
| ---: | ---: | ---: |
| 2015 | 7.999 | 28.167.840,00 |
| 2016 | 8.392 | 29.348.852,00 |
| 2017 | 9.000 | 31.045.856,00 |

Periodo com vendas:

| Dias com venda | Primeira venda | Ultima venda |
| ---: | --- | --- |
| 972 | 2015-01-01 | 2017-12-27 |

### Validacao

KPI validado.

### Cuidados

Semestre e trimestre nao estao materializados em `dim_tempo`, mas podem ser derivados no Power BI ou em Data Mart futuro.

## 18. Faturamento por estado

### Objetivo de negocio

Mapear concentracao geografica das vendas.

### Tabela fato utilizada

- `data_warehouse.fato_vendas_detalhes`

### Dimensoes utilizadas

- `dim_cliente`
- `dim_tempo`
- `dim_produto`
- `dim_vendedor`

### Medidas utilizadas

- `valor_venda_real`
- `numero_nf`

### SQL de validacao

```sql
select
  c.estado,
  count(distinct d.numero_nf) as vendas,
  sum(d.valor_venda_real)::numeric(18,2) as faturamento
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_cliente c
  on c.id = d.id_cliente
group by c.estado
order by faturamento desc
limit 10;

select count(distinct c.estado) as estados_com_venda
from data_warehouse.fato_vendas_detalhes d
join data_warehouse.dim_cliente c
  on c.id = d.id_cliente;
```

### Resultado obtido

| Indicador | Valor |
| --- | ---: |
| Estados com venda | 17 |
| Estado lider | CEARA: 82.636.744,00 |
| Segundo estado | SAO PAULO: 1.652.066,00 |
| Terceiro estado | MINAS GERAIS: 886.760,00 |

### Validacao

KPI validado.

### Cuidados

Ha forte concentracao no Ceara. Em dashboards, usar percentuais e ranking para evitar leitura distorcida por escalas muito diferentes.

## 19. Faturamento por forma de pagamento

### Objetivo de negocio

Entender a distribuicao financeira das vendas por modalidade de pagamento.

### Tabela fato utilizada

- `data_warehouse.fato_vendas`

### Dimensoes utilizadas

- `dim_forma_pagamento`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

### Medidas utilizadas

- `valor`
- `numero_nf`

### SQL de validacao

```sql
select
  coalesce(fp.descricao, 'SEM FORMA') as forma_pagamento,
  count(distinct f.numero_nf) as vendas,
  sum(f.valor)::numeric(18,2) as faturamento,
  round((sum(f.valor) / sum(sum(f.valor)) over () * 100)::numeric, 2) as participacao_percentual
from data_warehouse.fato_vendas f
join data_warehouse.dim_forma_pagamento fp
  on fp.id = f.id_forma_pagamento
group by fp.descricao
order by faturamento desc;

select
  count(distinct id_forma_pagamento) as formas_usadas,
  (select count(*) from data_warehouse.dim_forma_pagamento) as formas_cadastradas
from data_warehouse.fato_vendas;
```

### Resultado obtido

| Forma de pagamento | Vendas | Faturamento | Participacao |
| --- | ---: | ---: | ---: |
| Cartao de credito | 18.506 | 64.531.553,00 | 72,87% |
| Cartao de debito | 2.954 | 10.527.702,00 | 11,89% |
| Dinheiro | 1.952 | 6.953.308,00 | 7,85% |
| PIX | 1.016 | 3.317.826,00 | 3,75% |
| Boleto | 963 | 3.232.159,00 | 3,65% |

| Formas usadas | Formas cadastradas |
| ---: | ---: |
| 5 | 9 |

### Validacao

KPI validado.

### Cuidados

Forma de pagamento esta diretamente conectada apenas a `fato_vendas`. Para cruzar forma de pagamento com produto, e necessario relacionar as fatos por `numero_nf` e evitar somar o valor de cabecalho apos a expansao para itens.

## Validacoes complementares de qualidade analitica

### Repeticao de valor de nota na fato detalhe

```sql
select
  count(*) as linhas_detalhe,
  count(distinct numero_nf) as notas_distintas,
  sum(valor_nota_fiscal)::numeric(18,2) as soma_valor_nota_repetido,
  sum(valor_venda_real)::numeric(18,2) as soma_valor_item
from data_warehouse.fato_vendas_detalhes;
```

Resultado:

| Linhas detalhe | Notas distintas | Soma `valor_nota_fiscal` | Soma `valor_venda_real` |
| ---: | ---: | ---: | ---: |
| 35.752 | 25.391 | 173.555.131,00 | 88.562.548,00 |

Conclusao: `valor_nota_fiscal` nao deve ser somado no nivel de item. Ele repete valores de cabecalho.

### Validacao da view publica

```sql
select
  count(*) as registros_view,
  count(distinct numero_nf) as notas_view,
  sum(valor_venda_real)::numeric(18,2) as faturamento_view
from public.view_dt_mart_vendas;
```

Resultado:

| Registros view | Notas view | Faturamento view |
| ---: | ---: | ---: |
| 35.752 | 25.391 | 88.562.548,00 |

Conclusao: a view `public.view_dt_mart_vendas` esta coerente com a fato de detalhes para analises no nivel de item.

## Tabela resumo

| KPI | Pode ser calculado | Fato utilizada | Status |
| --- | --- | --- | --- |
| Faturamento total | Sim | `fato_vendas` / `fato_vendas_detalhes` | Validado |
| Lucro bruto estimado | Sim | `fato_vendas_detalhes` | Validado com ressalvas |
| Margem bruta estimada | Sim | `fato_vendas_detalhes` | Validado com ressalvas |
| Ticket medio | Sim | `fato_vendas` | Validado |
| Quantidade vendida | Sim | `fato_vendas_detalhes` | Validado |
| Numero de clientes ativos | Sim | `fato_vendas` / `fato_vendas_detalhes` | Validado |
| Numero de vendas | Sim | `fato_vendas` | Validado |
| Itens por venda | Sim | `fato_vendas_detalhes` | Validado |
| Faturamento por vendedor | Sim | `fato_vendas` | Validado |
| Ticket medio por vendedor | Sim | `fato_vendas` | Validado |
| Clientes por vendedor | Sim | `fato_vendas` | Validado |
| Faturamento por categoria | Sim | `fato_vendas_detalhes` | Validado |
| Participacao da categoria | Sim | `fato_vendas_detalhes` | Validado |
| Faturamento por produto | Sim | `fato_vendas_detalhes` | Validado |
| Preco medio praticado | Sim | `fato_vendas_detalhes` | Validado |
| Desconto ou variacao media estimada | Parcialmente | `fato_vendas_detalhes` | Necessita ajuste |
| Faturamento por periodo | Sim | `fato_vendas` | Validado |
| Faturamento por estado | Sim | `fato_vendas_detalhes` | Validado |
| Faturamento por forma de pagamento | Sim | `fato_vendas` | Validado |

## Conclusao tecnica

O Data Warehouse atende aos objetivos analiticos principais do projeto. A base suporta indicadores de faturamento, volume, ticket medio, clientes ativos, vendas, vendedores, produtos, categorias, periodo, estado e forma de pagamento.

Existem KPIs que precisam de ressalvas:

- lucro bruto estimado;
- margem bruta estimada;
- desconto ou variacao media estimada.

Lucro e margem dependem da confirmacao de que `valor_custo` e custo unitario. O indicador de desconto/variacao nao deve ser oficializado no formato atual, pois `valor_venda` e `valor_venda_real` apresentam comportamento incompatavel com uma leitura simples de desconto.

As metricas que podem gerar interpretacoes incorretas sao:

- `sum(valor_nota_fiscal)` em `fato_vendas_detalhes`, pois duplica valores de nota;
- `sum(fato_vendas.valor)` apos join com itens, pois repete o cabecalho;
- contagem simples de clientes ou notas sem `distinct` em analises detalhadas;
- desconto estimado sem validacao semantica dos campos de valor.

As limitacoes do modelo dimensional sao:

- forma de pagamento nao esta na fato de detalhes;
- produto nao esta na fato de cabecalho;
- semestre e trimestre nao estao materializados na dimensao tempo;
- custo e valor referencial precisam de dicionario de dados para virar indicadores financeiros oficiais.

O modelo esta pronto para a construcao de Data Marts de vendas, produtos, clientes, vendedores, tempo e formas de pagamento, desde que as regras de granularidade sejam respeitadas.

O modelo tambem esta pronto para Power BI. A recomendacao e criar medidas DAX separadas por grao:

- medidas de cabecalho baseadas em `fato_vendas`;
- medidas de item baseadas em `fato_vendas_detalhes`;
- medidas de margem marcadas como estimadas;
- desconto/variacao mantido como indicador em estudo ate validacao de negocio.

Confirmacao final: nenhuma estrutura do banco foi alterada nesta etapa.
