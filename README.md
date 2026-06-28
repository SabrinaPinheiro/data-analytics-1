# Data Warehouse Financeiro

## Trabalho Final – Unidade 01 | Banco de Dados

**Formação:** Data Analytics
**Instituição:** Digital College – Fortaleza/CE
**Professor:** Maurílio Oliveira
**Aluna:** Sabrina Pinheiro

---

# Resumo

Este projeto foi desenvolvido como requisito de avaliação da **Unidade 01 – Banco de Dados** da Formação **Data Analytics** da Digital College.

O trabalho consiste na construção de uma solução analítica baseada em conceitos de **Data Warehouse**, **Modelagem Dimensional**, **Data Marts** e **Business Intelligence**, utilizando uma base de dados corporativa fornecida pela instituição.

Todo o ambiente foi implementado utilizando **Supabase (PostgreSQL)** como banco de dados e **Power BI** para construção dos dashboards analíticos.

---

# Objetivos do Trabalho

Conforme especificação da disciplina, este projeto contempla as seguintes atividades:

* Criar e popular um ou mais modelos dimensionais com foco no esquema financeiro;
* Definir os objetivos analíticos do(s) modelo(s);
* Construir Data Marts para suportar as análises;
* Elaborar métricas, indicadores e gráficos;
* Desenvolver um Dashboard em Power BI;
* Apresentar os resultados utilizando técnicas de Data Storytelling.

---

# Arquitetura da Solução

```text
Base de Dados Corporativa
            │
            ▼
   Supabase (PostgreSQL)
            │
            ▼
     Data Warehouse
            │
            ▼
        Data Marts
            │
            ▼
       Power BI Desktop
            │
            ▼
 Dashboard e Storytelling
```

---

# Tecnologias Utilizadas

* Supabase (PostgreSQL)
* SQL
* Power BI Desktop
* Draw.io
* Git
* GitHub
* Markdown

---

# Estrutura do Projeto

```text
data-analytics-1/
│
├── 00_documentacao/
│   ├── 01_diagnostico_importacao.md
│   ├── 02_validacao_restore.md
│   ├── 03_inventario_banco.md
│   ├── 04_modelo_relacional.md
│   ├── 05_analise_modelo_dimensional.md
│   ├── 06_objetivos_analiticos.md
│   ├── 07_datamarts_metricas.md
│   ├── 08_powerbi_storytelling.md
│
├── 01_origem/
│   └── Base de dados original
│
├── 02_dw/
│   ├── Scripts de criação do Data Warehouse
│   ├── Dimensões
│   └── Tabela Fato
│
├── 03_datamarts/
│   └── Data Marts analíticos
│
├── 04_powerbi/
│   └── Dashboard (.pbix)
│
├── 05_apresentacao/
│   └── Apresentação Final
│
└── README.md
```

---

# Etapas do Projeto

* Importação e análise da base de dados.
* Inventário do banco de dados.
* Levantamento das entidades e relacionamentos.
* Definição dos objetivos analíticos.
* Modelagem Dimensional (Star Schema).
* Criação do Data Warehouse.
* Construção dos Data Marts.
* Desenvolvimento das consultas SQL.
* Definição de métricas e indicadores.
* Desenvolvimento do Dashboard em Power BI.
* Construção do Data Storytelling.
* Elaboração da apresentação final.

---

# Boas Práticas Aplicadas

* Organização dos scripts por etapa do projeto.
* Separação entre dados de origem, Data Warehouse e Data Marts.
* Utilização de nomenclatura padronizada para objetos do banco de dados.
* Documentação técnica do processo de desenvolvimento.
* Estrutura preparada para versionamento em Git.

---

# Observações

Embora desenvolvido para fins acadêmicos, este projeto foi estruturado seguindo práticas utilizadas em projetos de Engenharia de Dados, permitindo sua utilização como parte do portfólio técnico da autora.

---

# Licença

Projeto desenvolvido exclusivamente para fins acadêmicos como requisito de avaliação da Formação Data Analytics da Digital College.
