# Objetivos analiticos e KPIs do projeto

Data da analise: 2026-06-28

## 1. Objetivo Geral do Projeto

O Data Warehouse restaurado resolve o problema de transformar dados transacionais de vendas em uma base analitica confiavel para acompanhamento comercial, financeiro e executivo.

A estrutura permite consolidar vendas por periodo, cliente, localidade, vendedor, produto, categoria e forma de pagamento, reduzindo a dependencia de consultas diretamente sobre o modelo transacional. O objetivo de negocio e apoiar decisoes sobre crescimento de faturamento, mix de produtos, desempenho da equipe comercial, comportamento de clientes, sazonalidade, margem e priorizacao de acoes comerciais.

O modelo atual e especialmente indicado para responder perguntas como:

- quanto a empresa vendeu;
- quando vendeu mais ou menos;
- quais produtos e categorias sustentam o faturamento;
- quais vendedores geram maior resultado;
- quais clientes e regioes concentram receita;
- quais formas de pagamento sao mais utilizadas;
- onde existem oportunidades de ganho de margem, volume e ticket medio.

## 2. Objetivos Especificos

- Analisar faturamento total e sua evolucao ao longo do tempo.
- Acompanhar crescimento ou queda das vendas por ano, mes, dia e dia da semana.
- Identificar produtos mais vendidos por valor, quantidade e frequencia.
- Identificar categorias mais relevantes para faturamento e volume.
- Analisar desempenho dos vendedores por faturamento, quantidade, clientes atendidos e ticket medio.
- Analisar comportamento dos clientes por valor comprado, frequencia, localidade e mix de produtos.
- Identificar clientes de maior valor para a empresa.
- Analisar vendas por estado e cidade.
- Identificar concentracao geografica do faturamento.
- Analisar formas de pagamento mais utilizadas e seu peso no faturamento.
- Identificar sazonalidade de vendas por periodo.
- Comparar periodos equivalentes, como ano contra ano e mes contra mes.
- Acompanhar ticket medio por nota fiscal.
- Acompanhar quantidade vendida por produto, categoria, cliente e vendedor.
- Analisar lucro bruto estimado e margem estimada.
- Analisar diferenca entre valor de venda referencial e valor efetivo de venda.
- Avaliar desconto ou variacao comercial concedida quando houver semantica confirmada dos campos de valor.
- Medir participacao percentual de categorias, produtos, vendedores e clientes.
- Identificar dependencia de poucos produtos, clientes ou vendedores.
- Apoiar decisao sobre foco comercial por categoria e localidade.
- Apoiar planejamento de metas de vendas por periodo e vendedor.
- Apoiar storytelling executivo para apresentacao dos resultados em Power BI.
- Orientar a criacao de Data Marts por vendas, clientes, produtos, vendedores e tempo.

## 3. Perguntas de Negocio

