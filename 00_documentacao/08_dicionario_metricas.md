# Dicionário Oficial de Métricas Analíticas

Data da documentação: 2026-06-29

## Objetivo

Este documento define a referência oficial para as métricas analíticas do projeto. Seu objetivo é padronizar o cálculo de indicadores que serão usados em Power BI, Data Marts e Dashboards, eliminando ambiguidades sobre fatos, dimensões, campos, granularidade e cuidados de uso.

Esta etapa é exclusivamente documental. Nenhuma consulta foi executada e nenhuma estrutura do banco foi alterada.

## Regras gerais de governança

- Métricas de nota fiscal devem usar `data_warehouse.fato_vendas`.
- Métricas de produto, categoria, quantidade, custo e margem devem usar `data_warehouse.fato_vendas_detalhes`.
- Não somar medidas de cabecalho após expandir a venda para itens.
- Não somar `valor_nota_fiscal` diretamente na fato de detalhes.
- Usar `DISTINCT` para contagens de notas, clientes e produtos quando houver risco de repetição.
- Manter métricas estimadas identificadas claramente no Power BI.
- Não oficializar desconto ou margem financeira definitiva sem validar a semântica de `valor_custo`, `valor_venda`, `valor_unitario` e `valor_venda_real`.

## Métricas Financeiras

### 1. Faturamento Total

**Objetivo**

Medir a receita bruta total de vendas no período analisado.

**Descrição Técnica**

Soma do valor total da nota fiscal no grao de cabecalho. Para análises por produto ou categoria, usar a soma do valor real de venda no grao de item.

**Tabela Fato**

- `fato_vendas`, preferencial para visão executiva e nota fiscal.
- `fato_vendas_detalhes`, para visão de produto/categoria.

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`, somente com `fato_vendas`
- `dim_produto`, somente com `fato_vendas_detalhes`

**Campo(s) utilizado(s)**

- `valor`
- `valor_venda_real`

**Fórmula SQL**

```sql
-- Grao de nota fiscal
sum(fato_vendas.valor)

-- Grao de item/produto
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento Total =
SUM(fato_vendas[valor])

Faturamento Total Itens =
SUM(fato_vendas_detalhes[valor_venda_real])
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal, quando usar `fato_vendas`.
- Item da Nota Fiscal, quando usar `fato_vendas_detalhes`.

**Tipo**

- KPI

**Dependências**

- Depende da fato escolhida e das dimensões conectadas ao grao correspondente.

**Cuidados**

- Não somar `fato_vendas[valor]` após relacionamento com itens.
- Não somar `valor_nota_fiscal` no detalhe.

### 2. Valor Real de Venda

**Objetivo**

Representar o valor efetivo vendido no nível de item.

**Descrição Técnica**

Soma de `valor_venda_real` na fato de detalhes.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_produto`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Valor Real de Venda =
SUM(fato_vendas_detalhes[valor_venda_real])
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Métrica Base

**Dependências**

- Depende de `fato_vendas_detalhes`.

**Cuidados**

- Esta métrica é a base correta para análises por produto, categoria e margem estimada.

### 3. Valor da Nota Fiscal

**Objetivo**

Representar o valor total da venda no nível de nota fiscal.

**Descrição Técnica**

Soma do campo `valor` em `fato_vendas`.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`

**Campo(s) utilizado(s)**

- `valor`

**Fórmula SQL**

```sql
sum(fato_vendas.valor)
```

**Equivalente DAX**

```DAX
Valor da Nota Fiscal =
SUM(fato_vendas[valor])
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal

**Tipo**

- Métrica Base

**Dependências**

- Depende de `fato_vendas`.

**Cuidados**

- Não usar em visuais cujo eixo principal venha de `dim_produto`, a menos que exista modelagem controlada por nota fiscal.

### 4. Custo Estimado

**Objetivo**

Estimar o custo total dos itens vendidos.

**Descrição Técnica**

Multiplica o custo do item pela quantidade vendida e soma o resultado.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_custo`
- `quantidade`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_custo * fato_vendas_detalhes.quantidade)
```

**Equivalente DAX**

```DAX
Custo Estimado =
SUMX(
    fato_vendas_detalhes,
    fato_vendas_detalhes[valor_custo] * fato_vendas_detalhes[quantidade]
)
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Estimado

