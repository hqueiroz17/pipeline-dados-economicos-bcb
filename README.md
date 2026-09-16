# Pipeline de Dados Econômicos (BCB)

Pipeline de engenharia de dados que consome a API pública do Banco Central do Brasil, trata e consolida indicadores econômicos (SELIC, IPCA e Dólar) em uma arquitetura em camadas (Bronze, Silver, Gold), usando Databricks, AWS S3, Python, Pandas e SQL.

## Arquitetura

```
API do Banco Central (BCB)
        ↓
   [Bronze] — dado bruto, direto da API, particionado por data de extração
        ↓
   [Silver] — tipos corrigidos, duplicatas removidas, validação de qualidade via SQL
        ↓
   [Gold] — agregação mensal e JOIN das 3 séries (via SQL/CTE), consolidadas em uma única tabela
```

Todas as camadas são armazenadas no **AWS S3**, particionadas por data de execução, permitindo histórico auditável de cada extração.

## Indicadores utilizados

| Indicador | Código SGS (BCB) | Frequência |
|---|---|---|
| SELIC | 11 | Diária |
| IPCA | 433 | Mensal |
| Dólar comercial | 1 | Diária |

Período extraído: 01/01/2024 a 13/09/2026.

## Estrutura do repositório

```
pipeline-dados-economicos-bcb/
├── README.md
├── extracao/
│   └── 01_extrai_selic.ipynb      # extração da API, salva no bronze
└── transformacao/
    ├── 02_silver.ipynb             # tratamento e validação de qualidade
    └── 03_gold.ipynb               # agregação mensal e JOIN das séries
```

## Tecnologias

- **Python** — extração via API (`requests`) e tratamento de dados
- **Pandas** — estruturação e tratamento dos dados no silver
- **SQL (Spark SQL)** — validação de qualidade no silver; agregação e JOIN (via CTE) no gold
- **AWS S3** — armazenamento das três camadas, particionado por data
- **AWS IAM** — controle de acesso e credenciais para o S3
- **Databricks** — ambiente de desenvolvimento e execução dos notebooks
- **Git/GitHub** — versionamento de todo o código

## Principais decisões técnicas

- **Pandas em vez de PySpark**: o volume de dados (~1.400 linhas no total) é pequeno o suficiente para processar em memória local, tornando o processamento distribuído do PySpark desnecessário para este escopo.
- **Parquet em vez de CSV** na camada silver: preserva os tipos de dado (datas e números) ao ser lido novamente, diferente do CSV, que armazena tudo como texto.
- **Particionamento por data**: cada execução gera uma nova pasta (`bronze/AAAA-MM-DD/`, etc.), preservando o histórico de cada extração para fins de auditoria.
- **Agregação mensal antes do JOIN**: como SELIC e Dólar são diários e o IPCA é mensal, as três séries precisaram ser padronizadas para a mesma granularidade (mês) antes de serem unidas.
- **Credenciais via widgets do Databricks**: as chaves da AWS nunca ficam gravadas no código-fonte, sendo inseridas manualmente a cada execução.

## Possíveis evoluções (fora do escopo atual)

- Orquestração dos notebooks via Databricks Jobs
- Carga da camada gold em um banco relacional (SQL Server)
- Uso de Databricks Secrets para gerenciamento de credenciais
- Data final de extração calculada dinamicamente (ex: D-1), em vez de fixa

## Autor

Hugo Queiroz