| # | Pergunta de negocio | Objetivo da analise | Fato utilizada | Dimensoes utilizadas | Metricas necessarias | Granularidade |
| ---: | --- | --- | --- | --- | --- | --- |
| 1 | Qual foi o faturamento total da empresa? | Medir o resultado comercial consolidado. | `fato_vendas` ou `fato_vendas_detalhes` | Tempo, cliente, vendedor | `sum(valor)` ou `sum(valor_venda_real)` | Nota fiscal ou item, sem misturar graos |
| 2 | Como o faturamento evoluiu por ano e mes? | Acompanhar tendencia e crescimento. | `fato_vendas` | Tempo | `sum(valor)`, variacao percentual | Mes |
| 3 | Quais meses tiveram maior faturamento? | Identificar picos de vendas e sazonalidade. | `fato_vendas` | Tempo | `sum(valor)`, ranking | Mes |
| 4 | Quais dias da semana concentram mais vendas? | Identificar comportamento semanal de compra. | `fato_vendas_detalhes` | Tempo | `sum(valor_venda_real)`, `sum(quantidade)` | Dia da semana |
| 5 | Qual e o ticket medio por nota fiscal? | Medir valor medio das vendas. | `fato_vendas` | Tempo, cliente, vendedor, forma de pagamento | `sum(valor) / count(distinct numero_nf)` | Nota fiscal |
| 6 | Quantas vendas foram realizadas por periodo? | Medir volume de transacoes comerciais. | `fato_vendas` | Tempo | `count(distinct numero_nf)` | Nota fiscal |
| 7 | Qual e a quantidade total de itens vendidos? | Medir volume fisico vendido. | `fato_vendas_detalhes` | Tempo, produto, categoria | `sum(quantidade)` | Item da nota |
| 8 | Quais categorias geram maior faturamento? | Priorizar categorias mais relevantes. | `fato_vendas_detalhes` | Produto | `sum(valor_venda_real)`, participacao percentual | Categoria |
| 9 | Quais categorias geram maior volume vendido? | Entender categorias de maior giro. | `fato_vendas_detalhes` | Produto | `sum(quantidade)` | Categoria |
| 10 | Quais produtos sao mais vendidos por faturamento? | Identificar produtos de maior receita. | `fato_vendas_detalhes` | Produto | `sum(valor_venda_real)`, ranking | Produto |
| 11 | Quais produtos sao mais vendidos por quantidade? | Identificar produtos de maior volume. | `fato_vendas_detalhes` | Produto | `sum(quantidade)`, ranking | Produto |
| 12 | Quais produtos possuem maior preco medio praticado? | Avaliar posicionamento de preco. | `fato_vendas_detalhes` | Produto | `avg(valor_unitario)` ou `sum(valor_venda_real) / sum(quantidade)` | Produto |
| 13 | Qual e o faturamento por vendedor? | Medir desempenho individual da equipe comercial. | `fato_vendas` ou `fato_vendas_detalhes` | Vendedor, tempo | `sum(valor)` ou `sum(valor_venda_real)` | Vendedor |
| 14 | Quais vendedores possuem maior ticket medio? | Avaliar qualidade media das vendas. | `fato_vendas` | Vendedor | `sum(valor) / count(distinct numero_nf)` | Vendedor e nota fiscal |
| 15 | Quantos clientes cada vendedor atende? | Avaliar carteira ativa por vendedor. | `fato_vendas` | Vendedor, cliente | `count(distinct id_cliente)` | Vendedor |
| 16 | Quais vendedores vendem mais itens? | Medir produtividade por volume. | `fato_vendas_detalhes` | Vendedor, produto | `sum(quantidade)` | Vendedor e item |
| 17 | Quais clientes geram maior faturamento? | Identificar clientes estrategicos. | `fato_vendas` ou `fato_vendas_detalhes` | Cliente | `sum(valor)` ou `sum(valor_venda_real)`, ranking | Cliente |
| 18 | Quais clientes compram com maior frequencia? | Identificar recorrencia de compra. | `fato_vendas` | Cliente, tempo | `count(distinct numero_nf)` | Cliente e nota fiscal |
| 19 | Qual e o ticket medio por cliente? | Entender valor medio das compras por cliente. | `fato_vendas` | Cliente | `sum(valor) / count(distinct numero_nf)` | Cliente e nota fiscal |
| 20 | Quais estados concentram maior faturamento? | Avaliar desempenho geografico. | `fato_vendas_detalhes` | Cliente | `sum(valor_venda_real)` | Estado |
| 21 | Quais cidades concentram maior quantidade vendida? | Identificar polos de demanda. | `fato_vendas_detalhes` | Cliente | `sum(quantidade)` | Cidade |
| 22 | Quais categorias sao mais fortes em cada estado? | Cruzar mix de produtos com geografia. | `fato_vendas_detalhes` | Produto, cliente | `sum(valor_venda_real)`, `sum(quantidade)` | Categoria por estado |
| 23 | Qual forma de pagamento gera maior faturamento? | Entender distribuicao financeira das vendas. | `fato_vendas` | Forma de pagamento, tempo | `sum(valor)`, participacao percentual | Forma de pagamento |
| 24 | Qual forma de pagamento possui maior ticket medio? | Comparar perfil de compra por pagamento. | `fato_vendas` | Forma de pagamento | `sum(valor) / count(distinct numero_nf)` | Forma de pagamento e nota |
| 25 | Qual e a margem bruta estimada por categoria? | Avaliar rentabilidade por linha de produto. | `fato_vendas_detalhes` | Produto | `sum(valor_venda_real - valor_custo * quantidade)` | Categoria e item |
| 26 | Qual e a margem percentual estimada por produto? | Identificar produtos mais rentaveis. | `fato_vendas_detalhes` | Produto | `sum(valor_venda_real - valor_custo * quantidade) / sum(valor_venda_real)` | Produto |
| 27 | Onde ha maior diferenca entre valor referencial e valor real de venda? | Avaliar desconto, ajuste comercial ou variacao de preco. | `fato_vendas_detalhes` | Produto, vendedor, cliente, tempo | `sum(valor_venda - valor_venda_real)`, percentual sobre `valor_venda` | Item |
| 28 | Qual e a participacao de cada categoria no faturamento total? | Medir concentracao de receita por categoria. | `fato_vendas_detalhes` | Produto | `sum(valor_venda_real) / total geral` | Categoria |
| 29 | Quais clientes compram maior diversidade de produtos? | Identificar clientes com maior amplitude de relacionamento. | `fato_vendas_detalhes` | Cliente, produto | `count(distinct id_produto)`, `count(distinct categoria)` | Cliente |
| 30 | Quais periodos combinam alto faturamento e alta margem estimada? | Encontrar janelas comerciais mais rentaveis. | `fato_vendas_detalhes` | Tempo | `sum(valor_venda_real)`, margem estimada, margem percentual | Mes |

