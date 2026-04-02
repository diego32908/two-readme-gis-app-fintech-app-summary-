# Synthetic Personal Finance Dataset (24-Month Demo Bundle)

## Overview

This bundle contains **24 months of synthetic personal-finance transaction data** plus helper files for building and testing consumer-finance dashboards.

The dataset is designed to support common personal-finance app features such as:
- transaction categorization
- essential vs. discretionary spending classification
- recurring charge detection
- subscription price increase tracking
- cash-flow summaries and charts
- hypothetical “what if this spending had been invested?” analyses

All records are synthetic. No real customer, account, or institution data is included.

## Dataset summary

- Date range: **2024-03-01 through 2026-02-28**
- Total transactions: **1,152**
- Currency: **USD**
- Geography fields included: **city, state**
- Main account types represented: **checking, credit_card, brokerage**
- Merchant descriptions include realistic formatting noise to support keyword-based parsing and normalization
- Recurring subscriptions and services include price changes over time so price-increase detection can be tested end-to-end
- Market data file includes synthetic month-end prices for multiple tickers and benchmarks

## Files

### `transactions_24mo_raw.csv`
Main upload file for application demos.

Contains a realistic bank-export-style schema without helper labels. Use this file when testing the app’s ingestion, parsing, categorization, and dashboard logic.

Columns:
- `transaction_id`
- `transaction_date`
- `posted_date`
- `account_name`
- `account_type`
- `merchant_name`
- `description`
- `amount`
- `currency`
- `channel`
- `city`
- `state`

### `transactions_24mo_labeled.csv`
Enhanced version of the main transaction file with validation and feature-engineering fields.

Use this file to test or benchmark:
- category assignment
- essential vs. discretionary classification
- recurring transaction grouping
- subscription detection
- price-change detection
- budget inclusion / exclusion rules
- opportunity-cost or investment overlays

Key added columns include:
- `month_key`
- `merchant_normalized`
- `amount_abs`
- `transaction_kind`
- `category_primary`
- `category_secondary`
- `essentiality_label`
- `recurring_flag`
- `subscription_flag`
- `recurring_group_id`
- `billing_frequency`
- `recurrence_sequence_num`
- `days_since_prior_recurring`
- `previous_recurring_amount`
- `price_change_flag`
- `is_subscription_price_change_increase`
- `price_delta`
- `price_change_pct`
- `merchant_stock_ticker`
- `default_benchmark_ticker`
- `investment_ticker`
- `investment_shares`
- `exclude_from_budget_flag`
- `budget_spend_flag`
- `income_expense_flag`
- `notes`

### `merchant_keyword_map.csv`
Starter rule base for transaction categorization and merchant normalization.

This file can be used to implement deterministic keyword matching before adding any fallback heuristics or AI-assisted labeling.

Helpful columns include:
- `keyword`
- `merchant_normalized`
- `category_primary`
- `category_secondary`
- `essentiality_label`
- `recurring_default`
- `subscription_default`
- `billing_frequency_default`
- `merchant_stock_ticker`
- `default_benchmark_ticker`
- `account_type_hint`
- `notes`

### `market_prices_monthly_24mo.csv`
Synthetic month-end market data for multiple tickers and benchmarks.

This file supports opportunity-cost features such as:
- “What would this discretionary spending be worth if invested instead?”
- benchmark comparison charts
- ticker-level summary cards

Columns:
- `date`
- `ticker`
- `open_price`
- `close_price`
- `monthly_return_pct`
- `volume`

### `subscription_price_events.csv`
Summary table of consumer subscription price increases.

Use this file to power subscription alert cards, price-change tables, or timeline views focused only on subscription products.

Columns:
- `event_date`
- `recurring_group_id`
- `merchant_name`
- `category_primary`
- `category_secondary`
- `billing_frequency`
- `essentiality_label`
- `merchant_stock_ticker`
- `old_charge`
- `new_charge`
- `increase_amount`
- `price_change_pct`
- `increase_direction`

### `recurring_service_price_events.csv`
Broader recurring-service price-change summary.

Includes both subscriptions and other recurring services such as utilities or communications. Use this file when the app needs a wider view of recurring-cost changes.

Columns match `subscription_price_events.csv`.

### `data_dictionary.csv`
Field-level reference for the main schema.

Contains:
- column name
- data type
- description
- where the field appears
- implementation notes

## Recommended implementation workflow

1. **Use `transactions_24mo_raw.csv` as the primary demo upload file.**
   - Parse and preview the uploaded transactions.
   - Normalize merchant text as needed.

