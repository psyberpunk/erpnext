# CLAUDE.MD - ERPNext Development Guide

> Documentation for AI assistants and developers working with the ERPNext codebase

---

## 1. Project Overview

**ERPNext** is a powerful, intuitive, and fully open-source ERP (Enterprise Resource Planning) system built on the Frappe Framework.

### Key Information

- **Version:** 17.0.0-dev (Development version)
- **License:** GNU General Public License v3
- **Repository:** https://github.com/frappe/erpnext
- **Publisher:** Frappe Technologies Pvt. Ltd.

### Tech Stack

- **Backend:** Python 3.14+, Frappe Framework (>=17.0.0-dev)
- **Database:** MariaDB
- **Frontend:** JavaScript, Vue.js (via Frappe UI)
- **Development Tool:** Frappe Bench

### Codebase Size

- **Total Files:** 2,533 Python files
- **Main Modules:** 21 functional modules
- **Test Files:** 286 DocType test files
- **Total Size:** ~110MB

### Purpose

ERPNext provides comprehensive business management covering:
- Financial accounting and invoicing
- Inventory and warehouse management
- Manufacturing and production planning
- Customer relationship management (CRM)
- Project management and timesheets
- Human resources and payroll
- Asset management
- And much more...

---

## 2. Codebase Structure

### Core Architecture

ERPNext is built on the **Frappe Framework** which provides:
- ORM (Object-Relational Mapping)
- Web routing and authentication
- API framework
- UI components
- Background job system

**Key Architectural Concepts:**

1. **DocType-based:** All business objects are defined as DocTypes (data models)
2. **Hook System:** Central configuration in `/erpnext/hooks.py` (686 lines)
3. **Controller Pattern:** Business logic in Python controllers
4. **Regional Customization:** Support for country-specific features

### Module Organization

ERPNext has **21 primary modules** located in `/home/user/erpnext/erpnext/`:

#### Primary Business Modules

