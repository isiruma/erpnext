# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ERPNext is a comprehensive open-source ERP system built on the Frappe Framework. It's a full-stack application using Python for backend logic and JavaScript/Vue.js for frontend interfaces, with MariaDB/MySQL as the primary database.

## Development Commands

### Environment Setup
```bash
# Setup development environment
bench init --frappe-path ~/frappe frappe-bench
cd frappe-bench
bench new-site erpnext.localhost
bench get-app https://github.com/frappe/erpnext
bench --site erpnext.localhost install-app erpnext

# Start development server
bench start

# Access application: http://erpnext.localhost:8000/app
```

### Testing Commands
```bash
# Run all tests for ERPNext
bench --site [site-name] run-tests --app erpnext

# Run specific test module
bench --site [site-name] run-tests --app erpnext --module erpnext.accounts.doctype.sales_invoice.test_sales_invoice

# Run specific doctype tests
bench --site [site-name] run-tests --app erpnext --doctype "Sales Invoice"

# Run parallel tests (CI approach)
bench --site [site-name] run-parallel-tests --app erpnext --total-builds 4 --build-number 1
```

### Code Quality
```bash
# Run linter (Ruff)
ruff check .
ruff format .

# Pre-commit hooks (auto-installed)
pre-commit run --all-files
```

## Architecture Overview

### Technology Stack
- **Backend**: Python 3.10+ with Frappe Framework
- **Frontend**: JavaScript/Vue.js via Frappe UI
- **Database**: MariaDB (primary), PostgreSQL (supported)
- **Dependencies**: Core dependencies include pycountry, Unidecode, barcodenumber, rapidfuzz, holidays
- **Build System**: flit_core for Python packaging

### Module Structure
ERPNext follows a modular architecture with 21 core modules:

**Core Business Modules:**
- `accounts/` - Financial management and accounting
- `selling/` - Sales orders, customers, quotations
- `buying/` - Purchase orders, suppliers, procurement
- `stock/` - Inventory management and warehousing
- `manufacturing/` - Production planning and work orders
- `projects/` - Project management and timesheets
- `crm/` - Customer relationship management
- `support/` - Customer support and ticketing

**System Modules:**
- `setup/` - System configuration and company setup
- `utilities/` - Common utility functions
- `regional/` - Country-specific localizations
- `erpnext_integrations/` - Third-party service integrations

### DocType Architecture Pattern
Each business entity follows the DocType pattern:
```
erpnext/{module}/doctype/{doctype_name}/
├── {doctype_name}.json      # DocType definition (fields, permissions, etc.)
├── {doctype_name}.py        # Server-side controller logic
├── {doctype_name}.js        # Client-side scripting
├── test_{doctype_name}.py   # Unit tests
└── test_records.json        # Test data fixtures
```

### Controller Inheritance Hierarchy
Controllers follow a clear inheritance pattern:
- `Document` (Frappe base)
  - `AccountsController` (erpnext.controllers.accounts_controller)
    - `BuyingController` (erpnext.controllers.buying_controller)
    - `SellingController` (erpnext.controllers.selling_controller)
    - `StockController` (erpnext.controllers.stock_controller)

### Key Architectural Components

**Hooks System** (`erpnext/hooks.py`):
- Event-driven architecture for customizations
- Document lifecycle hooks (before_save, after_insert, etc.)
- Scheduled jobs and background tasks
- Custom field and permission overrides

**Regional Customizations** (`erpnext/regional/`):
- Country-specific tax calculations
- Localized chart of accounts
- Regional compliance features
- Address format templates

**Integration Layer** (`erpnext/erpnext_integrations/`):
- Payment gateway integrations
- External service APIs (Google Maps, Plaid, YouTube)
- EDI (Electronic Data Interchange) support

## Code Style and Standards

### Python Code Style
- **Formatter**: Ruff with 110 character line length
- **Target**: Python 3.10+
- **Indentation**: Tabs (not spaces)
- **Quotes**: Double quotes for strings
- **Import Organization**: Automatic sorting with Ruff

### Key Patterns to Follow

**DocType Controller Pattern**:
```python
class SalesInvoice(SellingController):
    def validate(self):
        super().validate()
        # Custom validation logic
    
    def on_submit(self):
        # Post-submission logic
        self.update_status()
    
    def on_cancel(self):
        # Cancellation logic
        self.update_status("Cancelled")
```

**Import Organization**:
```python
# Frappe imports first
import frappe
from frappe import _
from frappe.utils import flt, cint

# ERPNext imports second
import erpnext
from erpnext.accounts.utils import get_account_currency
```

**Testing Pattern**:
```python
class TestSalesInvoice(ERPNextTestSuite):
    def setUp(self):
        # Test setup
        
    def test_sales_invoice_calculation(self):
        # Test implementation
        si = make_sales_invoice()
        self.assertEqual(si.grand_total, 100)
```

## Important Development Notes

### Database Considerations
- Primary database is MariaDB; PostgreSQL support available
- All database operations go through Frappe ORM
- Multi-tenancy support via site-based architecture

### Performance Guidelines
- Use `frappe.db.get_value()` for single field retrieval
- Prefer `frappe.get_all()` over `frappe.get_list()` for better performance
- Implement proper indexing for custom fields in DocType JSON

