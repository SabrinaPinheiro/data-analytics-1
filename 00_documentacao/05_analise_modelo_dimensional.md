# Analise do modelo dimensional

Data da analise: 2026-06-28

## Objetivo

Documentar o modelo dimensional restaurado no schema `data_warehouse`, descrevendo granularidade, dimensoes, fatos, medidas, atributos, relacionamentos, vantagens, melhorias possiveis e cuidados para evitar duplicidade de metricas.

Esta etapa e exclusivamente documental. Nenhuma estrutura do banco foi alterada.

## Visao geral

O schema `data_warehouse` possui um modelo dimensional orientado a vendas. A modelagem esta proxima de um Star Schema, com dimensoes ao redor de duas tabelas fato:

- `fato_vendas`: fato no nivel de cabecalho da nota fiscal;
- `fato_vendas_detalhes`: fato no nivel de item da nota fiscal.

As dimensoes disponiveis sao:

- `dim_cliente`;
- `dim_produto`;
- `dim_tempo`;
- `dim_vendedor`;
- `dim_forma_pagamento`.

O arquivo visual correspondente e:

- [03B_star_schema.mmd](03B_star_schema.mmd)
- [03B_star_schema.png](03B_star_schema.png)

## Granularidade das fatos

### `fato_vendas`

Granularidade: uma linha por nota fiscal/venda.

Evidencias:

- Registros: 25.391.
- `id` distintos: 25.391.
- `numero_nf` distintos: 25.391.

Essa fato e adequada para analises de cabecalho, como:

- valor total da nota;
- quantidade de notas;
- ticket medio;
- vendas por cliente;
- vendas por vendedor;
- vendas por forma de pagamento;
- vendas por data.

Medida principal:

| Medida | Descricao | Uso recomendado |
| --- | --- | --- |
| `valor` | Valor total da nota fiscal | Faturamento no nivel de nota |

### `fato_vendas_detalhes`

Granularidade: uma linha por item/produto dentro da nota fiscal.

Evidencias:

- Registros: 35.752.
- `id` distintos: 35.752.
- `numero_nf` distintos: 25.391.
- Ha mais linhas de detalhe do que notas fiscais.

Essa fato e adequada para analises detalhadas, como:

- produto;
- categoria;
- quantidade vendida;
- valor vendido por item;
- custo;
- margem estimada;
- desempenho por vendedor no nivel de item;
- cliente e localidade no nivel de item.

Medidas principais:

| Medida | Descricao | Uso recomendado |
| --- | --- | --- |
| `quantidade` | Quantidade de unidades vendidas | Volume vendido |
| `valor_venda_real` | Valor efetivo da venda do item | Faturamento por item |
| `valor_unitario` | Valor unitario praticado | Preco medio/unitario |
| `valor_venda` | Valor de venda cadastrado ou referencial | Comparacao com valor efetivo |
| `valor_custo` | Custo unitario ou custo referencial | Analise de custo/margem |
| `valor_nota_fiscal` | Valor total da nota fiscal repetido no detalhe | Usar com deduplicacao por nota |

## Dimensoes

### `dim_cliente`

- Registros: 1.000.
- Chave primaria: `id`.
- Atributos:
  - `nome`;
  - `cidade`;
  - `estado`.

Papel analitico:

- analisar vendas por cliente;
- analisar vendas por cidade;
- analisar vendas por estado;
- apoiar segmentacoes geograficas.

Origem conceitual:

- `geral.pessoa`;
- `geral.pessoa_fisica`;
- `geral.pessoa_juridica`;
- `geral.endereco`;
- `geral.bairro`;
- `geral.cidade`;
- `geral.estado`.

### `dim_produto`

- Registros: 233.
- Chave primaria: `id`.
- Atributos:
  - `produto`;
  - `categoria`.

Papel analitico:

- analisar vendas por produto;
- analisar vendas por categoria;
- apoiar ranking de produtos;
- apoiar analise de mix de vendas.

Origem conceitual:

- `vendas.produto`;
- `vendas.categoria`.

### `dim_tempo`

- Registros: 12.785.
- Chave primaria: `id`.
- Atributos:
  - `data`;
  - `ano`;
  - `mes`;
  - `dia`;
  - `dia_da_semana`;
  - `mes_extenso`.

Periodo total da dimensao:

- primeira data: 1994-08-22;
- ultima data: 2029-08-22.

Periodo com vendas nas fatos:

- primeira venda: 2015-01-01;
- ultima venda: 2017-12-27.

Papel analitico:

- analisar vendas por dia;
- analisar vendas por mes;
- analisar vendas por ano;
- criar series temporais;
- apoiar analises de sazonalidade.

### `dim_vendedor`

- Registros: 24.
- Chave primaria: `id`.
- Atributos:
  - `nome`.

Papel analitico:

- analisar desempenho por vendedor;
- ranquear vendedores por faturamento;
- cruzar vendedor com cliente, produto, categoria e tempo.

Origem conceitual:

- pessoas cadastradas no schema `geral` que aparecem como vendedores nas vendas.

### `dim_forma_pagamento`

- Registros: 9.
- Chave primaria: `id`.
- Atributos:
  - `descricao`.

Papel analitico:

- analisar faturamento por forma de pagamento;
- comparar distribuicao de pagamentos;
- apoiar indicadores financeiros.

Observacao:

- esta dimensao se relaciona diretamente com `fato_vendas`;
- nao existe FK direta de `fato_vendas_detalhes` para `dim_forma_pagamento`.

## Relacionamentos entre fatos e dimensoes

