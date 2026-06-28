# Inventario do banco restaurado

Data da analise: 2026-06-28

## Objetivo

Compreender a estrutura completa da base corporativa restaurada no Supabase antes de iniciar qualquer desenvolvimento adicional.

Esta etapa foi exclusivamente analitica. Nenhuma estrutura do banco foi alterada.

## Escopo analisado

Schemas restaurados/analisados:

- `geral`
- `vendas`
- `stage`
- `data_warehouse`
- `public`

Resumo geral:

| Tipo de objeto | Quantidade |
| --- | ---: |
| Schemas analisados | 5 |
| Tabelas | 26 |
| Views | 6 |
| Sequencias | 15 |
| Primary keys | 26 |
| Foreign keys | 25 |
| Unique constraints | 2 |

## Visao por schema

| Schema | Papel na base | Tabelas | Views | Sequencias |
| --- | --- | ---: | ---: | ---: |
| `geral` | Cadastro corporativo: pessoas, contatos e localizacao | 10 | 2 | 9 |
| `vendas` | Modelo transacional de vendas: notas, itens, produtos e categorias | 5 | 2 | 5 |
| `stage` | Area intermediaria/apoio com recortes de carga | 4 | 0 | 0 |
| `data_warehouse` | Modelo dimensional analitico | 7 | 1 | 1 |
| `public` | Exposicao publica/consumo, com view analitica | 0 | 1 | 0 |

## Contagem de registros por tabela

| Schema | Tabela | Registros |
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

## Schema `geral`

O schema `geral` representa o cadastro corporativo basico: pessoas, pessoas fisicas, pessoas juridicas, contatos, enderecos e hierarquia geografica.

### Tabelas

#### `geral.estado`

- Registros: 27
- Papel: cadastro de estados.
- Colunas:
  - `id integer not null default nextval('geral.estado_id_seq')`
  - `sigla text not null`
  - `descricao text not null`
- Primary key:
  - `estado_pkey`: `PRIMARY KEY (id)`

#### `geral.cidade`

- Registros: 5.568
- Papel: cadastro de cidades associadas a estados.
- Colunas:
  - `id integer not null default nextval('geral.cidade_id_seq')`
  - `id_estado integer not null`
  - `descricao text not null`
- Primary key:
  - `cidade_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_cidade__id_estado`: `id_estado -> geral.estado(id)` validada

#### `geral.bairro`

- Registros: 42.158
- Papel: cadastro de bairros associados a cidades.
- Colunas:
  - `id integer not null default nextval('geral.bairro_id_seq')`
  - `id_cidade integer not null`
  - `descricao text not null`
- Primary key:
  - `bairro_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_bairro__id_cidade`: `id_cidade -> geral.cidade(id)` validada

#### `geral.pessoa`

- Registros: 15.962
- Papel: entidade base para clientes, vendedores, fornecedores e outras pessoas.
- Colunas:
  - `id integer not null default nextval('geral.pessoa_id_seq')`
  - `data_cadastro timestamp with time zone not null`
- Primary key:
  - `pessoa_pkey`: `PRIMARY KEY (id)`

#### `geral.pessoa_fisica`

- Registros: 14.155
- Papel: especializacao de pessoa para individuos.
- Colunas:
  - `id integer not null default nextval('geral.pessoa_fisica_id_seq')`
  - `nome text not null`
  - `cpf text not null`
  - `nascimento timestamp with time zone`
- Primary key:
  - `pessoa_fisica_pkey`: `PRIMARY KEY (id)`
- Unique constraints:
  - `pessoa_fisica_cpf_key`: `UNIQUE (cpf)`
- Foreign keys:
  - `fk_pessoa_fisica__id`: `id -> geral.pessoa(id)` validada

#### `geral.pessoa_juridica`

- Registros: 1.807
- Papel: especializacao de pessoa para empresas.
- Colunas:
  - `id integer not null default nextval('geral.pessoa_juridica_id_seq')`
  - `razao_social text not null`
  - `cnpj text not null`
- Primary key:
  - `pessoa_juridica_pkey`: `PRIMARY KEY (id)`
- Unique constraints:
  - `pessoa_juridica_cnpj_key`: `UNIQUE (cnpj)`
- Foreign keys:
  - `fk_pessoa_juridica__id`: `id -> geral.pessoa(id)` validada

#### `geral.responsavel_juridico`