## 4. KPIs

| KPI | Definicao | Formula recomendada | Objetivo | Interpretacao |
| --- | --- | --- | --- | --- |
| Faturamento total | Valor total vendido no periodo. | `sum(fato_vendas.valor)` ou `sum(fato_vendas_detalhes.valor_venda_real)` | Medir receita bruta de vendas. | Quanto maior, melhor o desempenho comercial; respeitar o grao escolhido. |
| Lucro bruto estimado | Diferenca entre valor efetivo de venda e custo estimado. | `sum(valor_venda_real - valor_custo * quantidade)` | Avaliar rentabilidade antes de despesas operacionais. | Valor positivo indica geracao de margem; depende da confirmacao semantica de `valor_custo`. |
| Margem bruta estimada | Percentual do faturamento que sobra apos custo estimado. | `sum(valor_venda_real - valor_custo * quantidade) / sum(valor_venda_real)` | Comparar rentabilidade entre categorias, produtos e periodos. | Percentuais maiores indicam melhor eficiencia de margem. |
| Ticket medio | Valor medio por nota fiscal. | `sum(valor) / count(distinct numero_nf)` | Medir tamanho medio da compra. | Aumento pode indicar vendas maiores, mix mais caro ou clientes com maior gasto. |
| Quantidade vendida | Total de unidades vendidas. | `sum(quantidade)` | Medir volume fisico comercializado. | Ajuda a distinguir crescimento por preco de crescimento por volume. |
| Numero de clientes ativos | Clientes que compraram no periodo. | `count(distinct id_cliente)` | Medir alcance comercial. | Crescimento indica expansao ou ativacao da base compradora. |
| Numero de vendas | Total de notas fiscais emitidas. | `count(distinct numero_nf)` em `fato_vendas` | Medir frequencia transacional. | Aumento indica maior volume de transacoes. |
| Itens por venda | Media de itens por nota fiscal. | `count(*) em detalhe / count(distinct numero_nf)` | Avaliar composicao media das compras. | Valores maiores indicam cestas mais amplas. |
| Faturamento por vendedor | Receita atribuida a cada vendedor. | `sum(valor)` por `dim_vendedor` | Comparar desempenho comercial individual. | Apoia metas, ranking e coaching comercial. |
| Ticket medio por vendedor | Valor medio das notas de cada vendedor. | `sum(valor) / count(distinct numero_nf)` por vendedor | Avaliar qualidade media das vendas por vendedor. | Vendedores com ticket alto podem atuar melhor em contas ou produtos de maior valor. |
| Clientes por vendedor | Quantidade de clientes atendidos por vendedor. | `count(distinct id_cliente)` por vendedor | Avaliar amplitude da carteira. | Ajuda a comparar cobertura comercial e dependencia de poucos clientes. |
| Faturamento por categoria | Receita por categoria de produto. | `sum(valor_venda_real)` por `categoria` | Identificar linhas de maior representatividade. | Categorias com maior participacao sustentam o resultado. |
| Participacao da categoria | Peso de cada categoria no faturamento. | `sum(valor_venda_real da categoria) / sum(valor_venda_real total)` | Medir concentracao do mix. | Alta concentracao exige atencao a risco de dependencia. |
| Faturamento por produto | Receita por produto. | `sum(valor_venda_real)` por produto | Identificar produtos lideres. | Produtos lideres devem orientar estoque, campanhas e foco comercial. |
| Preco medio praticado | Valor medio unitario efetivamente vendido. | `sum(valor_venda_real) / sum(quantidade)` | Medir preco medio real por produto ou categoria. | Variacoes podem indicar mudanca de mix, descontos ou estrategia comercial. |
| Desconto ou variacao media estimada | Diferenca entre valor referencial e valor real de venda. | `sum(valor_venda - valor_venda_real) / sum(valor_venda)` | Avaliar concessoes comerciais, se `valor_venda` for confirmado como referencia. | Percentuais altos podem indicar descontos relevantes ou diferenca de semantica nos campos. |
| Faturamento por periodo | Receita por dia, mes ou ano. | `sum(valor)` por atributos de `dim_tempo` | Acompanhar tendencia temporal. | Permite comparar periodos e identificar sazonalidade. |
| Faturamento por estado | Receita por UF do cliente. | `sum(valor_venda_real)` por `estado` | Mapear concentracao geografica. | Estados fortes podem ser priorizados; estados fracos podem indicar oportunidade. |
| Faturamento por forma de pagamento | Receita por meio de pagamento. | `sum(valor)` por `dim_forma_pagamento` | Entender comportamento financeiro de recebimento. | Mostra preferencias de pagamento e peso financeiro por modalidade. |

