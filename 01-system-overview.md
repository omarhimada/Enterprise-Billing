# System Overview

## Purpose

The enterprise billing system manages the full quote-to-cash billing lifecycle for customers with recurring, usage-based, one-time, and hybrid commercial models.

## Goals

- Support complex enterprise pricing and contract structures.
- Generate accurate invoices from subscriptions, usage, charges, credits, taxes, and discounts.
- Integrate with CRM, ERP, payment processors, tax providers, and data warehouses.
- Maintain auditable financial records.
- Support retries, reversals, adjustments, and dispute workflows.

## Non-Goals

- Acting as the system of record for CRM opportunities.
- Replacing the ERP or general ledger.
- Performing final statutory tax filing.

## Major Bounded Contexts

| Context | Responsibility |
|---|---|
| Customer Management | Customers, accounts, contacts, billing profiles |
| Catalog and Pricing | Products, plans, rate cards, price rules |
| Subscription Management | Subscriptions, contracts, entitlements |
| Usage Metering | Usage ingestion, validation, aggregation |
| Rating | Converts usage and charges into billable amounts |
| Invoicing | Invoice generation, tax, credits, adjustments |
| Payments | Collection, refunds, payment methods |
| Accounting | Revenue schedules, GL export, reconciliation |
| Notifications | Invoice delivery, dunning, payment reminders |
| Audit and Compliance | Immutable event history and controls |

## Key Design Principles

- Event-driven where state changes matter.
- Idempotent external and internal commands.
- Immutable financial ledger entries.
- Clear separation between operational state and accounting state.
- Versioned pricing, contracts, tax decisions, and invoice documents.