1. **accounts/** - Double-entry accounting, invoicing, payments, general ledger, payment ledger
2. **stock/** - Inventory management, items, warehouses, stock entries, serial/batch tracking
3. **buying/** - Purchase orders, supplier management, procurement, request for quotation
4. **selling/** - Sales orders, quotations, customers, sales persons
5. **manufacturing/** - Production planning, Bill of Materials (BOM), work orders, workstations
6. **projects/** - Project management, tasks, timesheets, activity tracking
7. **assets/** - Fixed asset management, depreciation, maintenance
8. **crm/** - Customer relationship management: leads, opportunities, prospects
9. **support/** - Issue tracking, service level agreements, warranty claims

#### Supporting Modules

10. **setup/** - Company setup, employees, authorization, global settings
11. **utilities/** - Bulk operations, videos, activation utilities
12. **portal/** - Customer/supplier portal interfaces
13. **maintenance/** - Maintenance schedules, visits, equipment tracking
14. **regional/** - Country-specific customizations (Italy, UAE, Saudi Arabia, France, India, etc.)
15. **communication/** - Email campaigns, newsletters
16. **telephony/** - Call logs, VoIP integration
17. **quality_management/** - Quality reviews, procedures, goals
18. **subcontracting/** - Subcontracting orders and receipts
19. **edi/** - Electronic Data Interchange

#### Infrastructure Modules

20. **controllers/** - Base controller classes shared across modules
21. **patches/** - Database migration scripts organized by version (v10_0, v11_0, v12_0, v13_0, v14_0, v15_0, etc.)
22. **tests/** - Module-level integration tests and test utilities
23. **templates/** - Jinja templates, web forms, print formats
24. **public/** - Static assets (JavaScript, CSS, images, sounds)
25. **startup/** - Boot session, notifications, leaderboards, filters
26. **config/** - Workspace and desktop configurations
27. **domains/** - Domain-specific configs (manufacturing, distribution, retail, services, education, healthcare, agriculture, non_profit)

### Directory Structure Pattern

Each module follows a consistent structure:

#### DocType Structure
```
/erpnext/<module>/doctype/<doctype_name>/
├── <doctype_name>.json          # Field definitions, permissions, metadata
├── <doctype_name>.py            # Server-side controller (Python)
├── <doctype_name>.js            # Client-side controller (JavaScript)
├── test_<doctype_name>.py       # Unit tests
├── <doctype_name>_dashboard.py  # Dashboard widgets (optional)
├── <doctype_name>_list.js       # List view customizations (optional)
└── test_records.json            # Test fixtures (optional)
```

#### Report Structure
```
/erpnext/<module>/report/<report_name>/
├── <report_name>.py       # Report query logic (implements execute())
├── <report_name>.js       # Client-side filters and settings
├── <report_name>.json     # Report metadata (name, type, ref_doctype)
├── <report_name>.html     # Custom HTML template (optional)
└── test_<report_name>.py  # Report tests (optional)
```

#### Page Structure
```
/erpnext/<module>/page/<page_name>/
├── <page_name>.py         # Server-side logic
├── <page_name>.js         # Client-side logic
├── <page_name>.json       # Page metadata
└── <page_name>.html       # Page template
```

---

## 3. Development Workflow

### Frappe Bench Basics

ERPNext uses **Frappe Bench** for development management. Bench is a command-line tool that orchestrates the development environment.

#### Initial Setup

```bash
# Start development server (all processes: web, worker, redis, socketio)
bench start

# Create a new site
bench new-site <sitename>

# Install ERPNext app on site
bench --site <sitename> install-app erpnext

# Get ERPNext app for development
bench get-app https://github.com/frappe/erpnext
```

#### Common Development Commands

```bash
# Python console (interactive REPL)
bench --site <sitename> console

# Execute a Python function
bench --site <sitename> execute <module.path.function>

# Clear cache (required after code changes)
bench --site <sitename> clear-cache

# Clear website cache
bench --site <sitename> clear-website-cache

# Migrate database (run patches)
bench --site <sitename> migrate

# Build assets (JS/CSS)
bench build --app erpnext

# Watch and rebuild assets automatically
bench watch

# Restart processes
bench restart
```

### Understanding Frappe Framework Integration

**Dependency:** ERPNext requires `frappe>=17.0.0-dev,<18.0.0`

Frappe provides the foundation:
- **ORM:** `frappe.get_doc()`, `frappe.get_list()`, `frappe.db.get_value()`
- **Routing:** Web routes, API endpoints
- **Authentication:** User management, permissions
- **API Framework:** RESTful API with `@frappe.whitelist()`
- **UI Components:** Form builders, list views, dashboards
- **Background Jobs:** Scheduler for cron jobs

### Hook System

**File:** `/erpnext/hooks.py` (686 lines)

This is the central configuration file that controls:

- **App Metadata:** Name, version, description
- **DocType Event Handlers:** `validate`, `on_submit`, `on_cancel`, etc.
- **Scheduled Tasks:** Hourly, daily, weekly, monthly cron jobs
- **Web Routes:** Portal pages, website generators
- **Boot Session Data:** Data loaded on login
- **Regional Overrides:** Country-specific function overrides
- **Permission Overrides:** Custom permission logic
- **Global Search:** Search configuration
- **Notification Config:** Email and system notifications

**Example hook configuration:**
```python
doc_events = {
    "Sales Invoice": {
        "validate": "erpnext.accounts.doctype.sales_invoice.sales_invoice.validate_inter_company_party",
        "on_submit": "erpnext.accounts.doctype.sales_invoice.sales_invoice.update_linked_doc",
    }
}

scheduler_events = {
    "daily": [
        "erpnext.stock.reorder_item.reorder_item",
        "erpnext.support.doctype.issue.issue.auto_close_tickets"
    ]
}
```

---

## 4. Common Tasks

### Adding a New DocType

**Steps:**

1. **Navigate to module:** `/erpnext/<module>/doctype/`
2. **Create DocType:** Use Frappe UI at `/app/doctype/new` or create files manually
3. **Define fields:** Edit `<doctype>.json` or use Form Builder in UI
4. **Create controller:** `<doctype>.py` - inherit from appropriate base class
5. **Add client logic:** `<doctype>.js` - form setup, validation, custom buttons
6. **Write tests:** `test_<doctype>.py` - inherit from `IntegrationTestCase`

**Controller Example:**
```python
# erpnext/stock/doctype/item/item.py
from frappe.model.document import Document

class Item(Document):
    def validate(self):
        # Pre-save validation
        self.validate_item_code()
        self.validate_item_name()

    def on_submit(self):
        # Post-submit logic
        pass

    def validate_item_code(self):
        if not self.item_code:
            frappe.throw(_("Item Code is required"))
```

### Controller Base Classes

Located in `/erpnext/controllers/`:

- **`AccountsController`** - Base for all accounting transactions (Sales Invoice, Purchase Invoice, etc.)
- **`SellingController`** - Base for sales documents (inherits from AccountsController)
- **`BuyingController`** - Base for purchase documents (inherits from AccountsController)
- **`StockController`** - Base for stock transactions (Stock Entry, Delivery Note, etc.)
- **`TransactionBase`** - Base for all transactions (from Frappe)

**Usage:**
```python
from erpnext.controllers.selling_controller import SellingController

class SalesInvoice(SellingController):
    def validate(self):
        super().validate()  # Call parent validation
        # Custom validation

    def on_submit(self):
        super().on_submit()  # Call parent submit logic
        # Custom submit logic
```

### Adding a New Report

**Steps:**

1. **Create report directory:** `/erpnext/<module>/report/<report_name>/`
2. **Define metadata:** `<report_name>.json` - report type, reference DocType
3. **Write query logic:** `<report_name>.py` - implement `execute()` function
4. **Add filters:** `<report_name>.js` - define filter fields
5. **Write tests:** `test_<report_name>.py`

**Report Example:**
```python
# erpnext/stock/report/stock_balance/stock_balance.py
import frappe
from frappe import _

def execute(filters=None):
    columns = get_columns()
    data = get_data(filters)
    return columns, data

def get_columns():
    return [
        {"label": _("Item"), "fieldname": "item_code", "fieldtype": "Link", "options": "Item", "width": 150},
        {"label": _("Quantity"), "fieldname": "qty", "fieldtype": "Float", "width": 100},
    ]

def get_data(filters):
    # Query logic
    return frappe.db.sql("""...""", filters, as_dict=1)
```

### Adding API Methods

**Server-side (Python):**
```python
# Whitelist method for API access
@frappe.whitelist()
def get_item_details(item_code, company):
    item = frappe.get_doc("Item", item_code)
    return {
        "item_name": item.item_name,
        "description": item.description,
        "stock_uom": item.stock_uom
    }
```

**Client-side (JavaScript):**
```javascript
// Call API method from client
frappe.call({
    method: 'erpnext.stock.doctype.item.item.get_item_details',
    args: {
        item_code: 'ITEM-001',
        company: 'My Company'
    },
    callback: function(r) {
        console.log(r.message);
    }
});
```

### Modifying Controllers

**Key methods to override:**

- `validate()` - Pre-save validation (runs before save)
- `before_save()` - Before save operations
- `on_submit()` - When document is submitted (posted)
- `on_cancel()` - When document is cancelled
- `before_insert()` - Before new document creation
- `on_update()` - After document save
- `onload()` - Load additional data for form view

**Example:**
```python
class SalesInvoice(SellingController):
    def validate(self):
        super().validate()
        self.calculate_totals()
        self.set_missing_values()
        self.validate_payment_schedule()

    def on_submit(self):
        super().on_submit()
        self.make_gl_entries()
        self.update_stock_ledger()
        self.update_sales_order()
```

### Working with Accounting

**Key Files:**
- **General Ledger:** `/erpnext/accounts/general_ledger.py`
- **Payment Ledger:** `/erpnext/accounts/doctype/payment_ledger_entry/`
- **Party Accounts:** `/erpnext/accounts/party.py`
- **Tax Calculation:** `/erpnext/controllers/taxes_and_totals.py`

**Creating GL Entries:**
```python
from erpnext.accounts.general_ledger import make_gl_entries

gl_entries = [
    {
        "account": "Debtors - XYZ",
        "debit": 1000,
        "credit": 0,
        "party_type": "Customer",
        "party": "CUST-001"
    }
]

make_gl_entries(gl_entries, cancel=False)
```

### Working with Stock

**Key Files:**
- **Stock Ledger:** `/erpnext/stock/doctype/stock_ledger_entry/`
- **Serial & Batch:** `/erpnext/stock/serial_batch_bundle.py`
- **Valuation:** `/erpnext/stock/stock_ledger.py`
- **Reposting:** `/erpnext/stock/doctype/repost_item_valuation/`

**Stock Entry Example:**
```python
stock_entry = frappe.get_doc({
    "doctype": "Stock Entry",
    "stock_entry_type": "Material Receipt",
    "company": "My Company",
    "items": [{
        "item_code": "ITEM-001",
        "qty": 100,
        "t_warehouse": "Stores - XYZ"
    }]
})
stock_entry.submit()
```

---

## 5. Testing & Quality

### Running Tests

#### Via Bench

```bash
# Run all tests for ERPNext
bench --site <sitename> run-tests --app erpnext

# Run specific module tests
bench --site <sitename> run-tests --app erpnext --module erpnext.stock

# Run specific DocType tests
bench --site <sitename> run-tests --app erpnext --doctype "Sales Invoice"

# Run specific test case
bench --site <sitename> run-tests --app erpnext --test erpnext.stock.doctype.item.test_item.TestItem.test_variant_item_code

# Run with coverage
bench --site <sitename> run-tests --app erpnext --coverage

# Run tests in parallel (faster)
bench --site <sitename> run-tests --app erpnext --parallel
```

### Test Patterns

**Test File Structure:**
```python
# erpnext/stock/doctype/item/test_item.py
from frappe.tests import IntegrationTestCase
import frappe

class TestItem(IntegrationTestCase):
    def setUp(self):
        super().setUp()
        # Setup code runs before each test
        self.test_item = make_item()

    def test_item_creation(self):
        item = frappe.get_doc("Item", self.test_item.name)
        self.assertEqual(item.item_name, "Test Item")

    def test_item_validation(self):
        item = frappe.get_doc({
            "doctype": "Item",
            "item_code": "TEST-001"
        })
        # This should raise validation error
        with self.assertRaises(frappe.ValidationError):
            item.save()

def make_item(**kwargs):
    """Helper function to create test item"""
    item = frappe.get_doc({
        "doctype": "Item",
        "item_code": kwargs.get("item_code", "_Test Item"),
        "item_name": kwargs.get("item_name", "Test Item"),
        "item_group": kwargs.get("item_group", "_Test Item Group"),
        "is_stock_item": 1
    })
    item.insert(ignore_if_duplicate=True)
    return item
```

**Test Fixtures:**

Use `test_records.json` for test data:
```json
[
    {
        "doctype": "Item",
        "item_code": "_Test Item",
        "item_name": "Test Item",
        "item_group": "_Test Item Group",
        "is_stock_item": 1
    }
]
```

### Code Quality Tools

#### Ruff (Python Linting & Formatting)

Configuration in `/pyproject.toml`:
- Line length: 110 characters
- Indent style: tabs
- Target version: Python 3.10+

```bash
# Check for linting issues
ruff check .

# Auto-fix issues
ruff check --fix .

# Format code
ruff format .

# Check specific module
ruff check erpnext/accounts/

# Format specific file
ruff format erpnext/accounts/utils.py
```

#### ESLint (JavaScript Linting)

Configuration in `.eslintrc`:

```bash
# Lint JavaScript files
eslint erpnext/public/js/

# Fix auto-fixable issues
eslint --fix erpnext/public/js/
```

#### Prettier (Code Formatting)

```bash
# Format SCSS files
prettier --write "erpnext/**/*.scss"

