# Buying Module - Database Table Relations

## Overview

This document provides a comprehensive analysis of the Primary Key (PK) and Foreign Key (FK) relationships for the main tables in the ERPNext Buying module. It focuses on the core procurement workflow and data relationships that maintain referential integrity across the purchasing process.

## Core Business Context

The Buying module implements the **Procure-to-Pay** business workflow:
- **Material Request** → **Request for Quotation** → **Supplier Quotation** → **Purchase Order** → **Purchase Receipt** → **Purchase Invoice**

This workflow ensures proper procurement control, three-way matching, and audit trails for business compliance.

## Database Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           BUYING MODULE TABLE RELATIONS                      │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────┐
│      Supplier        │
├──────────────────────┤
│ name (PK)           │
│ supplier_name       │
│ supplier_group (FK) │
│ country (FK)        │
│ default_currency(FK)│
│ payment_terms (FK)  │
└──────────────────────┘
           │
           │ Referenced by all purchase documents
           ▼
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│  Material Request    │    │ Request for Quotation│    │ Supplier Quotation   │
├──────────────────────┤    ├──────────────────────┤    ├──────────────────────┤
│ name (PK)           │    │ name (PK)           │    │ name (PK)           │
│ company (FK)        │    │ company (FK)        │    │ supplier (FK)       │
│ cost_center (FK)    │    │ opportunity (FK)    │    │ company (FK)        │
│ project (FK)        │    │ incoterm (FK)       │    │ currency (FK)       │
│ requested_by (FK)   │    │ tc_name (FK)        │    │ opportunity (FK)    │
└──────────────────────┘    └──────────────────────┘    │ incoterm (FK)       │
           │                           │                 └──────────────────────┘
           │                           │                            │
           └─────────────┐             │                            │
                         ▼             ▼                            │
                 ┌──────────────────────┐                          │
                 │   Purchase Order     │◄─────────────────────────┘
                 ├──────────────────────┤
                 │ name (PK)           │
                 │ supplier (FK)       │
                 │ company (FK)        │
                 │ currency (FK)       │
                 │ cost_center (FK)    │
                 │ project (FK)        │
                 │ ref_sq (FK)         │ ← References Supplier Quotation
                 │ set_warehouse (FK)  │
                 │ payment_terms (FK)  │
                 │ incoterm (FK)       │
                 └──────────────────────┘
                           │
                           │ Referenced by receipts and invoices
                 ┌─────────┴─────────┐
                 ▼                   ▼
        ┌──────────────────────┐    ┌──────────────────────┐
        │  Purchase Receipt    │    │  Purchase Invoice    │
        ├──────────────────────┤    ├──────────────────────┤
        │ name (PK)           │    │ name (PK)           │
        │ supplier (FK)       │    │ supplier (FK)       │
        │ company (FK)        │    │ company (FK)        │
        │ currency (FK)       │    │ currency (FK)       │
        │ cost_center (FK)    │    │ cost_center (FK)    │
        │ project (FK)        │    │ project (FK)        │
        │ set_warehouse (FK)  │    │ credit_to (FK)      │
        │ return_against (FK) │    │ return_against (FK) │
        └──────────────────────┘    └──────────────────────┘
```

## Parent-Child Relationships

Each main transaction document has corresponding line item tables that store detailed information:

### Purchase Order Structure
```
Purchase Order (Parent)
├── Purchase Order Item (Child)
│   ├── parent (FK) → Purchase Order.name
│   ├── item_code (FK) → Item.name
│   ├── warehouse (FK) → Warehouse.name
│   ├── uom (FK) → UOM.name
│   └── material_request (FK) → Material Request.name
│
└── Purchase Taxes and Charges (Child)
    ├── parent (FK) → Purchase Order.name
    ├── account_head (FK) → Account.name
    └── cost_center (FK) → Cost Center.name