- Registros: 928
- Papel: relacao entre uma pessoa juridica e seu responsavel pessoa fisica.
- Colunas:
  - `id_pessoa_fisica integer not null`
  - `id_pessoa_juridica integer not null`
- Primary key:
  - `responsavel_juridico_pkey`: `PRIMARY KEY (id_pessoa_fisica, id_pessoa_juridica)`
- Foreign keys:
  - `fk_responsavel_juridico__id_pessoa_fisica`: `id_pessoa_fisica -> geral.pessoa(id)` validada
  - `fk_responsavel_juridico__id_pessoa_juridica`: `id_pessoa_juridica -> geral.pessoa(id)` validada

#### `geral.endereco`

- Registros: 15.962
- Papel: endereco de pessoas.
- Colunas:
  - `id integer not null default nextval('geral.endereco_id_seq')`
  - `id_pessoa integer not null`
  - `id_bairro integer not null`
  - `rua text not null`
  - `numero text`
  - `cep text`
  - `complemento text`
- Primary key:
  - `endereco_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_endereco__id_bairro`: `id_bairro -> geral.bairro(id)` validada
  - `endereco_id_pessoa_fkey`: `id_pessoa -> geral.pessoa(id)` nao validada (`NOT VALID`)

#### `geral.tipo_contato`

- Registros: 5
- Papel: dominio dos tipos de contato.
- Colunas:
  - `id integer not null default nextval('geral.tipo_contato_id_seq')`
  - `descricao text not null`
  - `sigla text`
- Primary key:
  - `tipo_contato_pkey`: `PRIMARY KEY (id)`

#### `geral.contato`

- Registros: 28.304
- Papel: contatos das pessoas cadastradas.
- Colunas:
  - `id integer not null default nextval('geral.contato_id_seq')`
  - `id_tipo_contato integer not null`
  - `id_pessoa integer not null`
  - `valor text not null`
  - `principal boolean not null`
- Primary key:
  - `contato_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_contato__id_pessoa`: `id_pessoa -> geral.pessoa(id)` validada
  - `fk_contato__id_tipo_contato`: `id_tipo_contato -> geral.tipo_contato(id)` validada

### Views

#### `geral.vi_lista_pessoa_fisica`

- Papel: lista pessoas fisicas com endereco geografico.
- Origem: `pessoa_fisica`, `endereco`, `bairro`, `cidade`, `estado`.
- Colunas principais: `id`, `cpf`, `nome`, `bairro`, `cidade`, `sigla`, `estado`.

#### `geral.vi_lista_pessoas`

- Papel: lista unificada de pessoas fisicas e juridicas.
- Origem: uniao entre `pessoa_fisica` e `pessoa_juridica`, ambas enriquecidas com endereco.
- Colunas principais: `id`, `cpf`/`cnpj`, `nome`/`razao_social`, `bairro`, `cidade`, `sigla`, `estado`, `tipo`.

### Sequencias

| Sequencia | Tabela/coluna dona | Ultimo valor | Observacao |
| --- | --- | ---: | --- |
| `bairro_id_seq` | `geral.bairro.id` | 42.158 | Incremento 1 |
| `cidade_id_seq` | `geral.cidade.id` | 5.568 | Incremento 1 |
| `contato_id_seq` | `geral.contato.id` | 28.304 | Incremento 1 |
| `endereco_id_seq` | `geral.endereco.id` | 15.962 | Incremento 1 |
| `estado_id_seq` | `geral.estado.id` | 27 | Incremento 1 |
| `pessoa_fisica_id_seq` | `geral.pessoa_fisica.id` | nulo | Existe, mas a PK usa ids herdados de `pessoa` |
| `pessoa_id_seq` | sem ownership formal | 15.963 | Usada como default de `geral.pessoa.id` |
| `pessoa_juridica_id_seq` | `geral.pessoa_juridica.id` | nulo | Existe, mas a PK usa ids herdados de `pessoa` |
| `tipo_contato_id_seq` | `geral.tipo_contato.id` | 5 | Incremento 1 |

## Schema `vendas`

O schema `vendas` representa o modelo transacional original de vendas. A granularidade transacional aparece em duas camadas: nota fiscal e item da nota fiscal.

### Tabelas

#### `vendas.categoria`

- Registros: 9
- Papel: dominio de categorias de produtos.
- Colunas:
  - `id integer not null default nextval('vendas.categoria_id_seq')`
  - `descricao text not null`