Observacao importante: indicadores de produto, categoria, quantidade, custo e margem devem usar `fato_vendas_detalhes`. Indicadores de forma de pagamento e ticket medio por nota devem usar `fato_vendas`. Ao combinar produto com forma de pagamento, a juncao deve ser feita com controle por `numero_nf`, sem somar valores de cabecalho apos expandir para itens.

## 5. Hierarquias Analiticas

### Tempo

```text
Ano
  > Semestre
  > Trimestre
  > Mes
  > Dia
```

No modelo atual, `ano`, `mes`, `dia`, `data`, `dia_da_semana` e `mes_extenso` ja existem em `dim_tempo`. Semestre e trimestre podem ser derivados a partir do mes na camada de BI ou em Data Mart futuro.

### Produto

```text
Categoria
  > Produto
```

Essa hierarquia permite analisar participacao das categorias e fazer drill-down ate os produtos que compoem cada linha.

### Cliente e geografia

```text
Estado
  > Cidade
  > Cliente
```

Essa hierarquia permite entender concentracao geografica, relevancia de cidades e comportamento individual dos clientes.

### Vendedor

```text
Vendedor
```

O modelo atual possui apenas o nivel individual do vendedor. Em evolucoes futuras, seria possivel criar hierarquias por equipe, regional, supervisor ou canal comercial.

### Forma de pagamento

```text
Forma de pagamento
```

A dimensao permite analisar distribuicao de faturamento por modalidade, mas esta conectada diretamente apenas a `fato_vendas`.

### Venda

```text
Nota fiscal
  > Item da nota fiscal
```

Essa hierarquia conceitual e essencial para evitar duplicidade. A nota fiscal esta no grao de `fato_vendas`; o item esta no grao de `fato_vendas_detalhes`.

## 6. Possiveis Dashboards

### 6.1 Dashboard Executivo de Vendas

Objetivo: oferecer visao consolidada do resultado comercial.

Publico-alvo: diretoria, gerencia comercial e coordenacao executiva.

KPIs exibidos:

- faturamento total;
- numero de vendas;
- ticket medio;
- clientes ativos;
- quantidade vendida;
- margem bruta estimada.

Principais graficos:

- linha de faturamento por mes;
- barras de faturamento por categoria;
- ranking de vendedores;
- mapa ou barras por estado;
- cards executivos de KPIs;
- matriz de faturamento por ano e mes.

Filtros:

- ano;
- mes;
- estado;
- categoria;
- vendedor.

### 6.2 Dashboard de Produtos e Categorias

Objetivo: analisar mix de vendas, produtos lideres e categorias mais relevantes.

Publico-alvo: gerente comercial, compras, planejamento e produto.

KPIs exibidos:

- faturamento por categoria;
- quantidade vendida;
- participacao da categoria;
- preco medio praticado;
- margem estimada por categoria;
- top produtos por faturamento e volume.

Principais graficos:

- barras de faturamento por categoria;
- ranking de produtos;
- matriz categoria x produto;
- dispersao de quantidade versus margem;
- curva de participacao acumulada por produto.

Filtros:

- periodo;
- categoria;
- produto;
- estado;
- vendedor.

### 6.3 Dashboard de Performance de Vendedores

Objetivo: medir produtividade e resultado da equipe comercial.

Publico-alvo: gerente comercial e lideres de vendas.

KPIs exibidos:

- faturamento por vendedor;
- ticket medio por vendedor;
- clientes atendidos;
- numero de vendas;
- quantidade vendida;
- margem estimada por vendedor.

Principais graficos:

- ranking de vendedores por faturamento;
- barras comparando ticket medio;
- matriz vendedor x categoria;
- tendencia mensal por vendedor;
- dispersao faturamento versus quantidade de clientes.

Filtros:

- periodo;
- vendedor;
- categoria;
- estado;
- cliente.