```

### Complete Child Table Structure

| Parent Document | Child Table | Key Relationships |
|----------------|-------------|------------------|
| **Material Request** | Material Request Item | parent (FK), item_code (FK), warehouse (FK) |
| **Request for Quotation** | RFQ Item | parent (FK), item_code (FK), warehouse (FK) |
| | RFQ Supplier | parent (FK), supplier (FK) |
| **Supplier Quotation** | Supplier Quotation Item | parent (FK), item_code (FK), warehouse (FK) |
| **Purchase Order** | Purchase Order Item | parent (FK), item_code (FK), warehouse (FK) |
| | Purchase Taxes and Charges | parent (FK), account_head (FK) |
| **Purchase Receipt** | Purchase Receipt Item | parent (FK), item_code (FK), warehouse (FK) |
| | Purchase Taxes and Charges | parent (FK), account_head (FK) |
| **Purchase Invoice** | Purchase Invoice Item | parent (FK), item_code (FK), warehouse (FK) |
| | Purchase Taxes and Charges | parent (FK), account_head (FK) |

## Master Data Dependencies

All buying documents reference these core master data tables:

### Financial Masters
| Table | Primary Key | Business Purpose |
|-------|-------------|------------------|
| **Company** | name | Multi-company data segregation |
| **Currency** | name | Multi-currency transaction support |
| **Account** | name | General ledger integration |
| **Cost Center** | name | Departmental cost allocation |
| **Payment Terms Template** | name | Standard payment conditions |

### Operational Masters
| Table | Primary Key | Business Purpose |
|-------|-------------|------------------|
| **Warehouse** | name | Inventory location management |
| **Item** | name | Product/service catalog |
| **UOM** | name | Unit of measure standardization |
| **Price List** | name | Pricing strategy management |
| **Tax Category** | name | Tax calculation rules |

### Communication Masters
| Table | Primary Key | Business Purpose |
|-------|-------------|------------------|
| **Address** | name | Supplier and shipping locations |
| **Contact** | name | Communication points |
| **Incoterm** | name | International trade terms |
| **Terms and Conditions** | name | Legal terms templates |

## Business Validation Rules

### Referential Integrity Constraints
1. **Supplier Validation**: All purchase documents must reference valid suppliers
2. **Company Isolation**: Documents are segregated by company for multi-tenant operations
3. **Currency Consistency**: Transaction currency must align with supplier's default currency
4. **Warehouse Validation**: Only company-owned warehouses can be selected

### Document Flow Constraints
1. **Material Request** → Can generate **Request for Quotation**
2. **Supplier Quotation** → Can be referenced in **Purchase Order** (ref_sq field)
3. **Purchase Order** → Required for **Purchase Receipt** and **Purchase Invoice**
4. **Purchase Receipt** → Enables goods-received-not-invoiced tracking

### Status-Driven Workflow
```
Draft (0) → Submitted (1) → Cancelled (2)
    ↓           ↓              ↓
  Editable   Immutable    Reversed
```

## Multi-Currency Design Pattern

The buying module supports international procurement through a sophisticated multi-currency design:

### Currency Fields Pattern
```sql
-- Every transaction document includes:
currency (FK)               -- Transaction currency
conversion_rate            -- Exchange rate to company currency
base_total                -- Amount in company currency  
total                     -- Amount in transaction currency
base_grand_total          -- Final total in company currency
grand_total              -- Final total in transaction currency
```

### Exchange Rate Management
- Real-time currency conversion using **Currency Exchange** table
- Separate buying and selling rates for financial accuracy
- Historical rate tracking for audit compliance

## Performance Optimization Patterns

### Strategic Indexing
```sql
-- Critical performance indexes for buying module
CREATE INDEX idx_supplier_posting ON `tabPurchase Order` (supplier, posting_date);
CREATE INDEX idx_company_status ON `tabPurchase Order` (company, status, posting_date);
CREATE INDEX idx_item_warehouse ON `tabPurchase Order Item` (item_code, warehouse);
CREATE INDEX idx_parent_idx ON `tabPurchase Order Item` (parent, idx);
```

### Query Optimization Guidelines
1. **Compound Indexes**: Use multi-column indexes for common filter combinations
2. **Date Range Queries**: Always include company and date filters
3. **Status Filtering**: Include docstatus in reporting queries
4. **Parent-Child Joins**: Use proper join conditions with parenttype verification

## Integration Points

### Accounts Module Integration
- **GL Entry** generation for financial postings
- **Payment Ledger Entry** for outstanding management
- **Tax Calculation** through integrated tax engines

### Stock Module Integration
- **Stock Ledger Entry** for inventory movements
- **Bin** updates for real-time stock balances
- **Serial/Batch** tracking for traced items

### Projects Module Integration
- **Project** allocation for cost tracking
- **Timesheet** integration for service procurement
- **Budget** control and variance analysis

## Business Intelligence Support

### Key Metrics Tracking
The database design supports comprehensive procurement analytics:

1. **Supplier Performance**: Lead time, quality, pricing trends
2. **Category Analysis**: Spend by item group, supplier group
3. **Process Efficiency**: Cycle time from request to receipt
4. **Financial Control**: Budget variance, exchange rate impact

### Reporting Relationships
The foreign key structure enables drill-down reporting from summary to transaction level while maintaining data integrity and audit trails.

---

*This database design ensures robust procurement operations with proper business controls, multi-company support, and comprehensive audit capabilities while maintaining high performance for enterprise-scale operations.*