**Dependências**

- Depende da confirmação de que `valor_custo` representa custo unitário.

**Cuidados**

- Se `valor_custo` já representar custo total da linha, a multiplicação por quantidade superestima o custo.

### 5. Lucro Bruto Estimado

**Objetivo**

Estimar o resultado bruto das vendas após custo dos produtos.

**Descrição Técnica**

Diferença entre o valor real vendido e o custo estimado.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `valor_custo`
- `quantidade`

**Fórmula SQL**

```sql
sum(
  fato_vendas_detalhes.valor_venda_real
  - fato_vendas_detalhes.valor_custo * fato_vendas_detalhes.quantidade
)
```

**Equivalente DAX**

```DAX
Lucro Bruto Estimado =
SUMX(
    fato_vendas_detalhes,
    fato_vendas_detalhes[valor_venda_real]
        - fato_vendas_detalhes[valor_custo] * fato_vendas_detalhes[quantidade]
)
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Estimado

**Dependências**

- Depende de `Valor Real de Venda` e `Custo Estimado`.

**Cuidados**

- Deve ser rotulado como estimado até validação de negócio sobre `valor_custo`.

### 6. Margem Bruta Estimada

**Objetivo**

Medir o percentual estimado de lucro bruto sobre o faturamento.

**Descrição Técnica**

Divide o lucro bruto estimado pelo valor real de venda.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `valor_custo`
- `quantidade`

**Fórmula SQL**

```sql
sum(valor_venda_real - valor_custo * quantidade)
/ nullif(sum(valor_venda_real), 0)
```

**Equivalente DAX**

```DAX
Margem Bruta Estimada % =
DIVIDE([Lucro Bruto Estimado], [Valor Real de Venda])
```

**Unidade**

- %

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Estimado

**Dependências**

- Depende de `Lucro Bruto Estimado` e `Valor Real de Venda`.

**Cuidados**

- Não calcular com base em `valor_nota_fiscal`.
- Usar `DIVIDE` no DAX para evitar erro de divisão por zero.

### 7. Ticket Médio

**Objetivo**

Medir o valor médio de cada venda.

**Descrição Técnica**

Divide o faturamento de cabecalho pela quantidade distinta de notas fiscais.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`

**Campo(s) utilizado(s)**

- `valor`
- `numero_nf`

**Fórmula SQL**

```sql
sum(fato_vendas.valor)
/ nullif(count(distinct fato_vendas.numero_nf), 0)
```

**Equivalente DAX**

```DAX
Ticket Médio =
DIVIDE([Faturamento Total], [Número de Vendas])
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal

**Tipo**

- KPI

**Dependências**

- Depende de `Faturamento Total` e `Número de Vendas`.

**Cuidados**

- Não calcular ticket médio a partir de linhas de item sem deduplicar notas.

### 8. Preço Médio Praticado

**Objetivo**

Medir o preço médio efetivo por unidade vendida.

**Descrição Técnica**

Divide o valor real de venda pela quantidade vendida.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `quantidade`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
/ nullif(sum(fato_vendas_detalhes.quantidade), 0)
```

**Equivalente DAX**

```DAX
Preço Médio Praticado =
DIVIDE([Valor Real de Venda], [Quantidade Vendida])
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Derivado

**Dependências**

- Depende de `Valor Real de Venda` e `Quantidade Vendida`.

**Cuidados**

- Preferir média ponderada por quantidade, não `AVERAGE(valor_unitario)`.

### 9. Variação Comercial Estimada

**Objetivo**

Avaliar possível diferença entre valor referencial e valor real praticado.

**Descrição Técnica**

Calcula a diferença entre `valor_venda` e `valor_venda_real`.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda`
- `valor_venda_real`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda - fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Variação Comercial Estimada =
SUMX(
    fato_vendas_detalhes,
    fato_vendas_detalhes[valor_venda]
        - fato_vendas_detalhes[valor_venda_real]
)
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Estimado