### 6.4 Dashboard de Clientes e Geografia

Objetivo: entender comportamento de clientes e concentracao regional.

Publico-alvo: diretoria comercial, marketing e relacionamento com cliente.

KPIs exibidos:

- clientes ativos;
- faturamento por cliente;
- ticket medio por cliente;
- frequencia de compra;
- faturamento por estado e cidade;
- diversidade de produtos comprados.

Principais graficos:

- ranking de clientes;
- barras por estado e cidade;
- matriz cliente x categoria;
- linha de recorrencia por periodo;
- segmentacao de clientes por valor comprado.

Filtros:

- periodo;
- estado;
- cidade;
- cliente;
- categoria.

### 6.5 Dashboard de Sazonalidade e Tempo

Objetivo: identificar padroes temporais, picos e quedas de vendas.

Publico-alvo: diretoria, planejamento, marketing e operacoes.

KPIs exibidos:

- faturamento por periodo;
- quantidade vendida;
- numero de vendas;
- ticket medio;
- variacao mes contra mes;
- variacao ano contra ano.

Principais graficos:

- serie temporal mensal;
- heatmap de vendas por mes e dia da semana;
- comparativo anual;
- barras por dia da semana;
- cards de maior e menor mes.

Filtros:

- ano;
- mes;
- dia da semana;
- categoria;
- vendedor.

### 6.6 Dashboard Financeiro de Margem e Ticket

Objetivo: avaliar rentabilidade estimada e qualidade economica das vendas.

Publico-alvo: diretoria, financeiro e gerencia comercial.

KPIs exibidos:

- lucro bruto estimado;
- margem bruta estimada;
- ticket medio;
- preco medio praticado;
- desconto ou variacao media estimada;
- margem por categoria e produto.

Principais graficos:

- margem por categoria;
- ranking de produtos por margem;
- comparativo faturamento versus margem;
- dispersao preco medio versus quantidade;
- tendencia de ticket medio.

Filtros:

- periodo;
- categoria;
- produto;
- vendedor;
- estado.

### 6.7 Dashboard de Formas de Pagamento

Objetivo: analisar distribuicao das vendas por modalidade de pagamento.

Publico-alvo: financeiro, diretoria e gestao comercial.

KPIs exibidos:

- faturamento por forma de pagamento;
- numero de vendas por forma de pagamento;
- ticket medio por forma de pagamento;
- participacao percentual por modalidade.

Principais graficos:

- barras de faturamento por forma de pagamento;
- rosca ou barra 100% de participacao;
- linha temporal por modalidade;
- ranking de ticket medio por forma de pagamento.

Filtros:

- periodo;
- forma de pagamento;
- vendedor;
- cliente;
- estado.

## 7. Roadmap Analitico

1. Dashboard Executivo de Vendas
   - Prioridade maxima, pois consolida os principais indicadores do projeto e atende diretoria e gestao.

2. Dashboard de Produtos e Categorias
   - Prioridade alta, pois o modelo possui forte detalhamento de itens, produtos e categorias.

3. Dashboard de Performance de Vendedores
   - Prioridade alta, pois apoia metas, comparacao de desempenho e gestao comercial.

4. Dashboard de Clientes e Geografia
   - Prioridade alta, pois permite entender concentracao de receita, clientes estrategicos e regioes relevantes.

5. Dashboard de Sazonalidade e Tempo
   - Prioridade media-alta, pois fortalece planejamento comercial e comparacao de periodos.

6. Dashboard Financeiro de Margem e Ticket
   - Prioridade media, pois depende de validacao semantica dos campos de custo e valor referencial antes de virar indicador oficial.

7. Dashboard de Formas de Pagamento
   - Prioridade media, pois e importante para leitura financeira, mas a dimensao esta conectada diretamente apenas ao grao de nota fiscal.

## Conclusoes Analiticas

O modelo restaurado e suficiente para iniciar uma frente robusta de BI comercial. A base permite construir dashboards executivos de vendas, analises de produto, desempenho de vendedores, comportamento de clientes, distribuicao geografica, sazonalidade e formas de pagamento.

A tabela mais rica para analises de mix, produto, categoria, quantidade e margem e `data_warehouse.fato_vendas_detalhes`. A tabela mais segura para ticket medio, quantidade de vendas e forma de pagamento e `data_warehouse.fato_vendas`.

O principal cuidado analitico e respeitar a granularidade das fatos. Medidas de nota fiscal nao devem ser somadas apos expansao para itens. Com esse controle, a base oferece um caminho claro para criacao dos proximos Data Marts, metricas oficiais e dashboards em Power BI.
