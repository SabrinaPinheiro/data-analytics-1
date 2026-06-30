# Dashboard Final

Data da documentação: 2026-06-30

## Objetivo

Documentar a estrutura final do dashboard em Power BI que será utilizado como entrega prática do projeto.

O dashboard terá foco em vendas e será organizado em três páginas simples, com linguagem visual clara e adequada para apresentação acadêmica.

Esta etapa é apenas documental. Nenhuma estrutura do banco foi alterada.

## Estrutura do dashboard

O relatório final será composto por três páginas:

1. Visão Geral Executiva
2. Produtos e Categorias
3. Clientes, Região e Pagamento

## Página 1 — Visão Geral Executiva

### Objetivo

Apresentar um resumo geral do desempenho das vendas.

### Principais KPIs

- Faturamento Total
- Número de Vendas
- Ticket Médio
- Clientes Ativos

### Visuais previstos

- Cartões com os principais KPIs.
- Gráfico de linha com evolução mensal do faturamento.
- Gráfico de barras com faturamento por categoria.
- Gráfico de barras com faturamento por vendedor.

### Storytelling esperado

Esta página deve responder rapidamente:

- quanto foi vendido;
- quantas vendas foram realizadas;
- qual foi o ticket médio;
- como o faturamento evoluiu ao longo do tempo;
- quais categorias e vendedores mais contribuíram para o resultado.

## Página 2 — Produtos e Categorias

### Objetivo

Analisar o desempenho dos produtos e categorias vendidos.

### Principais KPIs

- Quantidade Vendida
- Faturamento por Produto
- Faturamento por Categoria
- Preço Médio Praticado
- Margem Bruta Estimada

### Visuais previstos

- Ranking de produtos por faturamento.
- Gráfico de barras com faturamento por categoria.
- Gráfico de barras com quantidade vendida por categoria.
- Cartão com preço médio praticado.
- Cartão ou gráfico simples com margem bruta estimada.

### Storytelling esperado

Esta página deve mostrar quais produtos e categorias têm maior importância para o negócio, tanto em faturamento quanto em volume.

A margem bruta estimada pode ser apresentada, mas com ressalva, pois depende da interpretação do campo de custo.

## Página 3 — Clientes, Região e Pagamento

### Objetivo

Analisar a distribuição das vendas por cliente, localização e forma de pagamento.

### Principais KPIs

- Clientes Ativos
- Faturamento por Estado
- Faturamento por Cidade
- Faturamento por Forma de Pagamento
- Ticket Médio

### Visuais previstos

- Gráfico de barras com faturamento por estado.
- Gráfico de barras com faturamento por cidade.
- Gráfico de barras ou rosca com faturamento por forma de pagamento.
- Ranking de clientes por faturamento, se for adequado para a apresentação.
- Cartões com clientes ativos e ticket médio.

### Storytelling esperado

Esta página deve explicar onde as vendas estão concentradas, quais regiões têm maior participação e quais formas de pagamento são mais utilizadas.

## Filtros globais

Filtros recomendados para o dashboard:

- Ano
- Mês
- Estado
- Vendedor
- Categoria
- Produto

Esses filtros devem permitir navegação simples entre as páginas, sem deixar o relatório complexo demais.

## Principais KPIs do dashboard

Os KPIs principais do dashboard são:

- Faturamento Total
- Número de Vendas
- Ticket Médio
- Clientes Ativos
- Quantidade Vendida
- Preço Médio Praticado
- Margem Bruta Estimada
- Faturamento por Produto
- Faturamento por Categoria
- Faturamento por Vendedor
- Faturamento por Estado
- Faturamento por Forma de Pagamento

## Como o dashboard responde ao trabalho final

O dashboard atende aos objetivos do trabalho final porque:

- utiliza o Data Warehouse restaurado no Supabase;
- aplica conceitos de modelo dimensional;
- usa dimensões e fatos no Power BI;
- cria métricas analíticas em DAX;
- apresenta indicadores comerciais e financeiros;
- organiza a análise em páginas com foco executivo;
- permite contar uma história com base nos dados.

## Orientação para apresentação final

Na apresentação, a explicação pode seguir esta ordem:

1. Apresentar o objetivo do projeto.
2. Explicar brevemente a origem dos dados e o Data Warehouse.
3. Mostrar a estrutura do modelo no Power BI.
4. Apresentar a página executiva.
5. Destacar produtos e categorias mais relevantes.
6. Explicar região, clientes e formas de pagamento.
7. Finalizar com os principais aprendizados.

## Conclusão

O dashboard final será simples, objetivo e alinhado ao escopo acadêmico. A estrutura em três páginas permite apresentar os principais resultados de vendas sem excesso de complexidade, mantendo foco em análise, métricas e storytelling.