# Format specific file
prettier --write "erpnext/public/css/erpnext.scss"
```

### Pre-commit Hooks

Configuration in `.pre-commit-config.yaml`:

**Hooks:**
- Trailing whitespace removal
- YAML syntax validation
- JSON syntax validation
- Python AST validation
- Ruff linting and formatting
- Prettier formatting
- ESLint

**Usage:**
```bash
# Install pre-commit hooks
pre-commit install

# Run manually on all files
pre-commit run --all-files

# Skip hooks (NOT recommended)
git commit --no-verify
```

### Test Data Setup

**Before Tests Hook:** `/erpnext/setup/utils.py::before_tests()`

This function:
- Creates test company
- Sets up default roles and permissions
- Creates test employees, customers, suppliers
- Sets up chart of accounts
- Configures default settings

### CI/CD Pipelines

Located in `.github/workflows/`:

- **server-tests-mariadb.yml** - Main test suite with MariaDB
- **linters.yml** - Code quality checks (Ruff, ESLint, Prettier)
- **patch.yml** - Database patch testing
- **release.yml** - Release automation
- **docker-release.yml** - Docker image building

---

## 6. Key Files & Entry Points

### Critical Configuration Files

#### `/erpnext/hooks.py` (686 lines)
Central app configuration file that controls:
- App metadata (name, version, description)
- DocType event handlers (`doc_events`)
- Scheduled tasks (`scheduler_events`)
- Regional overrides
- Portal menu items
- Web routes and website generators
- Global search configuration
- Notification settings

#### `/erpnext/__init__.py` (196 lines)
App initialization and core utilities:
- Version: `__version__ = "17.0.0-dev"`
- `get_default_company()` - Get default company
- `get_default_currency()` - Get default currency
- `@allow_regional` - Decorator for regional customization
- Company-related helper functions

#### `/pyproject.toml`
Python project configuration:
- **Dependencies:** Frappe, Unidecode, barcodenumber, rapidfuzz, holidays, googlemaps, plaid-python, mt-940, etc.
- **Ruff Configuration:** Line length, indent style, target Python version
- **Build System:** Setuptools configuration

#### `/.pre-commit-config.yaml`
Pre-commit hook configuration for automated code quality checks.

### Important Controller Files

#### `/erpnext/controllers/accounts_controller.py` (146KB)
**Base class for all accounting transactions**

Key methods:
- `validate()` - Master validation
- `set_missing_values()` - Set default values
- `validate_party()` - Customer/Supplier validation
- `make_gl_entries()` - Create general ledger entries
- `make_payment_gl_entries()` - Payment GL entries
- `calculate_taxes_and_totals()` - Tax calculation

Used by: Sales Invoice, Purchase Invoice, Sales Order, Purchase Order, etc.

#### `/erpnext/controllers/selling_controller.py` (36KB)
**Base class for sales transactions**

Inherits from: `AccountsController`

Additional features:
- Territory management
- Sales person assignment
- Commission calculation
- Customer-specific pricing

Used by: Sales Order, Quotation, Sales Invoice, Delivery Note

#### `/erpnext/controllers/buying_controller.py` (41KB)
**Base class for purchase transactions**

Inherits from: `AccountsController`

Additional features:
- Supplier quotation handling
- Purchase order logic
- Landed cost allocation

Used by: Purchase Order, Supplier Quotation, Purchase Invoice, Purchase Receipt

#### `/erpnext/controllers/stock_controller.py` (74KB)
**Base class for stock transactions**

Key methods:
- `make_sl_entries()` - Create stock ledger entries
- `make_gl_entries_on_cancel()` - Reverse GL entries
- `update_serial_nos()` - Serial number handling
- `get_pending_qty()` - Calculate pending quantities

Used by: Stock Entry, Delivery Note, Purchase Receipt, Stock Reconciliation

#### `/erpnext/controllers/taxes_and_totals.py` (44KB)
**Tax calculation engine**

Used by all invoicing documents to:
- Calculate line-level taxes
- Apply tax templates
- Calculate discounts
- Compute net total and grand total
- Handle multi-currency

### Module Entry Points

Each module has standard entry files:

- `__init__.py` - Module initialization
- `utils.py` - Common utility functions
- `doctype/` - All DocType definitions
- `report/` - All reports

### Important Utility Modules

#### `/erpnext/accounts/utils.py` (80KB)
Accounting utilities:
- `get_fiscal_year()` - Get fiscal year for date
- `get_account_currency()` - Get account currency
- `get_balance_on()` - Get account balance
- `make_payment_entry()` - Create payment entry
- `reconcile_against_document()` - Reconcile payments

#### `/erpnext/accounts/party.py` (33KB)
Party (Customer/Supplier) management:
- `get_party_account()` - Get party account
- `get_due_date()` - Calculate due date
- `get_party_details()` - Get party information
- `get_outstanding_amount()` - Calculate outstanding

#### `/erpnext/stock/utils.py`
Stock utilities:
- `get_latest_stock_qty()` - Get current stock level
- `get_incoming_rate()` - Get valuation rate
- `get_stock_balance()` - Get stock balance
- `get_stock_value_on()` - Get stock value at date

#### `/erpnext/setup/utils.py`
Setup and installation utilities:
- `before_tests()` - Initialize test environment
- `get_exchange_rate()` - Fetch exchange rates
- `enable_all_roles_and_domains()` - Setup permissions

### General Ledger Files

- **`/erpnext/accounts/general_ledger.py`** - GL entry creation and querying
- **`/erpnext/accounts/doctype/gl_entry/`** - GL Entry DocType
- **`/erpnext/accounts/doctype/payment_ledger_entry/`** - Payment Ledger (new)

### Stock Ledger Files

- **`/erpnext/stock/stock_ledger.py`** - Stock ledger management
- **`/erpnext/stock/doctype/stock_ledger_entry/`** - Stock Ledger Entry DocType
- **`/erpnext/stock/serial_batch_bundle.py`** - Serial/Batch handling

---

## 7. Patterns & Conventions

### Naming Conventions

#### Python

- **Module names:** `snake_case`
- **Class names:** `PascalCase` (must match DocType name)
- **Function names:** `snake_case`
- **Private methods:** `_leading_underscore`
- **Constants:** `UPPER_SNAKE_CASE`

#### DocTypes

- **DocType names:** Use spaces (e.g., "Sales Invoice")
- **File names:** Use underscores (e.g., `sales_invoice.py`)
- **Class names:** PascalCase matching DocType (e.g., `class SalesInvoice`)

#### JavaScript

- **Functions:** `camelCase`
- **Form reference:** Use `cur_frm` for current form
- **Field reference:** Use `frm.doc.field_name`

### Code Patterns

#### Controller Inheritance

```python
from erpnext.controllers.selling_controller import SellingController

