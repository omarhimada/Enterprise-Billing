# Domain Model and Entity Relationships

## Core Entities

| Entity | Description |
|---|---|
| Customer | Legal or commercial customer entity. |
| Account | Billing account under a customer. |
| BillingProfile | Invoice delivery, currency, tax, and payment settings. |
| Product | Sellable product family. |
| Plan | Commercial plan for a product. |
| Price | Versioned pricing rule for a plan or charge. |
| Contract | Enterprise commercial agreement. |
| Subscription | Active recurring entitlement under an account or contract. |
| SubscriptionItem | Product, plan, quantity, and pricing component. |
| UsageRecord | Raw or normalized metered usage. |
| RatedUsage | Usage transformed into billable charges. |
| Invoice | Legal billing document. |
| InvoiceLine | Billable line item on an invoice. |
| TaxLine | Tax amount associated with invoice lines. |
| Payment | Payment attempt or captured payment. |
| CreditMemo | Credit applied against invoice balance. |
| LedgerEntry | Immutable accounting movement. |

## Mermaid Entity Relationship Diagram

```mermaid
erDiagram
    CUSTOMER ||--o{ ACCOUNT : owns
    ACCOUNT ||--|| BILLING_PROFILE : has
    CUSTOMER ||--o{ CONTRACT : signs
    CONTRACT ||--o{ SUBSCRIPTION : governs
    ACCOUNT ||--o{ SUBSCRIPTION : has

    PRODUCT ||--o{ PLAN : offers
    PLAN ||--o{ PRICE : defines
    SUBSCRIPTION ||--o{ SUBSCRIPTION_ITEM : contains
    PLAN ||--o{ SUBSCRIPTION_ITEM : selected_by
    PRICE ||--o{ SUBSCRIPTION_ITEM : priced_by

    SUBSCRIPTION_ITEM ||--o{ USAGE_RECORD : records
    USAGE_RECORD ||--o{ RATED_USAGE : rated_as
    RATED_USAGE ||--o{ INVOICE_LINE : billed_as

    ACCOUNT ||--o{ INVOICE : receives
    INVOICE ||--o{ INVOICE_LINE : contains
    INVOICE_LINE ||--o{ TAX_LINE : taxed_by
    INVOICE ||--o{ PAYMENT : paid_by
    INVOICE ||--o{ CREDIT_MEMO : credited_by
    INVOICE ||--o{ LEDGER_ENTRY : posts

    CUSTOMER {
        uuid customer_id PK
        string legal_name
        string status
        datetime created_at
    }

    ACCOUNT {
        uuid account_id PK
        uuid customer_id FK
        string account_number
        string status
        datetime created_at
    }

    BILLING_PROFILE {
        uuid billing_profile_id PK
        uuid account_id FK
        string currency
        string invoice_delivery_method
        string payment_terms
        string tax_exemption_status
    }

    CONTRACT {
        uuid contract_id PK
        uuid customer_id FK
        date effective_date
        date end_date
        string status
    }

    PRODUCT {
        uuid product_id PK
        string name
        string status
    }

    PLAN {
        uuid plan_id PK
        uuid product_id FK
        string billing_period
        string status
    }

    PRICE {
        uuid price_id PK
        uuid plan_id FK
        string pricing_model
        decimal unit_amount
        string currency
        datetime valid_from
        datetime valid_to
    }

    SUBSCRIPTION {
        uuid subscription_id PK
        uuid account_id FK
        uuid contract_id FK
        string status
        date start_date
        date renewal_date
    }

    SUBSCRIPTION_ITEM {
        uuid subscription_item_id PK
        uuid subscription_id FK
        uuid plan_id FK
        uuid price_id FK
        decimal quantity
    }

    USAGE_RECORD {
        uuid usage_record_id PK
        uuid subscription_item_id FK
        string metric_name
        decimal quantity
        datetime usage_start
        datetime usage_end
        string idempotency_key
    }

    RATED_USAGE {
        uuid rated_usage_id PK
        uuid usage_record_id FK
        decimal rated_quantity
        decimal amount
        string currency
    }

    INVOICE {
        uuid invoice_id PK
        uuid account_id FK
        string invoice_number
        date invoice_date
        date due_date
        decimal total_amount
        string status
    }

    INVOICE_LINE {
        uuid invoice_line_id PK
        uuid invoice_id FK
        uuid rated_usage_id FK
        string description
        decimal quantity
        decimal amount
    }

    TAX_LINE {
        uuid tax_line_id PK
        uuid invoice_line_id FK
        string jurisdiction
        decimal tax_rate
        decimal tax_amount
    }

    PAYMENT {
        uuid payment_id PK
        uuid invoice_id FK
        decimal amount
        string status
        string processor_reference
    }

    CREDIT_MEMO {
        uuid credit_memo_id PK
        uuid invoice_id FK
        decimal amount
        string reason_code
    }

    LEDGER_ENTRY {
        uuid ledger_entry_id PK
        uuid invoice_id FK
        string account_code
        string debit_credit
        decimal amount
        datetime posted_at
    }
```

## Aggregate Boundaries

| Aggregate | Root | Notes |
|---|---|---|
| Customer Aggregate | Customer | Owns customer profile and account references. |
| Billing Account Aggregate | Account | Owns billing profile, payment terms, invoice preferences. |
| Subscription Aggregate | Subscription | Owns subscription items and lifecycle state. |
| Invoice Aggregate | Invoice | Owns invoice lines, tax lines, invoice totals, status transitions. |
| Payment Aggregate | Payment | Owns payment attempt lifecycle and processor references. |
| Ledger Aggregate | LedgerEntry | Immutable accounting postings. |