- Primary key:
  - `categoria_pkey`: `PRIMARY KEY (id)`

#### `vendas.forma_pagamento`

- Registros: 9
- Papel: dominio das formas de pagamento.
- Colunas:
  - `id integer not null default nextval('vendas.forma_pagamento_id_seq')`
  - `descricao text not null`
- Primary key:
  - `forma_pagamento_pkey`: `PRIMARY KEY (id)`

#### `vendas.produto`

- Registros: 233
- Papel: cadastro de produtos vendidos.
- Colunas:
  - `id integer not null default nextval('vendas.produto_id_seq')`
  - `id_fornecedor integer not null`
  - `id_categoria integer not null`
  - `nome text not null`
  - `valor_venda numeric(18,2) not null`
  - `valor_custo numeric(18,2) not null`
- Primary key:
  - `produto_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `produto_id_categoria_fkey`: `id_categoria -> vendas.categoria(id)` nao validada (`NOT VALID`)
  - `produto_id_fornecedor_fkey`: `id_fornecedor -> geral.pessoa(id)` nao validada (`NOT VALID`)

#### `vendas.nota_fiscal`

- Registros: 25.391
- Papel: cabecalho da venda/nota fiscal.
- Colunas:
  - `id integer not null default nextval('vendas.nota_fiscal_id_seq')`
  - `id_vendedor integer not null`
  - `id_cliente integer not null`
  - `id_forma_pagto integer not null`
  - `data_venda timestamp with time zone not null`
  - `numero_nf text`
  - `valor numeric(18,2)`
- Primary key:
  - `nota_fiscal_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `nota_fiscal_id_cliente_fkey`: `id_cliente -> geral.pessoa(id)` nao validada (`NOT VALID`)
  - `nota_fiscal_id_forma_pagto_fkey`: `id_forma_pagto -> vendas.forma_pagamento(id)` nao validada (`NOT VALID`)
  - `nota_fiscal_id_vendedor_fkey`: `id_vendedor -> geral.pessoa(id)` nao validada (`NOT VALID`)

#### `vendas.item_nota_fiscal`

- Registros: 35.752
- Papel: item da nota fiscal; representa os produtos de cada venda.
- Colunas:
  - `id integer not null default nextval('vendas.item_nota_fiscal_id_seq')`
  - `id_produto integer not null`
  - `id_nota_fiscal integer not null`
  - `quantidade integer not null`
  - `valor_venda_real numeric(18,2) not null`
  - `valor_unitario numeric(18,2)`
- Primary key:
  - `item_nota_fiscal_pkey`: `PRIMARY KEY (id)`
- Foreign keys:
  - `item_nota_fiscal_id_nota_fiscal_fkey`: `id_nota_fiscal -> vendas.nota_fiscal(id)` nao validada (`NOT VALID`)
  - `item_nota_fiscal_id_produto_fkey`: `id_produto -> vendas.produto(id)` nao validada (`NOT VALID`)

### Views

#### `vendas.vi_lista_cliente`

- Papel: ranking/listagem de clientes por valor vendido.
- Origem: `nota_fiscal`, `pessoa`, `pessoa_fisica`, `pessoa_juridica`.
- Observacao: agrega o valor da nota fiscal por cliente e converte para tipo `money`.

#### `vendas.vi_relatorio_gestao`

- Papel: relatorio transacional de gestao por cliente, vendedor, categoria e UF.
- Origem: `nota_fiscal`, `item_nota_fiscal`, `produto`, `categoria`, `pessoa_fisica`, `pessoa_juridica`, `endereco`, `bairro`, `cidade`, `estado`.
- Medida agregada: `sum(valor_venda_real)`.

### Sequencias

| Sequencia | Tabela/coluna dona | Ultimo valor |
| --- | --- | ---: |
| `categoria_id_seq` | `vendas.categoria.id` | 9 |
| `forma_pagamento_id_seq` | `vendas.forma_pagamento.id` | 9 |
| `item_nota_fiscal_id_seq` | `vendas.item_nota_fiscal.id` | 35.752 |
| `nota_fiscal_id_seq` | `vendas.nota_fiscal.id` | 25.391 |
| `produto_id_seq` | `vendas.produto.id` | 699 |

## Schema `stage`

O schema `stage` contem recortes intermediarios, aparentemente usados para apoio/carga do DW.