### `fato_vendas`

| FK na fato | Dimensao | Relacionamento |
| --- | --- | --- |
| `id_cliente` | `dim_cliente(id)` | N:1 |
| `id_vendedor` | `dim_vendedor(id)` | N:1 |
| `id_forma_pagamento` | `dim_forma_pagamento(id)` | N:1 |
| `id_data_venda` | `dim_tempo(id)` | N:1 |

Uso principal:

- analises de venda no nivel da nota fiscal.

### `fato_vendas_detalhes`

| FK na fato | Dimensao | Relacionamento |
| --- | --- | --- |
| `id_produto` | `dim_produto(id)` | N:1 |
| `id_cliente` | `dim_cliente(id)` | N:1 |
| `id_vendedor` | `dim_vendedor(id)` | N:1 |
| `id_tempo` | `dim_tempo(id)` | N:1 |

Uso principal:

- analises de venda no nivel de item/produto.

## Indicadores observados

### Fato de cabecalho

| Indicador | Valor |
| --- | ---: |
| Registros em `fato_vendas` | 25.391 |
| Notas fiscais distintas | 25.391 |
| Clientes distintos | 1.000 |
| Vendedores distintos | 24 |
| Formas de pagamento usadas | 5 |
| Datas distintas usadas | 972 |
| Valor total | 88.562.548,00 |
| Valor medio por nota | 3.487,95 |

### Fato de detalhe

| Indicador | Valor |
| --- | ---: |
| Registros em `fato_vendas_detalhes` | 35.752 |
| Notas fiscais distintas | 25.391 |
| Produtos distintos usados | 226 |
| Clientes distintos | 1.000 |
| Vendedores distintos | 24 |
| Datas distintas usadas | 972 |
| Quantidade total | 89.527 |
| Soma de `valor_venda_real` | 88.562.548,00 |
| Soma de `valor_venda` | 35.520.358,00 |
| Soma de `valor_custo` | 26.243.769,00 |
| Soma de `valor_nota_fiscal` | 173.555.131,00 |
| Margem bruta estimada | 23.070.617,00 |

## Vantagens do modelo adotado

1. Separacao clara entre camada relacional/transacional e camada analitica.
2. Dimensoes simples e compreensiveis para consumo em BI.
3. Duas granularidades de fato permitem analises de cabecalho e detalhe.
4. A view `public.view_dt_mart_vendas` facilita consumo direto por ferramentas como Power BI.
5. As FKs do `data_warehouse` estao validadas, aumentando confianca nos relacionamentos dimensionais.
6. O modelo suporta analises por tempo, cliente, localidade, vendedor, produto e categoria.

## Possiveis melhorias

1. Criar uma chave explicita entre `fato_vendas` e `fato_vendas_detalhes`, preferencialmente uma chave da nota fiscal, para reduzir dependencia de `numero_nf` textual.
2. Incluir `id_forma_pagamento` tambem em `fato_vendas_detalhes`, caso seja necessario analisar produto por forma de pagamento sem joins adicionais.
3. Separar melhor medidas de cabecalho e medidas de detalhe para evitar uso indevido de `valor_nota_fiscal`.
4. Validar semanticamente `valor_venda`, `valor_unitario`, `valor_venda_real` e `valor_custo`.
5. Adicionar atributos extras em `dim_tempo`, como trimestre, semestre, numero do mes e indicador de fim de semana.
6. Avaliar uma dimensao geografica separada se cidade/estado crescerem em complexidade.
7. Avaliar se `public.view_dt_mart_vendas` deve permanecer no schema `public` ou ser exposta de forma mais controlada.

## Cuidados para evitar duplicidade de metricas

### Nao somar fatos de granularidades diferentes sem controle

`fato_vendas` e `fato_vendas_detalhes` podem apresentar o mesmo faturamento total quando agregadas corretamente, mas representam graos diferentes:

- `fato_vendas`: uma linha por nota;
- `fato_vendas_detalhes`: uma linha por item.

Combinar ambas diretamente pode duplicar valores.

### Cuidado com `valor_nota_fiscal`

Na `fato_vendas_detalhes`, `valor_nota_fiscal` representa um valor de cabecalho repetido nas linhas de item. Portanto:

- errado: `sum(valor_nota_fiscal)` diretamente no nivel de item;
- correto: deduplicar por `numero_nf` ou usar `fato_vendas.valor`.

### Forma de pagamento e produto

Forma de pagamento esta em `fato_vendas`, enquanto produto esta em `fato_vendas_detalhes`.

Para analisar produto por forma de pagamento:

1. juntar as fatos por `numero_nf`;
2. manter medidas de item vindas de `fato_vendas_detalhes`;
3. usar a forma de pagamento apenas como atributo de cabecalho;
4. nao somar `fato_vendas.valor` apos expandir para itens.

### Medidas de custo

O campo `valor_custo` parece representar custo unitario ou referencial do produto. Para margem estimada, a formula mais prudente e:

```sql
sum(valor_venda_real - valor_custo * quantidade)
```

Essa formula deve ser confirmada antes de virar indicador oficial.

## Conclusao

O modelo dimensional restaurado e adequado para iniciar os Data Marts de vendas. A melhor tabela para analises detalhadas e `fato_vendas_detalhes`; a melhor tabela para indicadores de cabecalho e forma de pagamento e `fato_vendas`.

O principal cuidado tecnico e respeitar a granularidade de cada fato. Com essa disciplina, o modelo oferece uma base solida para Power BI, metricas financeiras e storytelling analitico.