class SalesInvoice(SellingController):
    def validate(self):
        super().validate()  # Always call parent
        self.validate_pos_invoice()
        self.set_missing_values()

    def on_submit(self):
        super().on_submit()
        self.update_stock_ledger()
        self.make_gl_entries()

    def on_cancel(self):
        super().on_cancel()
        self.update_stock_ledger()
        self.make_gl_entries()
```

#### Whitelisted API Methods

```python
import frappe

@frappe.whitelist()
def get_item_details(item_code, company):
    """
    API method callable from client-side JavaScript
    """
    if not item_code:
        frappe.throw(_("Item Code is required"))

    item = frappe.get_doc("Item", item_code)

    return {
        "item_name": item.item_name,
        "description": item.description,
        "stock_uom": item.stock_uom
    }
```

#### Regional Overrides

```python
from erpnext import allow_regional

@allow_regional
def calculate_tax(invoice):
    """
    Default implementation
    Can be overridden by regional module
    e.g., erpnext.regional.italy.utils.calculate_tax
    """
    # Default tax calculation logic
    pass
```

#### Database Queries

Use Frappe's query builder (preferred over raw SQL):

```python
from frappe.query_builder import DocType

# Example 1: Simple query
Item = DocType('Item')
items = (
    frappe.qb.from_(Item)
    .select(Item.name, Item.item_name, Item.stock_uom)
    .where(Item.disabled == 0)
    .where(Item.is_stock_item == 1)
).run(as_dict=True)