**Dependências**

- Depende da semântica de `valor_venda` e `valor_venda_real`.

**Cuidados**

- Não tratar como desconto oficial. A validação anterior indicou comportamento incompatível com leitura simples de desconto.

## Métricas Comerciais

### 10. Número de Vendas

**Objetivo**

Medir a quantidade de transações comerciais.

**Descrição Técnica**

Contagem distinta de notas fiscais.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_forma_pagamento`

**Campo(s) utilizado(s)**

- `numero_nf`

**Fórmula SQL**

```sql
count(distinct fato_vendas.numero_nf)
```

**Equivalente DAX**

```DAX
Número de Vendas =
DISTINCTCOUNT(fato_vendas[numero_nf])
```

**Unidade**

- Vendas

**Granularidade**

- Nota Fiscal

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas`.

**Cuidados**

- Em cruzamentos com item, manter `DISTINCTCOUNT`.

### 11. Clientes Ativos

**Objetivo**

Medir quantos clientes compraram no período.

**Descrição Técnica**

Contagem distinta de clientes presentes nas fatos de venda.

**Tabela Fato**

- `fato_vendas`
- `fato_vendas_detalhes`, para análises de produto/categoria

**Dimensões relacionadas**

- `dim_cliente`
- `dim_tempo`
- `dim_produto`, quando usar detalhe
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `id_cliente`

**Fórmula SQL**

```sql
count(distinct fato_vendas.id_cliente)
```

**Equivalente DAX**

```DAX
Clientes Ativos =
DISTINCTCOUNT(fato_vendas[id_cliente])
```

**Unidade**

- Clientes

**Granularidade**

- Nota Fiscal

**Tipo**

- KPI

**Dependências**

- Depende da chave `id_cliente`.

**Cuidados**

- Contagem simples de linhas não representa clientes.

### 12. Clientes por Vendedor

**Objetivo**

Medir a amplitude da carteira atendida por vendedor.

**Descrição Técnica**

Contagem distinta de clientes por contexto de vendedor.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_vendedor`
- `dim_cliente`
- `dim_tempo`

**Campo(s) utilizado(s)**

- `id_cliente`
- `id_vendedor`

**Fórmula SQL**

```sql
count(distinct fato_vendas.id_cliente)
```

**Equivalente DAX**

```DAX
Clientes por Vendedor =
DISTINCTCOUNT(fato_vendas[id_cliente])
```

**Unidade**

- Clientes

**Granularidade**

- Vendedor / Nota Fiscal

**Tipo**

- Indicador Derivado

**Dependências**

- Depende do filtro de `dim_vendedor`.

**Cuidados**

- Um cliente pode aparecer em mais de um vendedor; não somar clientes por vendedor para obter total geral.

### 13. Faturamento por Vendedor

**Objetivo**

Comparar resultado financeiro individual da equipe comercial.

**Descrição Técnica**

Soma do faturamento de cabecalho no contexto de vendedor.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_vendedor`
- `dim_tempo`
- `dim_cliente`
- `dim_forma_pagamento`

**Campo(s) utilizado(s)**

- `valor`
- `id_vendedor`

**Fórmula SQL**

```sql
sum(fato_vendas.valor)
```

**Equivalente DAX**

```DAX
Faturamento por Vendedor =
[Faturamento Total]
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal / Vendedor

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas` e `dim_vendedor`.

**Cuidados**

- A medida DAX pode ser a mesma de faturamento total, respeitando o contexto de filtro do vendedor.

### 14. Ticket Médio por Vendedor

**Objetivo**