2. **Use `merchant_keyword_map.csv` for the first-pass categorizer.**
   - Map merchants or keywords to categories.
   - Assign essential vs. discretionary labels.
   - Seed recurring / subscription defaults.

3. **Validate against `transactions_24mo_labeled.csv`.**
   - Compare predicted categories with reference labels.
   - Test recurring detection and price-change logic.

4. **Use the event summary files for UI features.**
   - `subscription_price_events.csv` for subscription-focused alerts
   - `recurring_service_price_events.csv` for broader recurring-cost summaries

5. **Use `market_prices_monthly_24mo.csv` for investment or opportunity-cost views.**
   - Map discretionary spending to a benchmark or ticker.
   - Compute hypothetical value over time using the synthetic close prices.

## Implementation notes and conventions

### Signed amounts
In the transaction files:
- **positive amounts = money in**
- **negative amounts = money out**

Examples:
- paycheck, transfer-in, refund = positive
- rent, subscriptions, card purchases = negative

### Absolute amount field
In the labeled file, `amount_abs` stores the absolute value of `amount`. This is useful for charts, grouping, and price-change calculations without reapplying `abs()` in the application.

### Date fields
- `transaction_date` = date the transaction occurred
- `posted_date` = date the transaction posted
- `month_key` = monthly aggregation key in `YYYY-MM` format

### Raw vs. labeled files
- The **raw** file is intended to simulate what a user uploads.
- The **labeled** file is intended for testing, benchmarking, and development.

Applications should not require labeled fields to function if they are meant to operate on raw uploaded data.

### Merchant normalization
Merchant names and descriptions intentionally include realistic variation so normalization logic can be tested. The `merchant_normalized` field in the labeled file provides a reference normalized value.

### Recurring transactions
Recurring transactions are linked using `recurring_group_id` and supported by:
- `billing_frequency`
- `recurrence_sequence_num`
- `days_since_prior_recurring`
- `previous_recurring_amount`

These fields make it possible to:
- detect subscriptions
- estimate monthly recurring burden
- identify price increases or decreases
- compare old vs. new charges

### Price-change conventions
In the labeled transactions file:
- `price_change_flag` indicates any recurring price change event
- `is_subscription_price_change_increase` isolates subscription increases specifically
- `price_delta` and `price_change_pct` quantify the change

In the two summary price-event files:
- `old_charge`, `new_charge`, and `increase_amount` are stored as **positive charge amounts**
- `increase_direction` is provided as a simple label for filtering or UI display

### Budget flags
The labeled file includes helper flags for budgeting views:
- `exclude_from_budget_flag`
- `budget_spend_flag`
- `income_expense_flag`

These can be used to exclude transfers or brokerage activity from spending summaries when appropriate.

### Investment-related fields
The labeled file includes optional fields for opportunity-cost and investing views:
- `merchant_stock_ticker`
- `default_benchmark_ticker`
- `investment_ticker`
- `investment_shares`

These fields support scenarios such as:
- linking discretionary merchants to public tickers
- comparing hypothetical investment outcomes against a benchmark
- displaying periods before and after brokerage activity begins

### Synthetic market data
`market_prices_monthly_24mo.csv` contains synthetic prices and returns intended for demo and application testing only. It should not be treated as historical market data.

## Suggested feature mapping

### 1. Essential vs. discretionary spending
Use:
- `essentiality_label`
- `category_primary`
- `category_secondary`
- `merchant_keyword_map.csv`

### 2. Subscription price increase tracker
Use:
- `subscription_flag`
- `recurring_group_id`
- `previous_recurring_amount`
- `price_change_flag`
- `price_delta`
- `price_change_pct`
- `subscription_price_events.csv`

### 3. Hypothetical investment / opportunity-cost view
Use:
- `essentiality_label`
- `merchant_stock_ticker`
- `default_benchmark_ticker`
- `market_prices_monthly_24mo.csv`

## Limitations

- This bundle is synthetic and simplified for software development and demonstrations.
- It is not intended to represent any real household, institution, or market history exactly.
- Merchant naming, billing cadence, and price changes are realistic enough for prototyping, but not guaranteed to reflect real-world vendor behavior.
- Market prices and returns are synthetic.

## Quick start

For most prototypes:
1. Upload `transactions_24mo_raw.csv`
2. Categorize with `merchant_keyword_map.csv`
3. Validate with `transactions_24mo_labeled.csv`
4. Add subscription alerts using `subscription_price_events.csv`
5. Add optional investment comparisons using `market_prices_monthly_24mo.csv`