### Testing Requirements
- Every DocType must have comprehensive unit tests
- Use test fixtures from `test_records.json`
- Tests run in parallel across 4 containers in CI
- Coverage tracking enabled for all non-PR builds

### Security Practices
- All user inputs are automatically sanitized by Frappe
- Use `frappe.throw()` for user-facing errors
- Implement proper role-based permissions in DocType JSON
- Never commit API keys or sensitive configuration

### Internationalization
- All user-facing strings must use `frappe._()` for translation
- Translation files managed via Crowdin
- Regional customizations go in `erpnext/regional/`

## Comprehensive Analysis Documentation

This repository now contains detailed analysis documentation for building new ERP systems:

### Available Analysis Documents
1. **`erpnext-business-analysis.md`** - Complete business logic and workflow analysis
   - 8 core business modules (Accounts, Selling, Buying, Stock, Manufacturing, CRM, Projects, Assets)
   - Key business workflows (Order-to-Cash, Procure-to-Pay, Plan-to-Produce)
   - Business rules, validation patterns, and process automation
   - Domain-specific implementations for Manufacturing, Retail, and Services

2. **`erpnext-technical-architecture.md`** - Comprehensive technical architecture guide
   - Document-centric architecture and MVC implementation
   - Controller inheritance hierarchy and event-driven patterns
   - Data layer architecture with ORM patterns and caching strategies
   - API design, security architecture, and extension mechanisms
   - Performance optimization and deployment patterns

3. **`erpnext-ui-patterns.md`** - UI/UX design patterns and frontend architecture
   - Frontend technology stack and component-based architecture
   - Workspace design with JSON-based configuration
   - Form design patterns, validation, and responsive design
   - Mobile optimization and accessibility features
   - User experience patterns and progressive disclosure

4. **`erpnext-database-design.md`** - Database design and relationship analysis
   - Metadata-driven schema with JSON definitions
   - Core entity relationships and hierarchical data models
   - Financial data model with double-entry accounting
   - Inventory management with stock ledger and real-time balances
   - Multi-tenancy design and performance optimization
   - Audit trails and data integrity patterns

5. **`erp-development-roadmap.md`** - Complete implementation roadmap
   - 4-phase development strategy (Foundation → Core → Advanced → Enterprise)
   - Technology stack recommendations with decision matrices
   - Architecture decisions and implementation best practices
   - Code organization, testing strategies, and security practices
   - Timeline estimates and MVP approach

### Using This Analysis for ERP Development

**IMPORTANT: User prefers learning business logic and workflows over technical implementation details**

**For Business Logic Understanding (PRIMARY FOCUS):**
- Study `erpnext-business-analysis.md` for comprehensive ERP workflows and business requirements
- Focus on business rules, validation patterns, and decision logic
- Understand process automation strategies and business efficiency improvements
- Learn domain-specific business patterns (Manufacturing, Retail, Services)

**For Business Process Flow (SECONDARY FOCUS):**
- Reference the 3 key workflows: Order-to-Cash, Procure-to-Pay, Plan-to-Produce
- Understand status transitions and business state management
- Learn percentage completion tracking and milestone-based processes
- Study approval workflows and business validation rules

**For User Experience and Business Workflows:**
- Reference `erpnext-ui-patterns.md` for understanding user journey patterns
- Focus on workspace design that supports business processes
- Learn progressive disclosure patterns for complex business workflows
- Understand form design that matches business logic flow

**For Data Relationships and Business Entities:**
- Study `erpnext-database-design.md` for business entity relationships
- Focus on how business data flows between different modules
- Understand multi-company business scenarios and data segregation
- Learn audit trail patterns for business compliance

**For Business-Driven Project Planning:**
- Use `erp-development-roadmap.md` for business-focused implementation approach
- Follow business module prioritization (CRM → Sales → Inventory → Accounting)
- Reference business value delivery in each development phase

### Key Business Insights for New ERP Development

**Core Business Principles:**
- Document-based business processes that mirror real-world business documents
- Event-driven business workflows that automate manual processes
- Multi-company business scenarios for group organizations
- Flexible business rules that adapt to different industry requirements

**Essential Business Logic Patterns:**
- Status-driven workflows with clear business milestones
- Percentage completion tracking for business transparency
- Double-entry accounting principles for financial accuracy
- Real-time business data for informed decision making
- Role-based business access reflecting organizational hierarchy

**Business Process Automation:**
- Automated reorder point management for inventory optimization
- Payment reminder workflows for cash flow management
- Recurring transaction automation for subscription businesses
- Three-way matching for procurement control (PO → Receipt → Invoice)
- Credit limit enforcement for risk management

**Business User Experience Principles:**
- Progressive disclosure that matches business complexity levels
- Workspace design organized by business functions
- Form flows that follow natural business process sequences
- Dashboard design focused on business KPIs and actionable insights
- Mobile access for field-based business operations

**Business Compliance and Control:**
- Audit trail maintenance for business compliance requirements
- Approval workflows that enforce business authorization policies
- Data segregation for multi-company business scenarios
- Version control for critical business document changes

This comprehensive analysis provides everything needed to build a modern, scalable ERP system using proven patterns from ERPNext's successful implementation.