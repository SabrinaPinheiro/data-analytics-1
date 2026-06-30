# Modelo de Dados no Power BI

Data da documentação: 2026-06-30

## Objetivo

Documentar como o modelo de dados será organizado no Power BI a partir das tabelas importadas do schema `data_warehouse` do Supabase.

O objetivo é manter um modelo simples, em formato estrela, adequado ao trabalho acadêmico e suficiente para a construção do dashboard final.

Esta etapa é apenas documental. Nenhuma estrutura do banco foi alterada.

## Tabelas importadas

As tabelas importadas no Power BI são:

- `dim_cliente`
- `dim_produto`
- `dim_tempo`
- `dim_vendedor`
- `dim_forma_pagamento`
- `fato_vendas`
- `fato_vendas_detalhes`

A view `view_dt_mart_vendas` também pode ser utilizada como apoio para conferência, validação ou consultas analíticas simples.

## Papel das dimensões

As dimensões descrevem os principais eixos de análise:

- `dim_cliente`: cliente, cidade e estado.
- `dim_produto`: produto e categoria.
- `dim_tempo`: data, ano, mês, dia e dia da semana.
- `dim_vendedor`: vendedor responsável pela venda.
- `dim_forma_pagamento`: forma de pagamento utilizada na venda.

Essas tabelas devem ser usadas como filtros, segmentações e eixos dos gráficos.

## Papel das fatos

As tabelas fato armazenam os valores numéricos analisados no dashboard.

### `fato_vendas`

Representa a venda no nível da nota fiscal.

Uso principal:

- faturamento total;
- número de vendas;
- ticket médio;
- clientes ativos;
- faturamento por vendedor;
- faturamento por forma de pagamento.

### `fato_vendas_detalhes`

Representa a venda no nível do item da nota fiscal.

Uso principal:

- quantidade vendida;
- faturamento por produto;
- faturamento por categoria;
- preço médio praticado;
- margem bruta estimada;
- análise por cliente, produto, vendedor, tempo e estado.

## Diferença entre as fatos

A principal diferença está na granularidade:

- `fato_vendas`: uma linha por nota fiscal.
- `fato_vendas_detalhes`: uma linha por item vendido dentro da nota fiscal.

Por isso, as duas fatos não devem ser somadas juntas sem cuidado. O valor da nota pode se repetir quando a venda é aberta por item.

## Relacionamentos previstos

Relacionamentos recomendados no Power BI:

| Tabela dimensão | Chave | Tabela fato | Chave na fato | Cardinalidade |
| --- | --- | --- | --- | --- |
| `dim_cliente` | `id` | `fato_vendas` | `id_cliente` | 1:* |
| `dim_vendedor` | `id` | `fato_vendas` | `id_vendedor` | 1:* |
| `dim_forma_pagamento` | `id` | `fato_vendas` | `id_forma_pagamento` | 1:* |
| `dim_tempo` | `id` | `fato_vendas` | `id_data_venda` | 1:* |
| `dim_cliente` | `id` | `fato_vendas_detalhes` | `id_cliente` | 1:* |
| `dim_vendedor` | `id` | `fato_vendas_detalhes` | `id_vendedor` | 1:* |
| `dim_produto` | `id` | `fato_vendas_detalhes` | `id_produto` | 1:* |
| `dim_tempo` | `id` | `fato_vendas_detalhes` | `id_tempo` | 1:* |

## Direção de filtro recomendada

A direção de filtro recomendada é:

```text
Dimensão -> Fato
```

Ou seja, filtro em direção única, das dimensões para as tabelas fato.

Essa configuração evita ambiguidades e mantém o modelo mais simples para o Power BI.

## Cuidados com granularidade

- Medidas de nota fiscal devem usar `fato_vendas`.
- Medidas de produto, categoria e quantidade devem usar `fato_vendas_detalhes`.
- Não somar `valor_nota_fiscal` dentro da tabela de detalhes.
- Não usar `fato_vendas[valor]` em análises abertas por produto sem cuidado.
- Utilizar `DISTINCTCOUNT` para contar notas fiscais e clientes.

## Uso da view `view_dt_mart_vendas`

A view `view_dt_mart_vendas` pode ser usada como apoio para:

- conferir valores;
- validar consultas;
- visualizar rapidamente os dados de vendas;
- testar análises simples.

No entanto, ela não precisa substituir o modelo estrela. Para o dashboard final, a recomendação é priorizar as dimensões e fatos importadas separadamente, pois esse modelo é mais organizado e mais adequado para criação de medidas DAX.

## Conclusão

O modelo de dados no Power BI deve seguir uma estrutura estrela simples, com dimensões filtrando as tabelas fato. Essa organização atende ao objetivo acadêmico do projeto e permite construir os visuais do dashboard final com clareza e segurança.