# Example 2: Join query
StockLedgerEntry = DocType('Stock Ledger Entry')
Warehouse = DocType('Warehouse')

stock_data = (
    frappe.qb.from_(StockLedgerEntry)
    .join(Warehouse).on(StockLedgerEntry.warehouse == Warehouse.name)
    .select(
        StockLedgerEntry.item_code,
        StockLedgerEntry.qty_after_transaction,
        Warehouse.warehouse_name
    )
    .where(StockLedgerEntry.item_code == 'ITEM-001')
).run(as_dict=True)
```

#### Form Scripts (JavaScript)

```javascript
frappe.ui.form.on('Sales Invoice', {
    refresh: function(frm) {
        // Runs when form loads or refreshes
        if (frm.doc.docstatus === 1) {
            frm.add_custom_button(__('Payment Entry'), function() {
                // Custom button logic
            });
        }
    },

    customer: function(frm) {
        // Runs when customer field changes
        if (frm.doc.customer) {
            frappe.call({
                method: 'erpnext.accounts.party.get_party_details',
                args: {
                    party: frm.doc.customer,
                    party_type: 'Customer'
                },
                callback: function(r) {
                    frm.set_value('customer_name', r.message.party_name);
                }
            });
        }
    },

    before_save: function(frm) {
        // Runs before document is saved
        frm.doc.total_qty = 0;
        frm.doc.items.forEach(function(item) {
            frm.doc.total_qty += item.qty;
        });
    }
});

// Child table events
frappe.ui.form.on('Sales Invoice Item', {
    qty: function(frm, cdt, cdn) {
        // Runs when qty changes in child table
        let row = locals[cdt][cdn];
        frappe.model.set_value(cdt, cdn, 'amount', row.qty * row.rate);
    }
});
```

### Error Handling

#### Custom Exceptions

```python
import frappe
from frappe import _

# Throw validation error
frappe.throw(_("Invalid quantity"), frappe.ValidationError)

# Throw with custom exception
class InsufficientStockError(frappe.ValidationError):
    pass

if stock_qty < required_qty:
    frappe.throw(_("Insufficient stock for item {0}").format(item_code), InsufficientStockError)

# Message without exception
frappe.msgprint(_("Document saved successfully"))
```

#### Translation

Always use `_()` for translatable strings:

```python
from frappe import _

# Translatable message
frappe.throw(_("Item {0} not found").format(item_code))

# In templates
<h3>{{ _("Sales Invoice") }}</h3>
```

### Auto-generated Type Annotations

Many DocTypes have auto-generated type hints:

```python
from frappe.model.document import Document

class Item(Document):
    # begin: auto-generated types
    # This code is auto-generated. Do not modify anything in this block.

    from typing import TYPE_CHECKING
    if TYPE_CHECKING:
        from frappe.types import DF
        from erpnext.stock.doctype.item_barcode.item_barcode import ItemBarcode

        item_code: DF.Data
        item_name: DF.Data
        item_group: DF.Link
        description: DF.TextEditor | None
        is_stock_item: DF.Check
        barcodes: DF.Table[ItemBarcode]
    # end: auto-generated types

    def validate(self):
        # Your code here
        pass
```

**Note:** Do NOT manually edit the auto-generated types section.

### Testing Patterns

#### Test Helper Functions

Create reusable test data helpers:

```python
def make_item(**kwargs):
    """Helper to create test item"""
    item = frappe.get_doc({
        "doctype": "Item",
        "item_code": kwargs.get("item_code", "_Test Item"),
        "item_name": kwargs.get("item_name", "Test Item"),
        "item_group": kwargs.get("item_group", "_Test Item Group"),
        "stock_uom": kwargs.get("stock_uom", "Nos"),
        "is_stock_item": kwargs.get("is_stock_item", 1)
    }).insert(ignore_if_duplicate=True)
    return item

def make_stock_entry(**kwargs):
    """Helper to create test stock entry"""
    se = frappe.get_doc({
        "doctype": "Stock Entry",
        "stock_entry_type": kwargs.get("stock_entry_type", "Material Receipt"),
        "company": kwargs.get("company", "_Test Company"),
        "items": kwargs.get("items", [])
    })
    se.insert()
    if kwargs.get("submit", False):
        se.submit()
    return se