### Tabelas

#### `stage.forma_pagamento`

- Registros: 9
- Colunas:
  - `id integer not null`
  - `descricao text not null`
- Primary key:
  - `forma_pagamento_pkey`: `PRIMARY KEY (id)`

#### `stage.nota_fiscal`

- Registros: 25.391
- Colunas:
  - `id integer not null`
  - `id_vendedor integer not null`
  - `id_cliente integer not null`
  - `id_forma_pagto integer not null`
  - `data_venda timestamp with time zone not null`
  - `numero_nf text`
  - `valor numeric(18,2)`
- Primary key:
  - `nota_fiscal_pkey`: `PRIMARY KEY (id)`

#### `stage.pessoa_fisica`

- Registros: 14.155
- Colunas:
  - `id integer not null`
  - `nome text not null`
- Primary key:
  - `pessoa_fisica_pkey`: `PRIMARY KEY (id)`

#### `stage.pessoa_juridica`

- Registros: 1.807
- Colunas:
  - `id integer not null`
  - `razao_social text not null`
- Primary key:
  - `pessoa_juridica_pkey`: `PRIMARY KEY (id)`

### Views e sequencias

Nao ha views nem sequencias no schema `stage`.

## Schema `data_warehouse`

O schema `data_warehouse` e a camada analitica da base. Ele ja contem um modelo dimensional com dimensoes, fatos e uma view pronta para consumo de Data Mart.

### Tabelas de dimensao

#### `data_warehouse.dim_cliente`

- Registros: 1.000
- Tipo: dimensao.
- Chave: `id`.
- Colunas:
  - `id integer not null`
  - `nome text`
  - `cidade text`
  - `estado text`
- Primary key:
  - `pk_dim_cliente`: `PRIMARY KEY (id)`
- Cardinalidades observadas:
  - Clientes distintos: 1.000
  - Cidades distintas: 126
  - Estados distintos: 17
- Papel analitico: permite analisar vendas por cliente e localidade.

#### `data_warehouse.dim_produto`

- Registros: 233
- Tipo: dimensao.
- Chave: `id`.
- Colunas:
  - `id integer not null`
  - `produto text`
  - `categoria text`
- Primary key:
  - `pk_dim_produto`: `PRIMARY KEY (id)`
- Cardinalidades observadas:
  - Produtos distintos por chave: 233
  - Nomes de produto distintos: 227
  - Categorias distintas: 9
- Papel analitico: permite analisar vendas por produto e categoria.

#### `data_warehouse.dim_vendedor`

- Registros: 24
- Tipo: dimensao.
- Chave: `id`.
- Colunas:
  - `id integer not null`
  - `nome text`
- Primary key:
  - `pk_dim_vendedor`: `PRIMARY KEY (id)`
- Cardinalidades observadas:
  - Vendedores distintos por chave: 24
  - Nomes distintos: 23
- Papel analitico: permite analisar desempenho por vendedor.

#### `data_warehouse.dim_forma_pagamento`

- Registros: 9
- Tipo: dimensao.
- Chave: `id`.
- Colunas:
  - `id integer not null`
  - `descricao text`
- Primary key:
  - `pk_dim_forma_pagamento`: `PRIMARY KEY (id)`
- Cardinalidades observadas:
  - Formas de pagamento: 9
- Papel analitico: permite analisar vendas por forma de pagamento.

#### `data_warehouse.dim_tempo`

- Registros: 12.785
- Tipo: dimensao calendario.
- Chave: `id`.
- Colunas:
  - `data date`
  - `ano integer`
  - `mes integer`
  - `dia integer`
  - `dia_da_semana text`
  - `mes_extenso text`
  - `id integer not null default nextval('data_warehouse.dim_tempo_id_seq')`
- Primary key:
  - `dim_tempo_pkey`: `PRIMARY KEY (id)`
- Periodo total da dimensao:
  - Primeira data: 1994-08-22
  - Ultima data: 2029-08-22
  - Anos distintos: 36
- Periodo com vendas nas fatos:
  - Primeira venda: 2015-01-01
  - Ultima venda: 2017-12-27
- Papel analitico: permite analise por data, ano, mes, dia e atributos textuais de calendario.

### Tabelas fato

#### `data_warehouse.fato_vendas`

