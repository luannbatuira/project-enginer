# project-enginer
Solução de Analytics Engineering desenvolvida como case técnico para a posição de Analytics Engineer Jr. na Zé Delivery.

## 🗂️ Estrutura do Repositório

```
ze_delivery_analytics/
│
├── queries/
│   ├── raw/
│   │   ├── ddl_raw_tables.sql        # CREATE TABLE das tabelas brutas
│   │   └── load_raw_bq.sh            # Script de carga via bq load (gcloud CLI)
│   │
│   ├── staging/
│   │   └── ddl_staging_views.sql     # Views de limpeza e regras de negócio
│   │
│   └── trusted/
│       └── ddl_trusted_views.sql     # Views analíticas prontas para consumo
│
├── insights/
│   └── business_questions.md         # Respostas às 5 questões de negócio
│
└── README.md
```

---

## 🏗️ Arquitetura

```
CSVs (fonte)
    │
    ▼
raw_*          → Tabelas brutas, espelho fiel dos arquivos originais
    │
    ▼
stg_*          → Views de staging: limpeza, regras de negócio, DQ
    │
    ▼
trusted_*      → Views analíticas: grãos consolidados prontos para BI
```

Todas as camadas vivem no mesmo dataset do BigQuery:

```
analytics-project-499319.ze_delivery_project
```

---

## 📐 Modelo de Dados

```
raw_customers ──< raw_orders ──< raw_order_items >── raw_products
                      │
                      ├──< raw_payments
                      └──< raw_refunds
```

| Tabela | Descrição | Linhas |
|---|---|---|
| `raw_customers` | Cadastro de clientes | 12 |
| `raw_products` | Catálogo de produtos | 8 |
| `raw_orders` | Pedidos realizados | 14 |
| `raw_order_items` | Itens por pedido | 20 |
| `raw_payments` | Pagamentos por pedido | 17 |
| `raw_refunds` | Reembolsos por pedido | 4 |

---

## ✅ Definição de Pedido Válido

> **Pedido válido** = `status <> 'canceled'` **E** possui ao menos um pagamento com `payment_status = 'captured'`

Essa regra é aplicada na view `stg_valid_orders` e herdada por todas as camadas superiores.

---

## 📊 Views da Camada Staging

| View | Descrição |
|---|---|
| `stg_dim_products` | Produtos com categoria normalizada (`NULL` → `'Sem Categoria'`) |
| `stg_payments_captured` | Pagamentos `captured` agregados por pedido |
| `stg_valid_orders` | Pedidos válidos conforme regra de negócio |
| `stg_order_gross_revenue` | Receita bruta por pedido válido |
| `stg_order_refunds` | Total de reembolsos por pedido |
| `stg_q1_net_revenue_by_month` | Q1: Receita líquida por mês |
| `stg_q2_new_customers_by_month` | Q2: Novos clientes por mês |
| `stg_q3_gross_revenue_by_category` | Q3: Receita bruta por categoria |
| `stg_q4_avg_ticket` | Q4: Ticket médio de pedidos válidos |
| `stg_dq_duplicate_payments` | DQ: Pagamentos possivelmente duplicados |
| `stg_dq_mixed_payment_status` | DQ: Pedidos com múltiplos status de pagamento |
| `stg_dq_products_no_category` | DQ: Produtos sem categoria |
| `stg_dq_price_divergence` | DQ: Divergência entre unit_price e list_price |
| `stg_dq_canceled_with_capture` | DQ: Pedidos cancelados com pagamento captured |
| `stg_dq_refunds_on_canceled` | DQ: Reembolsos em pedidos cancelados |

---

## 📈 Views da Camada Trusted

| View | Granularidade | Uso |
|---|---|---|
| `trusted_daily_revenue` | 1 linha por dia | Gráficos diários, detecção de anomalias |
| `trusted_monthly_revenue` | 1 linha por mês | Relatórios executivos, variação MoM |
| `trusted_customer_orders_summary` | 1 linha por cliente | Segmentação RFM, churn, CRM |
| `trusted_fact_orders` | 1 linha por pedido válido | Tabela base para qualquer slice analítico |

---

## 🚀 Como Reproduzir

### Pré-requisitos
- Conta no [Google Cloud](https://console.cloud.google.com/) com projeto criado
- [gcloud CLI](https://cloud.google.com/sdk/docs/install) instalado e autenticado
- Dataset criado no BigQuery:
```bash
bq mk --location=US analytics-project-499319:ze_delivery_project
```

### 1. Criar as tabelas RAW
Execute `queries/raw/ddl_raw_tables.sql` no BigQuery Studio.

### 2. Carregar os dados
```bash
cd queries/raw
bash load_raw_bq.sh
```

### 3. Criar as views Staging e Trusted
Execute `queries/staging/ddl_staging_views.sql` e `queries/trusted/ddl_trusted_views.sql` no BigQuery Studio — **um bloco `CREATE VIEW` por vez**.

---

## 🔍 Questões de Negócio

As respostas detalhadas com queries e resultados estão em [`business_questions.md`](business_questions.md).

---

## 🛠️ Stack

- **BigQuery** (Google Cloud) — armazenamento e processamento
- **GoogleSQL** — linguagem de queries
- **gcloud CLI / bq CLI** — carga de dados
- **Git / GitHub** — versionamento