Avaliar o valor médio das vendas por vendedor.

**Descrição Técnica**

Divide o faturamento do vendedor pelo número de notas fiscais do vendedor.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_vendedor`
- `dim_tempo`
- `dim_cliente`

**Campo(s) utilizado(s)**

- `valor`
- `numero_nf`
- `id_vendedor`

**Fórmula SQL**

```sql
sum(fato_vendas.valor)
/ nullif(count(distinct fato_vendas.numero_nf), 0)
```

**Equivalente DAX**

```DAX
Ticket Médio por Vendedor =
DIVIDE([Faturamento Total], [Número de Vendas])
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal / Vendedor

**Tipo**

- Indicador Derivado

**Dependências**

- Depende do filtro de vendedor aplicado ao modelo.

**Cuidados**

- Não criar uma fórmula separada se `Ticket Médio` já respeitar o contexto de vendedor.

### 15. Faturamento por Forma de Pagamento

**Objetivo**

Medir a distribuição das vendas por modalidade de pagamento.

**Descrição Técnica**

Soma de `valor` no contexto de `dim_forma_pagamento`.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_forma_pagamento`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor`
- `id_forma_pagamento`

**Fórmula SQL**

```sql
sum(fato_vendas.valor)
```

**Equivalente DAX**

```DAX
Faturamento por Forma de Pagamento =
[Faturamento Total]
```

**Unidade**

- R$

**Granularidade**

- Nota Fiscal

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas` e `dim_forma_pagamento`.

**Cuidados**

- Forma de pagamento não está diretamente disponível em `fato_vendas_detalhes`.

## Métricas de Produtos

### 16. Quantidade Vendida

**Objetivo**

Medir o volume físico de unidades vendidas.

**Descrição Técnica**

Soma da quantidade vendida nos itens de nota fiscal.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `quantidade`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.quantidade)
```

**Equivalente DAX**

```DAX
Quantidade Vendida =
SUM(fato_vendas_detalhes[quantidade])
```

**Unidade**

- Quantidade

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Métrica Base

**Dependências**

- Depende de `fato_vendas_detalhes`.

**Cuidados**

- Não substituir por contagem de linhas.

### 17. Itens por Venda

**Objetivo**

Medir a quantidade média de linhas de item por nota fiscal.

**Descrição Técnica**

Divide o total de linhas de detalhe pela quantidade distinta de notas fiscais.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `id`
- `numero_nf`

**Fórmula SQL**

```sql
count(*)
/ nullif(count(distinct fato_vendas_detalhes.numero_nf), 0)
```

**Equivalente DAX**

```DAX
Itens por Venda =
DIVIDE(
    COUNTROWS(fato_vendas_detalhes),
    DISTINCTCOUNT(fato_vendas_detalhes[numero_nf])
)
```

**Unidade**

- Itens

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Derivado

**Dependências**

- Depende de linhas de detalhe e notas fiscais distintas.

**Cuidados**

- Mede linhas de item, não unidades vendidas.

### 18. Produtos Vendidos

**Objetivo**

Medir quantos produtos distintos tiveram vendas.

**Descrição Técnica**

Contagem distinta de `id_produto` presente na fato de detalhes.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `id_produto`

**Fórmula SQL**

```sql
count(distinct fato_vendas_detalhes.id_produto)
```

**Equivalente DAX**

```DAX
Produtos Vendidos =
DISTINCTCOUNT(fato_vendas_detalhes[id_produto])
```

**Unidade**

- Produtos

**Granularidade**

- Item da Nota Fiscal

**Tipo**

- Indicador Derivado

**Dependências**

- Depende de `fato_vendas_detalhes` e `dim_produto`.

**Cuidados**

- Usar chave do produto, não nome, pois nomes podem se repetir.

### 19. Faturamento por Categoria

**Objetivo**

Medir a contribuição financeira de cada categoria de produto.

**Descrição Técnica**

