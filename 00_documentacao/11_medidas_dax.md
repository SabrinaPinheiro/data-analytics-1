# Medidas DAX

Data da documentação: 2026-06-30

## Objetivo

Documentar as principais medidas DAX que serão criadas no Power BI para o dashboard final.

As medidas devem ser centralizadas no Power BI e reutilizadas nos cartões, gráficos e tabelas do relatório.

Esta etapa é apenas documental. Nenhuma estrutura do banco foi alterada.

## Medidas principais

### 1. Faturamento Total

Tabela recomendada: `fato_vendas`

```DAX
Faturamento Total =
SUM(fato_vendas[valor])
```

Observação: usar para análises no nível de nota fiscal, vendedor, cliente, tempo e forma de pagamento.

### 2. Número de Vendas

Tabela recomendada: `fato_vendas`

```DAX
Número de Vendas =
DISTINCTCOUNT(fato_vendas[numero_nf])
```

Observação: usar `DISTINCTCOUNT` para evitar duplicidade.

### 3. Ticket Médio

Tabela recomendada: `fato_vendas`

```DAX
Ticket Médio =
DIVIDE([Faturamento Total], [Número de Vendas])
```

Observação: representa o valor médio por nota fiscal.

### 4. Clientes Ativos

Tabela recomendada: `fato_vendas`

```DAX
Clientes Ativos =
DISTINCTCOUNT(fato_vendas[id_cliente])
```

Observação: conta clientes que realizaram compras no período filtrado.

### 5. Quantidade Vendida

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Quantidade Vendida =
SUM(fato_vendas_detalhes[quantidade])
```

Observação: deve ser usada para análises de produto, categoria e volume vendido.

### 6. Valor Real de Venda

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Valor Real de Venda =
SUM(fato_vendas_detalhes[valor_venda_real])
```

Observação: medida de apoio para análises no nível de item.

### 7. Preço Médio Praticado

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Preço Médio Praticado =
DIVIDE([Valor Real de Venda], [Quantidade Vendida])
```

Observação: calcula o preço médio ponderado pela quantidade vendida.

### 8. Custo Estimado

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Custo Estimado =
SUMX(
    fato_vendas_detalhes,
    fato_vendas_detalhes[valor_custo] * fato_vendas_detalhes[quantidade]
)
```

Observação: medida de apoio para margem. Deve ser tratada como estimativa.

### 9. Margem Bruta Estimada

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Margem Bruta Estimada =
SUMX(
    fato_vendas_detalhes,
    fato_vendas_detalhes[valor_venda_real]
        - fato_vendas_detalhes[valor_custo] * fato_vendas_detalhes[quantidade]
)
```

Observação: usar com ressalva, pois depende da interpretação do campo `valor_custo`.

### 10. Margem Bruta Estimada %

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Margem Bruta Estimada % =
DIVIDE([Margem Bruta Estimada], [Valor Real de Venda])
```

Observação: exibir como percentual e indicar que é uma margem estimada.

### 11. Faturamento por Produto

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Faturamento por Produto =
[Valor Real de Venda]
```

Observação: usar com a dimensão `dim_produto[produto]`.

### 12. Faturamento por Categoria

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Faturamento por Categoria =
[Valor Real de Venda]
```

Observação: usar com a dimensão `dim_produto[categoria]`.

### 13. Faturamento por Vendedor

Tabela recomendada: `fato_vendas`

```DAX
Faturamento por Vendedor =
[Faturamento Total]
```

Observação: usar com a dimensão `dim_vendedor[nome]`.

### 14. Faturamento por Estado

Tabela recomendada: `fato_vendas_detalhes`

```DAX
Faturamento por Estado =
[Valor Real de Venda]
```

Observação: usar com a dimensão `dim_cliente[estado]`.

### 15. Faturamento por Forma de Pagamento

Tabela recomendada: `fato_vendas`

```DAX
Faturamento por Forma de Pagamento =
[Faturamento Total]
```

Observação: usar com a dimensão `dim_forma_pagamento[descricao]`.

## Resumo das medidas

| Medida | Tabela recomendada | Granularidade | Observação |
| --- | --- | --- | --- |
| Faturamento Total | `fato_vendas` | Nota Fiscal | Indicador executivo principal |
| Número de Vendas | `fato_vendas` | Nota Fiscal | Usar `DISTINCTCOUNT` |
| Ticket Médio | `fato_vendas` | Nota Fiscal | Faturamento dividido por vendas |
| Clientes Ativos | `fato_vendas` | Cliente | Usar `DISTINCTCOUNT` |
| Quantidade Vendida | `fato_vendas_detalhes` | Item | Volume vendido |
| Valor Real de Venda | `fato_vendas_detalhes` | Item | Apoio para produto e categoria |
| Preço Médio Praticado | `fato_vendas_detalhes` | Item | Média ponderada |
| Margem Bruta Estimada | `fato_vendas_detalhes` | Item | Usar com ressalva |
| Faturamento por Produto | `fato_vendas_detalhes` | Produto | Usa produto como eixo |
| Faturamento por Categoria | `fato_vendas_detalhes` | Categoria | Usa categoria como eixo |
| Faturamento por Vendedor | `fato_vendas` | Vendedor | Usa vendedor como eixo |
| Faturamento por Estado | `fato_vendas_detalhes` | Estado | Usa estado como eixo |
| Faturamento por Forma de Pagamento | `fato_vendas` | Forma de Pagamento | Usa forma de pagamento como eixo |

## Cuidados

- Não misturar medidas de `fato_vendas` e `fato_vendas_detalhes` no mesmo cálculo sem regra clara.
- Usar `DIVIDE()` para evitar erro em divisões por zero.
- Manter as medidas em uma área centralizada no Power BI.
- Evitar criar cálculos diretamente nos visuais.
- Identificar margem como estimada na apresentação final.

## Conclusão

As medidas listadas são suficientes para construir o dashboard acadêmico proposto. Elas cobrem faturamento, vendas, clientes, produtos, categorias, vendedores, região, forma de pagamento e margem estimada.