```

#### Test Dependencies

Declare test dependencies:

```python
# At module level
IGNORE_TEST_RECORD_DEPENDENCIES = ["BOM"]
EXTRA_TEST_RECORD_DEPENDENCIES = ["Warehouse", "Item Group"]
```

---

## 8. Navigation Tips

### Finding Code by Business Function

#### For Accounting Tasks
- **Start:** `/erpnext/accounts/`
- **Controllers:** `/erpnext/controllers/accounts_controller.py`
- **GL Posting:** `/erpnext/accounts/general_ledger.py`
- **Party Management:** `/erpnext/accounts/party.py`
- **Tax Calculation:** `/erpnext/controllers/taxes_and_totals.py`
- **Key DocTypes:** Sales Invoice, Purchase Invoice, Payment Entry, Journal Entry

#### For Stock/Inventory
- **Start:** `/erpnext/stock/`
- **Controllers:** `/erpnext/controllers/stock_controller.py`
- **Item Master:** `/erpnext/stock/doctype/item/`
- **Stock Ledger:** `/erpnext/stock/doctype/stock_ledger_entry/`
- **Serial/Batch:** `/erpnext/stock/serial_batch_bundle.py`
- **Key DocTypes:** Item, Stock Entry, Delivery Note, Purchase Receipt, Warehouse

#### For Manufacturing
- **Start:** `/erpnext/manufacturing/`
- **BOM:** `/erpnext/manufacturing/doctype/bom/`
- **Work Order:** `/erpnext/manufacturing/doctype/work_order/`
- **Production Planning:** `/erpnext/manufacturing/doctype/production_plan/`
- **Key DocTypes:** BOM, Work Order, Production Plan, Job Card, Workstation

#### For Sales/Purchase
- **Sales:** `/erpnext/selling/` + `/erpnext/accounts/doctype/sales_invoice/`
- **Purchase:** `/erpnext/buying/` + `/erpnext/accounts/doctype/purchase_invoice/`
- **Key DocTypes:** Quotation, Sales Order, Purchase Order, Delivery Note

#### For Projects
- **Start:** `/erpnext/projects/`
- **Key DocTypes:** Project, Task, Timesheet, Activity Type

#### For CRM
- **Start:** `/erpnext/crm/`
- **Key DocTypes:** Lead, Opportunity, Prospect, Campaign

### Search Strategies

#### Find DocType Definition
```bash
# Find DocType JSON file
find /home/user/erpnext/erpnext -name "*<keyword>*.json" -path "*/doctype/*"

# Example: Find Sales Invoice
find /home/user/erpnext/erpnext -name "*sales_invoice*.json" -path "*/doctype/*"
```

#### Find API Methods
```bash
# Find whitelisted methods
grep -r "@frappe.whitelist" erpnext/<module>/

# Example: Find whitelisted methods in accounts
grep -r "@frappe.whitelist" erpnext/accounts/
```

#### Find Tests
```bash
# Find all test files
find /home/user/erpnext/erpnext -name "test_*.py"

# Find tests for specific module
find /home/user/erpnext/erpnext/accounts -name "test_*.py"
```

#### Find Reports
```bash
# Find report directories
find /home/user/erpnext/erpnext -type d -name "report"

# Find specific report
find /home/user/erpnext/erpnext -name "*stock_balance*" -path "*/report/*"
```

#### Find Event Handlers
Check `/erpnext/hooks.py` `doc_events` dictionary for event handlers.

#### Find Scheduled Tasks
Check `/erpnext/hooks.py` `scheduler_events` for cron jobs.

### Module Cross-References

**Common cross-module dependencies:**

- All transaction modules use `/erpnext/controllers/` base classes
- Accounting modules reference `/erpnext/accounts/`
- Stock transactions link to `/erpnext/stock/`
- Setup data defined in `/erpnext/setup/doctype/`
- Regional customizations in `/erpnext/regional/<country>/`

### Understanding Document Flow

#### Sales Flow
```
Lead → Opportunity → Quotation → Sales Order → Delivery Note → Sales Invoice → Payment Entry
```

#### Purchase Flow
```
Material Request → Supplier Quotation → Purchase Order → Purchase Receipt → Purchase Invoice → Payment Entry
```

#### Manufacturing Flow
```
Sales Order → Production Plan → Work Order → Stock Entry (Manufacture) → Delivery Note
```

#### Stock Flow
```
Purchase Receipt → Stock Entry (Transfer) → Delivery Note
```

---

## 9. Helpful Commands

### Bench Commands

#### Development Server

```bash
# Start all processes (web, worker, redis, socketio)
bench start

# Start only web server
bench serve --port 8000

# Watch and auto-rebuild assets
bench watch

# Build assets for production
bench build --app erpnext

# Build with minification
bench build --app erpnext --production
```

#### Cache Management

```bash
# Clear all cache
bench --site <sitename> clear-cache

# Clear website cache
bench --site <sitename> clear-website-cache

# Rebuild cache
bench --site <sitename> rebuild-cache
```

#### Database Operations

```bash
# Run migrations (patches)
bench --site <sitename> migrate

# Skip patches and only run Frappe migrations
bench --site <sitename> migrate --skip-failing

# Enter MariaDB console
bench --site <sitename> mariadb

# Backup site
bench --site <sitename> backup
bench --site <sitename> backup --with-files

# Restore backup
bench --site <sitename> restore /path/to/backup.sql.gz
```

#### Code Execution

```bash
# Python console (REPL)
bench --site <sitename> console

# Execute Python function
bench --site <sitename> execute erpnext.stock.reorder_item.reorder_item

# Execute with arguments
bench --site <sitename> execute "frappe.get_doc('Item', 'ITEM-001').save()"