Soma de `valor_venda_real` no contexto de categoria.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `categoria`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento por Categoria =
[Valor Real de Venda]
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal / Categoria

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas_detalhes` e `dim_produto[categoria]`.

**Cuidados**

- Categoria só deve ser analisada no grao de item/produto.

### 20. Participação da Categoria

**Objetivo**

Medir o peso percentual da categoria no faturamento.

**Descrição Técnica**

Divide o faturamento da categoria pelo faturamento total no contexto selecionado.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `categoria`

**Fórmula SQL**

```sql
sum(valor_venda_real)
/ nullif(sum(sum(valor_venda_real)) over (), 0)
```

**Equivalente DAX**

```DAX
Participação da Categoria % =
DIVIDE(
    [Valor Real de Venda],
    CALCULATE([Valor Real de Venda], ALL(dim_produto[categoria]))
)
```

**Unidade**

- %

**Granularidade**

- Categoria

**Tipo**

- Indicador Derivado

**Dependências**

- Depende de `Valor Real de Venda` e do contexto de filtro de categoria.

**Cuidados**

- Definir se o denominador deve ignorar apenas categoria ou todos os filtros de produto.

### 21. Faturamento por Produto

**Objetivo**

Medir a receita gerada por cada produto.

**Descrição Técnica**

Soma de `valor_venda_real` no contexto de produto.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_produto`
- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `id_produto`
- `produto`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento por Produto =
[Valor Real de Venda]
```

**Unidade**

- R$

**Granularidade**

- Produto / Item da Nota Fiscal

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas_detalhes` e `dim_produto`.

**Cuidados**

- Usar `dim_produto[id]` como identificador único.

## Métricas Geográficas

### 22. Faturamento por Estado

**Objetivo**

Medir a distribuição do faturamento por UF do cliente.

**Descrição Técnica**

Soma de `valor_venda_real` no contexto de estado.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_cliente`
- `dim_tempo`
- `dim_produto`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `estado`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento por Estado =
[Valor Real de Venda]
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal / Estado

**Tipo**

- KPI

**Dependências**

- Depende de `fato_vendas_detalhes` e `dim_cliente[estado]`.

**Cuidados**

- O modelo tem forte concentração no Ceará; usar ranking, percentuais e filtros temporais para análise executiva.

### 23. Faturamento por Cidade

**Objetivo**

Medir a distribuição do faturamento por cidade do cliente.

**Descrição Técnica**

Soma de `valor_venda_real` no contexto de cidade.

**Tabela Fato**

- `fato_vendas_detalhes`

**Dimensões relacionadas**

- `dim_cliente`
- `dim_tempo`
- `dim_produto`
- `dim_vendedor`

**Campo(s) utilizado(s)**

- `valor_venda_real`
- `cidade`

**Fórmula SQL**

```sql
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento por Cidade =
[Valor Real de Venda]
```

**Unidade**

- R$

**Granularidade**

- Item da Nota Fiscal / Cidade

**Tipo**

- Indicador Derivado

**Dependências**

- Depende de `fato_vendas_detalhes` e `dim_cliente[cidade]`.

**Cuidados**

- Cidades com mesmo nome em estados diferentes devem ser analisadas em conjunto com `estado`.

## Métricas Temporais

### 24. Faturamento por Período

**Objetivo**

Acompanhar evolução temporal das vendas.

**Descrição Técnica**

Soma do faturamento no contexto dos atributos da dimensão tempo.

**Tabela Fato**

- `fato_vendas`, para nota fiscal.
- `fato_vendas_detalhes`, para produto/categoria.

**Dimensões relacionadas**

- `dim_tempo`
- `dim_cliente`
- `dim_vendedor`
- `dim_produto`, no detalhe
- `dim_forma_pagamento`, no cabecalho

**Campo(s) utilizado(s)**

- `valor`
- `valor_venda_real`
- `id_data_venda`
- `id_tempo`

**Fórmula SQL**

```sql
-- Cabecalho
sum(fato_vendas.valor)

