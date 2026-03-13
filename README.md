# Financial Ledger API

A financial ledger microservice implementing double-entry bookkeeping, ACID-safe transactions, balance validation, and transaction isolation.

--> This project is adapted and extended from an existing repository for learning and development purposes.
Modifications include restructuring documentation, improvements to validation logic, and additional explanations of the financial accounting workflow.

# Double-Entry Bookkeeping Implementation

The system follows the double-entry accounting principle, where every financial transaction produces two ledger entries:
- One debit
- One credit

This guarantees that:
- Total debits always equal total credits
- The ledger remains balanced
- Every transaction is traceable and auditable

Example: Transfer from Account A → Account B for ₹100
Entry	Account	Type	Amount
1	A	Debit	−100
2	B	Credit	+100

For each operation, the API creates
1. A transaction record in the transactions table
2. Corresponding ledger entries in the ledger_entries table

# Ensuring ACID Properties

Financial operations run inside a database transaction using MySQL and Knex.

-> Atomicity
All database updates either complete fully or roll back entirely if an error occurs.

-> Consistency
The double-entry model ensures that the ledger never becomes unbalanced.

-> Isolation
Concurrent transactions are isolated to avoid incorrect balance calculations.

-> Durability
Committed transactions are persisted by the MySQL InnoDB storage engine, ensuring data safety.

# Transaction Isolation Level

The system uses MySQL's REPEATABLE READ isolation level.

Why REPEATABLE READ?

This level prevents issues such as:
- Non-repeatable reads
- Phantom reads
- Race conditions during concurrent transfers

It ensures balance calculations remain consistent even when multiple transactions run simultaneously.

# Balance Calculation & Negative Balance Prevention

Account balances are determined using:
- The account's stored balance
- Ledger entry updates associated with the account

Before processing withdrawals or transfers:
1. The account balance is retrieved inside a transaction
2. The requested amount is validated
3. If insufficient funds are detected, the request is rejected

Example response:
{
  "error": "Insufficient funds"
}

This ensures:
- No negative balances
- No overdrafts
- Consistent ledger integrity

# System Architecture
                         ┌─────────────────────┐
                         │   Client / Postman  │
                         └─────────┬───────────┘
                                   │ HTTP
                                   ▼
                        ┌───────────────────────┐
                        │      Express API      │
                        ├───────────────────────┤
                        │ Routes                │
                        │ Controllers           │
                        │ Services              │
                        │ Validation            │
                        └───────────┬───────────┘
                                    │
                                    │ Knex Query Builder
                                    ▼
                        ┌─────────────────────────┐
                        │       MySQL Database     │
                        ├─────────────────────────┤
                        │ accounts                │
                        │ transactions            │
                        │ ledger_entries          │
                        └─────────────────────────┘
# Database Schema (ERD)
┌──────────────────────────┐
│        accounts          │
├──────────────────────────┤
│ id (PK)                  │
│ name                     │
│ balance                  │
│ currency                 │
│ created_at               │
└────────────┬─────────────┘
             │
             │ 1..n
             ▼
┌──────────────────────────┐
│       transactions        │
├──────────────────────────┤
│ id (PK)                  │
│ type                     │
│ amount                   │
│ currency                 │
│ description              │
│ created_at               │
└────────────┬─────────────┘
             │
             │ 1..n
             ▼
┌──────────────────────────┐
│      ledger_entries       │
├──────────────────────────┤
│ id (PK)                  │
│ transaction_id (FK)      │
│ account_id (FK)          │
│ entry_type               │
│ amount                   │
│ created_at               │
└──────────────────────────┘

# Key Features

- Double-entry bookkeeping system
- ACID-compliant transactions
- Transaction isolation for concurrency safety
- Validation to prevent negative balances
- Structured API architecture
- Clean relational database schema