# Run scheduler manually (all jobs)
bench --site <sitename> scheduler

# Run specific scheduler event
bench --site <sitename> trigger-scheduler-event daily
```

#### Testing

```bash
# Run all ERPNext tests
bench --site <sitename> run-tests --app erpnext

# Run specific module
bench --site <sitename> run-tests --app erpnext --module erpnext.accounts

# Run specific DocType
bench --site <sitename> run-tests --app erpnext --doctype "Sales Invoice"

# Run specific test method
bench --site <sitename> run-tests --app erpnext --test erpnext.accounts.doctype.sales_invoice.test_sales_invoice.TestSalesInvoice.test_sales_invoice_calculation

# Run with coverage
bench --site <sitename> run-tests --app erpnext --coverage

# Run tests in parallel (faster)
bench --site <sitename> run-tests --app erpnext --parallel

# Run only failed tests
bench --site <sitename> run-tests --app erpnext --failfast
```

#### Site Management

```bash
# Create new site
bench new-site <sitename>

# Drop site (delete)
bench drop-site <sitename>

# Install app on site
bench --site <sitename> install-app erpnext

# Uninstall app from site
bench --site <sitename> uninstall-app erpnext

# Reinstall app (fresh install)
bench --site <sitename> reinstall

# List all sites
bench --site all list-apps
```

#### Configuration

```bash
# Set config value
bench --site <sitename> set-config developer_mode 1
bench --site <sitename> set-config logging 2

# Add system manager user
bench --site <sitename> add-system-manager user@example.com

# Enable developer mode
bench --site <sitename> set-config developer_mode 1

# Enable debug mode
bench --site <sitename> set-config server_script_enabled 1
```

#### App Management

```bash
# Get app from GitHub
bench get-app https://github.com/frappe/erpnext

# Get specific branch
bench get-app https://github.com/frappe/erpnext --branch develop

# Remove app from bench
bench remove-app erpnext

# Update app
bench update --app erpnext

# Update all apps and bench
bench update
```

### Git Workflow

#### Branches

- **Main development:** `develop`
- **Stable releases:** `version-14`, `version-15`, etc.
- **Feature branches:** `feature/<feature-name>`
- **Hotfix branches:** `hotfix/<issue-number>`

#### Best Practices

```bash
# Always create feature branch from develop
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# Make changes, commit
git add .
git commit -m "feat: add new feature"

# Push to remote
git push origin feature/my-feature

# Create pull request on GitHub
# Never commit directly to develop
```

### Pre-commit Hooks

```bash
# Install pre-commit hooks
pre-commit install

# Run manually on all files
pre-commit run --all-files

# Run specific hook
pre-commit run ruff --all-files

# Update hooks to latest versions
pre-commit autoupdate

# Skip hooks (NOT recommended)
git commit --no-verify -m "message"
```

### Linting & Formatting

#### Python (Ruff)

```bash
# Check for issues
ruff check .

# Auto-fix issues
ruff check --fix .

# Format code
ruff format .

# Check specific module
ruff check erpnext/accounts/

# Format specific file
ruff format erpnext/accounts/utils.py

# Check and fix in one command
ruff check --fix . && ruff format .
```

#### JavaScript (ESLint)

```bash
# Lint JavaScript files
eslint erpnext/public/js/

# Fix auto-fixable issues
eslint --fix erpnext/public/js/

# Lint specific file
eslint erpnext/public/js/utils.js
```

#### CSS/SCSS (Prettier)

```bash
# Format SCSS files
prettier --write "erpnext/**/*.scss"

# Check formatting without changing
prettier --check "erpnext/**/*.scss"
```

### Debugging

#### Enable Developer Mode

```bash
# Enable developer mode
bench --site <sitename> set-config developer_mode 1

# Restart bench
bench restart
```

**Developer mode enables:**
- Detailed error pages
- Auto-reload on code changes
- SQL query logging
- Debug toolbar

#### Python Debugger

```python
# In Python code
import pdb; pdb.set_trace()

# Or use IPython debugger (if installed)
import ipdb; ipdb.set_trace()

# Or use Frappe's debugger
frappe.debug()
```

#### SQL Query Logging

```bash
# Enable SQL logging
bench --site <sitename> set-config logging 2

# View logs
tail -f sites/<sitename>/logs/web.log
```

#### JavaScript Console

```javascript
// In browser console
frappe.get_doc('Sales Invoice', 'SINV-00001')

// Debug form
cur_frm.doc

