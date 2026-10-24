# 🏗️ Bespoke ERP: Master Blueprint & Living Idea Log

> **Core Philosophy**: Building an independent, modern ERP from the ground up.
> **Reference Repositories**: `erpnext/` and `frontaccounting/` are strictly read-only references for database schemas, accounting rules, and domain models.

---

## 📌 1. Project Vision & Architecture Principles
- **Clean & Independent**: No legacy baggage. Purpose-built for performance and clean developer experience.
- **Strict Double-Entry Bookkeeping**: 
  - Every financial transaction must satisfy: $\sum \text{Debits} = \sum \text{Credits}$.
  - General Ledger (`gl_entries`) records are **immutable** (corrections require credit/debit reversing entries, never in-place row edits).
- **Extensibility**: Modular domain boundaries (Accounting, Inventory, Sales, Purchasing, HR/Payroll).

---

## 📚 2. Reference Insights (What We Learn from Reference Repos)

### From `erpnext/`
- **Validation Engine**: See `erpnext/accounts/general_ledger.py` for how entries are validated, rounded, and checked against fiscal periods and closed accounting periods.
- **Hierarchical COA**: Parent-child nested tree structure where only "leaf" accounts (not group accounts) can have transactions posted to them.
- **Dimensions & Cost Centers**: Attaching Project, Cost Center, or Department to individual GL rows for granular reporting.

### From `frontaccounting/`
- **Normalized Relational SQL Schema**: Direct SQL tables (`gl_trans`, `chart_master`, `chart_types`) offering clean blueprints for PostgreSQL / MySQL schemas.
- **Audit Trails**: Storing `type` (transaction type: invoice, journal entry, payment), `type_no`, `tran_date`, and counterparty links for clean auditing.

---

## 🎯 3. Phase 1: Core Financial Engine (Immediate Focus)

### 3.1 Chart of Accounts (COA) Model
A hierarchical tree structure representing all accounts in the organization:
- **Root Account Types**:
  1. **Assets** (Debit normal balance)
  2. **Liabilities** (Credit normal balance)
  3. **Equity / Capital** (Credit normal balance)
  4. **Income / Revenue** (Credit normal balance)
  5. **Expenses** (Debit normal balance)
- **Account Node Schema**:
  - `id`: Unique identifier (UUID or integer)
  - `code`: Standardized account number (e.g., `1000` Assets, `1100` Current Assets, `1110` Cash on Hand)
  - `name`: Account name
  - `parent_id`: Pointer to parent account node (null for root classes)
  - `is_group`: Boolean (true = folder/group, false = ledger account that accepts postings)
  - `root_type`: `Asset` | `Liability` | `Equity` | `Income` | `Expense`
  - `currency`: Default currency code (e.g., `USD`, `INR`)

### 3.2 General Ledger (GL Entry) Model
The central transaction table recording every movement of money:
- `id`: Unique ID
- `posting_date`: Date the transaction affects the books
- `account_id`: Foreign key to `accounts` table (must have `is_group = false`)
- `debit`: Amount debited (decimal)
- `credit`: Amount credited (decimal)
- `voucher_type`: Document type that caused this entry (`JournalEntry`, `SalesInvoice`, `Payment`)
- `voucher_no`: ID or sequence number of source document
- `party_type`: Optional (`Customer`, `Supplier`, `Employee`)
- `party_id`: Foreign key to party
- `remarks`: Description of the transaction
- `is_cancelled`: Boolean flag for cancellation audit trail

### 3.3 Core Reports (Target Deliverables)
1. **General Ledger Enquiry**: Filtered list of transactions for any account over a date range with running balance.
2. **Trial Balance**: Verification that all ledger debits equal credits at any given cut-off date.
3. **Profit & Loss (Income Statement)**: $\text{Total Income} - \text{Total Expenses} = \text{Net Profit}$.
4. **Balance Sheet**: $\text{Assets} = \text{Liabilities} + \text{Equity}$.

---

## 💡 4. Technology Stack & Design Decisions
- **Project Location**: All development is contained within **`erp_core/`**
  ```
  c:\Users\yokes\erp\
  ├── erp_core/              <-- Bespoke ERP Project Root
  │   ├── backend/           # Python 3.12 + Django + DRF
  │   └── frontend/          # React + Vite + TypeScript + TanStack Table
  ├── erpnext/               # Read-only architectural reference
  ├── frontaccounting/       # Read-only schema & SQL reference
  ├── AGENTS.md              # Global workspace directives
  └── ERP_BLUEPRINT.md       # Master project specification
  ```
- **Backend**: Python 3.12 + **Django** + **Django REST Framework (DRF)**
  - Financial calculations: Python `Decimal` (exact fractional math)
  - Atomic transactions: `transaction.atomic` for zero-leak double-entry posting
  - Custom Posting Engine service layer (`core/services/posting_engine.py`)
- **Database**: **MariaDB / MySQL 8.0+** (InnoDB engine for strict ACID, Recursive CTEs for Chart of Accounts hierarchy). *SQLite can be used for instant offline zero-config unit tests.*
- **Frontend**: **React** (Vite + TypeScript)
  - High-performance data tables: `@tanstack/react-table` (for ledger grids with sorting/filtering/virtualization)
  - Visual hierarchy: Tree view component for Chart of Accounts
  - Icons & Styling: `lucide-react`, clean modern styling / Tailwind CSS

---

## 📝 5. Idea Log & Backlog
- [ ] Finalize preferred backend language & database engine.
- [ ] Define initial standard Chart of Accounts template (JSON/SQL).
- [ ] Implement double-entry transaction manager with atomic database transactions.
- [ ] Add support for Multi-currency transactions & exchange gain/loss calculation.
- [ ] Design modern Chart of Accounts visual tree UI.
