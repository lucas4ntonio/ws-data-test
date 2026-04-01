# Teste Técnico — Data / Analytics Engineer

## Contexto

Você vai trabalhar com dados de **estabelecimentos comerciais** provenientes de três fontes distintas:

| Fonte | Arquivo | Descrição |
|-------|---------|-----------|
| **RFB** (Receita Federal do Brasil) | `data/rfb.csv` | Cadastro oficial de CNPJs — razão social, nome fantasia, endereço, CNAE e situação cadastral |
| **Google** | `data/google.csv` | Dados extraídos do Google Places — nome, tipo de local, endereço, avaliações e coordenadas |
| **iFood** | `data/ifood.csv` | Dados extraídos da plataforma iFood — nome, categoria, endereço e coordenadas |

---

## O Desafio

Construa um **pipeline de dados** que realize o *record linkage* (linkagem de registros) entre as três fontes, identificando quais registros em fontes distintas representam o **mesmo estabelecimento físico**. Em seguida, consolide os dados em um modelo relacional.

---

## Requisitos

### 1. Orquestração

O pipeline deve ser orquestrado por uma ferramenta de sua escolha. Algumas sugestões:

- [Prefect](https://www.prefect.io/)
- [Apache Airflow](https://airflow.apache.org/)
- [Dagster](https://dagster.io/)
- [Mage](https://www.mage.ai/)

> **Justifique** brevemente sua escolha no README do seu repositório.

---

### 2. Etapas do Pipeline

O pipeline deve conter, no mínimo, as seguintes etapas:

1. **Ingestão** — carregamento dos arquivos CSV
2. **Normalização** — limpeza e padronização dos dados (nomes, endereços, CEP, etc.)
3. **Match RFB × Google** — identificação de correspondências entre registros RFB e Google
4. **Match RFB × iFood** — identificação de correspondências entre registros RFB e iFood
5. **Match Google × iFood** — identificação de correspondências entre registros Google e iFood
6. **Consolidação** — geração das tabelas de saída (veja abaixo)

> Os critérios e algoritmos de match são **livres**. Use sua criatividade e justifique suas escolhas.

---

### 3. Tecnologia

A tecnologia de transformação e o banco de dados de destino são **livres**. Exemplos:

- Python (pandas, SQLAlchemy, DuckDB, etc.)
- dbt para transformações SQL
- SQL puro com PostgreSQL, DuckDB, SQLite, etc.
- Qualquer combinação das anteriores

---

### 4. Modelo de Saída

O resultado final deve ser armazenado nas seguintes tabelas:

#### `companies` — tabela consolidada principal

Cada linha representa um **estabelecimento único**, independente de quantas fontes o identificaram.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | integer (PK) | Identificador único gerado |
| `name` | text | Nome consolidado do estabelecimento |
| `address_street_name` | text | Logradouro consolidado |
| `address_street_number` | text | Número |
| `address_district` | text | Bairro |
| `address_city` | text | Município |
| `address_state` | text | UF |
| `address_zipcode` | text | CEP (8 dígitos, sem máscara) |
| `latitude` | float | Latitude (preferencialmente do Google ou iFood) |
| `longitude` | float | Longitude |
| `has_rfb` | boolean | `true` se o estabelecimento foi associado a um CNPJ |
| `rfb_document_number` | text | CNPJ (quando disponível) |
| `rfb_situation` | text | Situação cadastral RFB (ATIVA, BAIXADA, etc.) |
| `rfb_cnae` | text | CNAE principal |
| `rfb_social_name` | text | Razão social |
| `rfb_trading_name` | text | Nome fantasia |
| `google_cid` | text | CID do Google (quando disponível) |
| `google_place_type` | text | Tipo de local no Google |
| `google_rating` | float | Avaliação média |
| `google_rating_count` | integer | Número de avaliações |
| `ifood_slug` | text | Slug do iFood (quando disponível) |
| `ifood_category` | text | Categoria no iFood |
| `source` | text | Fontes onde o estabelecimento foi identificado (ex: `rfb,google,ifood`) |

---

#### `rfb_matches` — relação entre registros RFB e empresas consolidadas

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `company_id` | integer (FK → companies.id) | |
| `rfb_document_number` | text | CNPJ |
| `match_policy` | text | Critério que gerou o match (ex: `name_exact`, `address+name_fuzzy`) |

#### `google_matches` — relação entre registros Google e empresas consolidadas

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `company_id` | integer (FK → companies.id) | |
| `google_cid` | text | CID do Google |
| `match_policy` | text | Critério que gerou o match |

#### `ifood_matches` — relação entre registros iFood e empresas consolidadas

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `company_id` | integer (FK → companies.id) | |
| `ifood_slug` | text | Slug do iFood |
| `match_policy` | text | Critério que gerou o match |

---

## O que será avaliado

- Critérios de Match
- Pipeline
- Consolidação
- Código e Documentação

---

## Entregáveis

1. **Repositório Git** (GitHub, GitLab ou similar) com:
   - Código-fonte do pipeline
   - `README.md` com instruções de instalação e execução
   - Justificativa das escolhas técnicas (orquestrador, tecnologia, critérios de match)
2. **Tabelas de saída** populadas — pode ser um arquivo SQLite exportado, um dump SQL, ou o banco em memória executado durante a apresentação.
3. Prompts utilizados com IA (caso IA tenha sido usada).

---

## Dados de Entrada

Os arquivos estão na pasta `data/`:

- `rfb.csv` — registros da Receita Federal
- `google.csv` — registros do Google Places
- `ifood.csv` — registros do iFood

> **Atenção:** os dados foram gerados para fins de teste. CNPJs, telefones e coordenadas são fictícios e não correspondem a estabelecimentos reais.

---

## Desejável (porém não obrigatório)

Os itens abaixo não são requisitos, mas demonstram maturidade de engenharia e serão considerados positivamente na avaliação:

### Qualidade de Código
- **Linter e formatador** configurados no projeto (ex: `ruff`/`flake8` + `black` para Python, `eslint`/`prettier` para JS/TS)
- **Type hints** nas funções principais (Python) ou tipagem estrita (TypeScript)
- **`pre-commit` hooks** garantindo que o código commitado passa no lint antes de entrar no repositório

### Testes
- **Testes unitários** para as funções de normalização de texto e endereço (ex: remoção de sufixos legais, limpeza de CEP)
- **Testes de match** com casos conhecidos — dado um par de registros, o resultado do match é o esperado?
- **Cobertura mínima** documentada no README (mesmo que parcial)

### Reprodutibilidade
- **`docker-compose`** ou instruções claras para subir o ambiente sem dependências globais instaladas na máquina
- **Gerenciamento de dependências** explícito (`requirements.txt`, `pyproject.toml`, `package.json`, etc.) com versões fixadas
- **Variáveis de ambiente** separadas do código (`.env.example` documentando o que é necessário)

### Observabilidade do Pipeline
- **Logs estruturados** nas etapas do pipeline (quantos registros foram lidos, quantos matches foram gerados por etapa)
- **Métricas de qualidade** ao final da execução: total de empresas consolidadas por fonte (`rfb_only`, `google_only`, `rfb+google+ifood`, etc.)
- **Tratamento de erros** explícito — o pipeline falha de forma clara e informativa quando algo inesperado ocorre

### Modelagem
- **Constraints e índices** definidos nas tabelas de saída (chaves estrangeiras, `UNIQUE` onde aplicável)
- **Schema versionado** via migrations (ex: Alembic, Flyway, ou arquivos `.sql` numerados)