// Call API method
frappe.call({
    method: 'erpnext.accounts.utils.get_balance_on',
    args: {account: 'Cash - XYZ', date: '2024-01-01'},
    callback: function(r) {
        console.log(r.message);
    }
});
```

---

## 10. Quick Reference

### Module Purpose Matrix

| Module | Primary Purpose | Key DocTypes |
|--------|----------------|--------------|
| **accounts** | Financial accounting | Sales Invoice, Purchase Invoice, Payment Entry, Journal Entry, GL Entry, Payment Ledger Entry |
| **stock** | Inventory management | Item, Stock Entry, Delivery Note, Purchase Receipt, Warehouse, Serial No, Batch |
| **buying** | Procurement | Purchase Order, Supplier, Supplier Quotation, Request for Quotation, Material Request |
| **selling** | Sales operations | Sales Order, Quotation, Customer, Sales Person, Sales Partner |
| **manufacturing** | Production | BOM, Work Order, Production Plan, Workstation, Job Card, Operation |
| **projects** | Project management | Project, Task, Timesheet, Activity Type, Project Type |
| **crm** | Customer relations | Lead, Opportunity, Prospect, Campaign, Email Campaign |
| **assets** | Asset management | Asset, Asset Category, Asset Maintenance, Asset Depreciation Schedule |
| **support** | Customer support | Issue, Warranty Claim, Service Level Agreement, Issue Type |
| **setup** | System configuration | Company, Employee, Territory, Brand, Item Group, UOM, Currency |
| **utilities** | Bulk operations | Rename Tool, Bulk Update, Video |
| **portal** | Customer/Supplier portal | Portal Settings, Homepage, Web Form |
| **maintenance** | Equipment maintenance | Maintenance Schedule, Maintenance Visit |
| **regional** | Country customizations | Country-specific tax, compliance, reports |
| **quality_management** | Quality control | Quality Review, Quality Procedure, Quality Goal |
| **communication** | Marketing | Newsletter, Email Campaign, Campaign |
| **telephony** | Phone integration | Call Log, Incoming Call Settings |
| **subcontracting** | Subcontracting | Subcontracting Order, Subcontracting Receipt |
| **edi** | Electronic Data Interchange | EDI settings and integration |

### Common File Locations

| What | Where |
|------|-------|
| Hooks configuration | `/erpnext/hooks.py` |
| App initialization | `/erpnext/__init__.py` |
| Base controllers | `/erpnext/controllers/` |
| Accounting controller | `/erpnext/controllers/accounts_controller.py` |
| Selling controller | `/erpnext/controllers/selling_controller.py` |
| Buying controller | `/erpnext/controllers/buying_controller.py` |
| Stock controller | `/erpnext/controllers/stock_controller.py` |
| Tax calculation | `/erpnext/controllers/taxes_and_totals.py` |
| Test utilities | `/erpnext/tests/utils.py` |
| Migration patches | `/erpnext/patches/v*/` |
| Static assets | `/erpnext/public/` |
| JavaScript files | `/erpnext/public/js/` |
| CSS/SCSS files | `/erpnext/public/css/` |
| Web templates | `/erpnext/templates/` |
| Regional customizations | `/erpnext/regional/<country>/` |
| Accounting utils | `/erpnext/accounts/utils.py` |
| Party management | `/erpnext/accounts/party.py` |
| General ledger | `/erpnext/accounts/general_ledger.py` |
| Stock utils | `/erpnext/stock/utils.py` |
| Stock ledger | `/erpnext/stock/stock_ledger.py` |
| Setup utils | `/erpnext/setup/utils.py` |

### Important Frappe Imports

```python
# Core Frappe
import frappe
from frappe import _  # Translation function
from frappe import msgprint, throw  # User messages
from frappe.model.document import Document
from frappe.utils import (
    flt,  # Float conversion
    cint,  # Integer conversion
    getdate,  # Date conversion
    nowdate,  # Current date
    add_days,  # Date arithmetic
    get_datetime,  # DateTime conversion
    formatdate,  # Format date for display
    money_in_words,  # Number to words
    fmt_money  # Format currency
)
from frappe.query_builder import DocType  # Query builder
```

### Common Decorators

```python
@frappe.whitelist()  # Make method callable from client API
@frappe.validate_and_sanitize_search_inputs  # For search queries
@erpnext.allow_regional  # Allow regional override
```

### Key ERPNext Imports

```python
from erpnext import get_default_company, get_default_currency
from erpnext.accounts.utils import get_fiscal_year, get_account_currency
from erpnext.accounts.party import get_party_account
from erpnext.stock.utils import get_latest_stock_qty
from erpnext.controllers.accounts_controller import AccountsController
from erpnext.controllers.selling_controller import SellingController
from erpnext.controllers.buying_controller import BuyingController
from erpnext.controllers.stock_controller import StockController
```

### Common Frappe Methods

```python
# Document operations
frappe.get_doc(doctype, name)  # Get single document
frappe.get_all(doctype, filters, fields)  # Get list of documents
frappe.get_list(doctype, filters, fields)  # Get list (with permissions)
frappe.db.get_value(doctype, name, fieldname)  # Get single value
frappe.db.set_value(doctype, name, fieldname, value)  # Set value
frappe.db.exists(doctype, name)  # Check if document exists

# Database queries
frappe.db.sql(query, values, as_dict=True)  # Raw SQL query
frappe.db.get_all(doctype, filters, fields)  # Query without permissions

# Messages
frappe.msgprint(message)  # Info message
frappe.throw(message)  # Error message with exception
frappe.log_error(message, title)  # Log error

# Current context
frappe.session.user  # Current user
frappe.db  # Database connection
frappe.local.site  # Current site
```

### Keyboard Shortcuts (Frappe UI)

- **Ctrl+K** - Quick search / AwesomeBar
- **Ctrl+G** - Go to list view
- **Ctrl+S** - Save document
- **Ctrl+Shift+S** - Save and submit
- **Ctrl+P** - Print
- **Ctrl+E** - Edit document
- **Ctrl+B** - Toggle sidebar
- **/** - Focus search

---

## Additional Resources

### Official Documentation

- **ERPNext Docs:** https://docs.erpnext.com
- **Frappe Framework Docs:** https://frappeframework.com/docs
- **ERPNext GitHub:** https://github.com/frappe/erpnext
- **Frappe GitHub:** https://github.com/frappe/frappe

### Community

- **Forum:** https://discuss.frappe.io
- **Telegram:** ERPNext and Frappe communities
- **Stack Overflow:** `erpnext` and `frappe` tags

### Contributing

- **Contribution Guidelines:** `.github/CONTRIBUTING.md`
- **Code of Conduct:** `CODE_OF_CONDUCT.md`
- **Security Policy:** `SECURITY.md`

---

**Last Updated:** 2024-01-31
**ERPNext Version:** 17.0.0-dev
**Maintained by:** ERPNext Development Team
