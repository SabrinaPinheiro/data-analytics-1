# Modelagem do Dashboard Power BI

Data da documentação: 2026-06-29

## Objetivo do dashboard

O dashboard final em Power BI tem como objetivo apresentar uma visão executiva das vendas, permitindo acompanhar faturamento, número de vendas, clientes, produtos, vendedores, categorias, região e forma de pagamento.

O relatório será utilizado como entrega prática da Unidade 01, com foco em modelo dimensional, Data Warehouse, métricas principais, visualização analítica e storytelling da apresentação final.

Esta etapa é exclusivamente documental. Nenhuma estrutura do banco foi alterada.

## Estrutura do relatório

O relatório será organizado em no máximo três páginas, mantendo uma navegação simples e adequada ao escopo acadêmico do projeto.

### Página 1 — Visão Geral Executiva

Objetivo: apresentar os principais indicadores de desempenho das vendas.

Elementos previstos:

- Faturamento Total
- Número de Vendas
- Ticket Médio
- Clientes Ativos
- Evolução mensal do faturamento
- Faturamento por categoria
- Faturamento por vendedor

### Página 2 — Produtos e Categorias

Objetivo: analisar o desempenho comercial dos produtos e das categorias.

Elementos previstos:

- Quantidade Vendida
- Faturamento por Produto
- Faturamento por Categoria
- Participação da Categoria
- Preço Médio Praticado
- Ranking de produtos

### Página 3 — Clientes, Região e Pagamento

Objetivo: analisar a distribuição das vendas por cliente, localização e forma de pagamento.

Elementos previstos:

- Faturamento por Estado
- Faturamento por Cidade
- Clientes Ativos
- Faturamento por Forma de Pagamento
- Ticket Médio por Forma de Pagamento
- Ranking de clientes, caso os dados permitam

## Filtros globais

Os filtros globais devem ser limitados aos campos essenciais para a navegação do relatório:

- Ano
- Mês
- Estado
- Vendedor
- Categoria
- Produto

## Medidas principais no Power BI

As principais medidas a serem criadas no Power BI são:

- Faturamento Total
- Número de Vendas
- Ticket Médio
- Clientes Ativos
- Quantidade Vendida
- Faturamento por Categoria
- Faturamento por Produto
- Faturamento por Vendedor
- Faturamento por Estado
- Faturamento por Forma de Pagamento
- Preço Médio Praticado
- Margem Bruta Estimada, se usada com ressalva

## Cuidados de modelagem

Para manter a consistência das análises, alguns cuidados devem ser observados:

- Métricas de nota fiscal devem usar `data_warehouse.fato_vendas`.
- Métricas de produto, categoria e quantidade devem usar `data_warehouse.fato_vendas_detalhes`.
- Não misturar granularidades sem cuidado, especialmente ao combinar indicadores de nota fiscal com indicadores de item vendido.
- Não usar indicadores estimados como valores financeiros oficiais sem ressalva.
- Medidas derivadas de margem, custo ou preço médio devem ser apresentadas como indicadores analíticos, respeitando as limitações documentadas no dicionário de métricas.

## Encaminhamento prático

Após este documento, o próximo passo do projeto será abrir o Power BI Desktop, conectar ao Supabase/PostgreSQL, importar as tabelas do schema `data_warehouse` e construir o dashboard final.

A documentação técnica detalhada já produzida permanece no repositório como apoio técnico e evidência do processo de análise. A entrega acadêmica, a partir deste ponto, será concentrada no dashboard Power BI e na apresentação final com storytelling.