-- Detalhe
sum(fato_vendas_detalhes.valor_venda_real)
```

**Equivalente DAX**

```DAX
Faturamento por Período =
[Faturamento Total]
```

**Unidade**

- R$

**Granularidade**

- Dia, mês, ano ou outro agrupamento temporal.

**Tipo**

- KPI

**Dependências**

- Depende de `dim_tempo`.

**Cuidados**

- Trimestre e semestre devem ser derivados no Power BI ou em Data Mart futuro, pois não estão materializados na dimensão.

### 25. Dias com Venda

**Objetivo**

Medir quantos dias tiveram movimento de venda.

**Descrição Técnica**

Contagem distinta de datas com venda.

**Tabela Fato**

- `fato_vendas`

**Dimensões relacionadas**

- `dim_tempo`

**Campo(s) utilizado(s)**

- `id_data_venda`
- `data`

**Fórmula SQL**

```sql
count(distinct dim_tempo.data)
```

**Equivalente DAX**

```DAX
Dias com Venda =
DISTINCTCOUNT(dim_tempo[data])
```

**Unidade**

- Dias

**Granularidade**

- Dia

**Tipo**

- Indicador Derivado

**Dependências**

- Depende do relacionamento entre `fato_vendas[id_data_venda]` e `dim_tempo[id]`.

**Cuidados**

- A medida deve ser filtrada pela fato; contar todas as datas da dimensão tempo retorna o calendário completo, não dias com venda.

## Classificação das Métricas

### Financeiras

- Faturamento Total
- Valor Real de Venda
- Valor da Nota Fiscal
- Custo Estimado
- Lucro Bruto Estimado
- Margem Bruta Estimada
- Ticket Médio
- Preço Médio Praticado
- Variação Comercial Estimada
- Faturamento por Forma de Pagamento

### Comerciais

- Número de Vendas
- Clientes Ativos
- Clientes por Vendedor
- Faturamento por Vendedor
- Ticket Médio por Vendedor

### Produtos

- Quantidade Vendida
- Itens por Venda
- Produtos Vendidos
- Faturamento por Categoria
- Participação da Categoria
- Faturamento por Produto

### Geográficas

- Faturamento por Estado
- Faturamento por Cidade

### Temporais

- Faturamento por Período
- Dias com Venda
- Ano
- Mês
- Dia
- Trimestre, derivado
- Semestre, derivado

## Tabela Resumo

| Métrica | Tipo | Tabela | Granularidade | Unidade |
| --- | --- | --- | --- | --- |
| Faturamento Total | KPI | `fato_vendas` / `fato_vendas_detalhes` | Nota Fiscal ou Item | R$ |
| Valor Real de Venda | Métrica Base | `fato_vendas_detalhes` | Item da Nota Fiscal | R$ |
| Valor da Nota Fiscal | Métrica Base | `fato_vendas` | Nota Fiscal | R$ |
| Custo Estimado | Indicador Estimado | `fato_vendas_detalhes` | Item da Nota Fiscal | R$ |
| Lucro Bruto Estimado | Indicador Estimado | `fato_vendas_detalhes` | Item da Nota Fiscal | R$ |
| Margem Bruta Estimada | Indicador Estimado | `fato_vendas_detalhes` | Item da Nota Fiscal | % |
| Ticket Médio | KPI | `fato_vendas` | Nota Fiscal | R$ |
| Preço Médio Praticado | Indicador Derivado | `fato_vendas_detalhes` | Item da Nota Fiscal | R$ |
| Variação Comercial Estimada | Indicador Estimado | `fato_vendas_detalhes` | Item da Nota Fiscal | R$ |
| Número de Vendas | KPI | `fato_vendas` | Nota Fiscal | Vendas |
| Clientes Ativos | KPI | `fato_vendas` / `fato_vendas_detalhes` | Cliente | Clientes |
| Clientes por Vendedor | Indicador Derivado | `fato_vendas` | Vendedor / Nota Fiscal | Clientes |
| Faturamento por Vendedor | KPI | `fato_vendas` | Vendedor / Nota Fiscal | R$ |
| Ticket Médio por Vendedor | Indicador Derivado | `fato_vendas` | Vendedor / Nota Fiscal | R$ |
| Faturamento por Forma de Pagamento | KPI | `fato_vendas` | Nota Fiscal | R$ |
| Quantidade Vendida | Métrica Base | `fato_vendas_detalhes` | Item da Nota Fiscal | Quantidade |
| Itens por Venda | Indicador Derivado | `fato_vendas_detalhes` | Item / Nota Fiscal | Itens |
| Produtos Vendidos | Indicador Derivado | `fato_vendas_detalhes` | Produto | Produtos |
| Faturamento por Categoria | KPI | `fato_vendas_detalhes` | Categoria / Item | R$ |
| Participação da Categoria | Indicador Derivado | `fato_vendas_detalhes` | Categoria | % |
| Faturamento por Produto | KPI | `fato_vendas_detalhes` | Produto / Item | R$ |
| Faturamento por Estado | KPI | `fato_vendas_detalhes` | Estado / Item | R$ |
| Faturamento por Cidade | Indicador Derivado | `fato_vendas_detalhes` | Cidade / Item | R$ |
| Faturamento por Período | KPI | `fato_vendas` / `fato_vendas_detalhes` | Tempo | R$ |
| Dias com Venda | Indicador Derivado | `fato_vendas` | Dia | Dias |

## Boas práticas para Power BI

- Criar uma tabela de medidas centralizada no modelo Power BI.
- Nomear medidas de forma idêntica a este dicionário.
- Evitar cálculos diretamente nos visuais.
- Priorizar medidas DAX reutilizáveis.
- Separar medidas de cabecalho e medidas de item.
- Nunca misturar fatos de granularidades diferentes sem uma regra explícita.
- Sempre usar `DISTINCTCOUNT` para notas fiscais, clientes e produtos quando houver risco de duplicidade.
- Evitar somar medidas repetidas após joins ou relacionamentos muitos-para-muitos.
- Rotular medidas estimadas com o sufixo "Estimado" ou "Estimada".
- Usar `DIVIDE()` em medidas percentuais e médias.
- Documentar qualquer alteração futura de fórmula, campo, fato ou granularidade.
- Validar totais do Power BI contra os totais documentados em `07_validacao_kpis.md`.

## Conclusão

O conjunto atual de métricas cobre os objetivos do projeto para análise comercial, financeira, temporal, geográfica, de produtos, vendedores, clientes e formas de pagamento.

Existem algumas métricas conceitualmente próximas, como `Faturamento Total`, `Valor da Nota Fiscal` e `Valor Real de Venda`. Elas não devem ser tratadas como redundantes de forma automática, pois representam graos diferentes: nota fiscal e item da nota fiscal.

As métricas que devem permanecer apenas como estimativas são:

- Custo Estimado;
- Lucro Bruto Estimado;
- Margem Bruta Estimada;
- Variação Comercial Estimada.

As métricas oficiais para uso imediato são:

- Faturamento Total;
- Valor Real de Venda;
- Valor da Nota Fiscal;
- Número de Vendas;
- Clientes Ativos;
- Quantidade Vendida;
- Ticket Médio;
- Preço Médio Praticado;
- Faturamento por Vendedor;
- Faturamento por Categoria;
- Participação da Categoria;
- Faturamento por Produto;
- Faturamento por Estado;
- Faturamento por Cidade;
- Faturamento por Forma de Pagamento;
- Faturamento por Período;
- Dias com Venda.

O projeto está preparado para iniciar a modelagem do Power BI, desde que as medidas sejam criadas de forma centralizada, com separação clara entre grao de nota fiscal e grao de item da nota fiscal.

Confirmação final: nenhuma estrutura do banco foi alterada nesta etapa.