- Registros: 25.391
- Tipo: fato de venda no nivel de cabecalho/nota fiscal.
- Granularidade: uma linha por nota fiscal/venda.
- Evidencias da granularidade:
  - `id` distintos: 25.391
  - `numero_nf` distintos: 25.391
  - Registros totais: 25.391
- Colunas:
  - `id integer not null`
  - `id_cliente integer`
  - `id_vendedor integer`
  - `id_forma_pagamento integer`
  - `id_data_venda integer`
  - `numero_nf text`
  - `valor numeric(18,2)`
- Primary key:
  - `pk_fato_vendas`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_fato_vendas_dim_cliente`: `id_cliente -> data_warehouse.dim_cliente(id)` validada
  - `fk_fato_vendas_dim_vendedor`: `id_vendedor -> data_warehouse.dim_vendedor(id)` validada
  - `fk_fato_vendas_dim_forma_pagamento`: `id_forma_pagamento -> data_warehouse.dim_forma_pagamento(id)` validada
  - `fato_vendas_id_data_venda_fkey`: `id_data_venda -> data_warehouse.dim_tempo(id)` validada
- Medidas:
  - `valor`: valor total da nota/venda.
- Indicadores observados:
  - Clientes distintos: 1.000
  - Vendedores distintos: 24
  - Formas de pagamento distintas usadas: 5
  - Datas distintas usadas: 972
  - Valor total: 88.562.548,00
  - Valor medio por nota: 3.487,95
- Uso recomendado:
  - Analises por nota fiscal, faturamento total, ticket medio, vendas por cliente, vendedor, forma de pagamento e tempo.

#### `data_warehouse.fato_vendas_detalhes`

- Registros: 35.752
- Tipo: fato detalhada de itens de venda.
- Granularidade: uma linha por item/produto vendido dentro de uma nota fiscal.
- Evidencias da granularidade:
  - `id` distintos: 35.752
  - `numero_nf` distintos: 25.391
  - Ha mais linhas de detalhe do que notas, indicando multiplos itens por nota.
- Colunas:
  - `id integer not null`
  - `id_produto integer`
  - `id_cliente integer`
  - `id_vendedor integer`
  - `id_tempo integer`
  - `quantidade integer`
  - `valor_venda_real numeric(18,2)`
  - `valor_unitario numeric(18,2)`
  - `valor_venda numeric(18,2)`
  - `valor_custo numeric(18,2)`
  - `valor_nota_fiscal numeric(18,2)`
  - `numero_nf text`
- Primary key:
  - `pk_fato_vendas_detalhes`: `PRIMARY KEY (id)`
- Foreign keys:
  - `fk_fato_vendas_detalhes_dim_produto`: `id_produto -> data_warehouse.dim_produto(id)` validada
  - `fk_fato_vendas_detalhes_dim_cliente`: `id_cliente -> data_warehouse.dim_cliente(id)` validada
  - `fk_fato_vendas_detalhes_dim_vendedor`: `id_vendedor -> data_warehouse.dim_vendedor(id)` validada
  - `fk_fato_vendas_detalhes_dim_tempo`: `id_tempo -> data_warehouse.dim_tempo(id)` validada
- Medidas:
  - `quantidade`: unidades vendidas.
  - `valor_venda_real`: valor efetivo da venda do item.
  - `valor_unitario`: preco unitario do item.
  - `valor_venda`: valor de venda cadastrado/referencial.
  - `valor_custo`: custo unitario ou custo referencial do produto.
  - `valor_nota_fiscal`: valor total da nota fiscal repetido no nivel de item.
- Indicadores observados:
  - Produtos distintos usados: 226
  - Clientes distintos: 1.000
  - Vendedores distintos: 24
  - Datas distintas usadas: 972
  - Notas fiscais distintas: 25.391
  - Quantidade total: 89.527
  - Soma de `valor_venda_real`: 88.562.548,00
  - Soma de `valor_venda`: 35.520.358,00
  - Soma de `valor_custo`: 26.243.769,00
  - Soma de `valor_nota_fiscal`: 173.555.131,00
  - Margem bruta estimada (`valor_venda_real - valor_custo * quantidade`): 23.070.617,00
- Observacao importante:
  - `valor_nota_fiscal` esta no nivel da nota, mas aparece repetido em linhas de item. Em analises por item, nao deve ser somado diretamente sem deduplicar por `numero_nf`.
- Uso recomendado:
  - Analises por produto, categoria, quantidade, margem estimada, desempenho de vendedor, cliente e tempo no nivel de item.

### View analitica

#### `data_warehouse.view_dt_mart_vendas`

- Registros retornados: 35.752
- Papel: view de Data Mart de vendas no nivel de item.
- Origem principal: `data_warehouse.fato_vendas_detalhes`.
- Enriquecimentos:
  - `dim_produto`: produto e categoria.
  - `dim_cliente`: nome, cidade e estado.
  - `dim_vendedor`: nome do vendedor.
  - `dim_tempo`: data, ano, dia, dia da semana e mes.
- A mesma definicao tambem existe em `public.view_dt_mart_vendas`.
- Medidas expostas:
  - `quantidade`
  - `valor_venda_real`
  - `valor_unitario`
  - `valor_venda`
  - `valor_custo`
  - `valor_nota_fiscal`
- Atributos expostos:
  - Produto/categoria
  - Cliente/cidade/estado
  - Vendedor
  - Data/ano/dia/dia da semana/mes
- Indicadores da view publica equivalente:
  - Linhas: 35.752
  - Notas distintas: 25.391
  - Produtos distintos: 226
  - Clientes distintos: 1.000
  - Vendedores distintos: 24
  - Primeira data: 2015-01-01
  - Ultima data: 2017-12-27
  - Quantidade total: 89.527
  - Valor total: 88.562.548,00

### Sequencia

| Sequencia | Tabela/coluna dona | Ultimo valor |
| --- | --- | ---: |
| `dim_tempo_id_seq` | `data_warehouse.dim_tempo.id` | 12.785 |

### Relacionamentos do modelo dimensional

`fato_vendas`:

| Fato | FK | Dimensao | Cardinalidade logica |
| --- | --- | --- | --- |
| `fato_vendas` | `id_cliente` | `dim_cliente(id)` | N:1 |
| `fato_vendas` | `id_vendedor` | `dim_vendedor(id)` | N:1 |
| `fato_vendas` | `id_forma_pagamento` | `dim_forma_pagamento(id)` | N:1 |
| `fato_vendas` | `id_data_venda` | `dim_tempo(id)` | N:1 |

`fato_vendas_detalhes`:

| Fato | FK | Dimensao | Cardinalidade logica |
| --- | --- | --- | --- |
| `fato_vendas_detalhes` | `id_produto` | `dim_produto(id)` | N:1 |
| `fato_vendas_detalhes` | `id_cliente` | `dim_cliente(id)` | N:1 |
| `fato_vendas_detalhes` | `id_vendedor` | `dim_vendedor(id)` | N:1 |
| `fato_vendas_detalhes` | `id_tempo` | `dim_tempo(id)` | N:1 |

### Analise da modelagem `data_warehouse`

O schema `data_warehouse` ja esta modelado em formato dimensional, proximo de um star schema. Existem duas tabelas fato, mas elas representam granularidades diferentes:

- `fato_vendas`: cabecalho da venda, uma linha por nota fiscal.
- `fato_vendas_detalhes`: detalhe da venda, uma linha por item/produto da nota fiscal.

A tabela mais apropriada para Data Marts de produto/categoria e `fato_vendas_detalhes`, pois contem `id_produto`, quantidade, custo e valores no nivel de item. A tabela `fato_vendas` e mais adequada para indicadores de cabecalho, como ticket medio por nota, total por forma de pagamento e quantidade de notas.

As dimensoes disponiveis cobrem os principais eixos analiticos:

- Tempo: `dim_tempo`.
- Cliente/localidade: `dim_cliente`.
- Produto/categoria: `dim_produto`.
- Vendedor: `dim_vendedor`.
- Forma de pagamento: `dim_forma_pagamento`, presente apenas em `fato_vendas`.

Ha uma diferenca importante entre as duas fatos: `fato_vendas_detalhes` nao possui `id_forma_pagamento`, enquanto `fato_vendas` nao possui `id_produto`. Assim, qualquer analise que combine produto e forma de pagamento exigira juntar as duas fatos por `numero_nf`, com cuidado para nao duplicar medidas de cabecalho.

### Medidas recomendadas para desenvolvimento analitico

Medidas seguras no nivel de nota (`fato_vendas`):

- Faturamento por nota: `sum(valor)`.
- Quantidade de notas: `count(distinct numero_nf)` ou `count(*)`.
- Ticket medio: `sum(valor) / count(distinct numero_nf)`.
- Faturamento por forma de pagamento.

Medidas seguras no nivel de item (`fato_vendas_detalhes`):

- Faturamento por item: `sum(valor_venda_real)`.
- Quantidade vendida: `sum(quantidade)`.
- Valor medio por item: `avg(valor_venda_real)`.
- Preco unitario medio: `avg(valor_unitario)`.
- Custo referencial: `sum(valor_custo * quantidade)`, se `valor_custo` for unitario.
- Margem bruta estimada: `sum(valor_venda_real - valor_custo * quantidade)`.

Medidas que exigem cuidado:

- `sum(valor_nota_fiscal)` em `fato_vendas_detalhes`: nao deve ser usada diretamente porque o valor da nota se repete por item.
- Comparacoes entre `valor_venda_real`, `valor_venda`, `valor_unitario` e `valor_custo`: exigem definir semanticamente se `valor_venda` e preco cadastrado, preco praticado ou valor total de linha.

### Categorias observadas no DW

| Categoria | Itens | Quantidade | Valor venda real |
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

## Schema `public`

O schema `public` contem uma view analitica exposta:

### View `public.view_dt_mart_vendas`

- Registros retornados: 35.752
- Papel: exposicao da view de vendas para consumo externo, provavelmente Power BI ou Data API.
- Definicao: equivalente a `data_warehouse.view_dt_mart_vendas`.
- Origem:
  - `data_warehouse.fato_vendas_detalhes`
  - `data_warehouse.dim_produto`
  - `data_warehouse.dim_cliente`
  - `data_warehouse.dim_vendedor`
  - `data_warehouse.dim_tempo`
- Observacao de seguranca:
  - Como `public` pode ser exposto no Supabase, esta view deve ser revisada antes de uso em aplicacoes publicas ou Data API.

## Foreign keys nao validadas

O dump original continha algumas FKs como `NOT VALID`, e esse estado foi preservado no restore.

| Schema | Tabela | Constraint | Definicao |
| --- | --- | --- | --- |
| `geral` | `endereco` | `endereco_id_pessoa_fkey` | `id_pessoa -> geral.pessoa(id)` |
| `vendas` | `item_nota_fiscal` | `item_nota_fiscal_id_nota_fiscal_fkey` | `id_nota_fiscal -> vendas.nota_fiscal(id)` |
| `vendas` | `item_nota_fiscal` | `item_nota_fiscal_id_produto_fkey` | `id_produto -> vendas.produto(id)` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_cliente_fkey` | `id_cliente -> geral.pessoa(id)` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_forma_pagto_fkey` | `id_forma_pagto -> vendas.forma_pagamento(id)` |
| `vendas` | `nota_fiscal` | `nota_fiscal_id_vendedor_fkey` | `id_vendedor -> geral.pessoa(id)` |
| `vendas` | `produto` | `produto_id_categoria_fkey` | `id_categoria -> vendas.categoria(id)` |
| `vendas` | `produto` | `produto_id_fornecedor_fkey` | `id_fornecedor -> geral.pessoa(id)` |

Essas constraints existem, mas o PostgreSQL nao validou retrospectivamente todos os registros. Antes de qualquer etapa de hardening ou producao, pode ser interessante executar uma etapa planejada de validacao com `ALTER TABLE ... VALIDATE CONSTRAINT`, apos checar se nao ha violacoes.

## Conclusoes tecnicas

1. O banco restaurado possui uma separacao clara entre camada transacional (`geral` e `vendas`), camada intermediaria (`stage`) e camada analitica (`data_warehouse`).
2. O `data_warehouse` ja esta pronto para analises iniciais de vendas, principalmente usando `fato_vendas_detalhes` e a view `public.view_dt_mart_vendas`.
3. Existem duas granularidades de fato, nota fiscal e item de nota fiscal. Essa diferenca precisa guiar todos os calculos para evitar duplicidade.
4. A view `public.view_dt_mart_vendas` e conveniente para Power BI, mas deve ser revisada em termos de exposicao no Supabase.
5. A modelagem atual e suficiente para iniciar Data Marts de vendas por tempo, produto, categoria, cliente, localidade e vendedor.
6. Analises que exigirem forma de pagamento junto com produto precisam de modelagem cuidadosa, porque forma de pagamento esta na fato de cabecalho e produto esta na fato de detalhe.
