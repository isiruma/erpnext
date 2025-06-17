# ERPNext Database Design and Relationships Analysis

This document provides a comprehensive analysis of ERPNext's database design patterns, entity relationships, and data modeling strategies for building scalable ERP systems.

## Table of Contents
1. [Database Schema Architecture](#database-schema-architecture)
2. [Core Entity Relationships](#core-entity-relationships)
3. [Financial Data Model](#financial-data-model)
4. [Inventory Management Model](#inventory-management-model)
5. [Multi-Tenancy Design](#multi-tenancy-design)
6. [Audit and Versioning Patterns](#audit-and-versioning-patterns)
7. [Performance Optimization](#performance-optimization)
8. [Data Integrity Patterns](#data-integrity-patterns)

## Database Schema Architecture

### Metadata-Driven Schema

ERPNext uses a metadata-driven approach where table structures are defined in JSON files and automatically managed by the framework.

**DocType Schema Definition**:
```json
{
    "name": "Customer",
    "naming_series": ["CUST-.YYYY.-"],
    "autoname": "naming_series:",
    "fields": [
        {
            "fieldname": "customer_name",
            "fieldtype": "Data",
            "label": "Customer Name",
            "reqd": 1,
            "unique": 1,
            "search_index": 1
        },
        {
            "fieldname": "customer_group",
            "fieldtype": "Link",
            "options": "Customer Group",
            "reqd": 1,
            "default": "Individual"
        },
        {
            "fieldname": "territory",
            "fieldtype": "Link", 
            "options": "Territory",
            "reqd": 1
        }
    ]
}
```

**Generated Database Schema**:
```sql
-- Auto-generated table structure
CREATE TABLE `tabCustomer` (
    `name` varchar(140) NOT NULL PRIMARY KEY,
    `creation` datetime(6) DEFAULT NULL,
    `modified` datetime(6) DEFAULT NULL,
    `modified_by` varchar(140) DEFAULT NULL,
    `owner` varchar(140) DEFAULT NULL,
    `docstatus` int(1) NOT NULL DEFAULT 0,
    `customer_name` varchar(140) DEFAULT NULL,
    `customer_group` varchar(140) DEFAULT NULL,
    `territory` varchar(140) DEFAULT NULL,
    `_user_tags` text,
    `_comments` text,
    `_assign` text,
    `_liked_by` text,
    
    INDEX `modified` (`modified`),
    INDEX `customer_group` (`customer_group`),
    INDEX `territory` (`territory`),
    UNIQUE KEY `customer_name` (`customer_name`)
);
```

### Standard Field Patterns

**Automatic Fields** (included in every table):
```python
# Core fields added to every DocType
STANDARD_FIELDS = {
    "name": "Primary key (varchar(140))",
    "creation": "Creation timestamp",
    "modified": "Last modification timestamp", 
    "modified_by": "User who last modified",
    "owner": "User who created the record",
    "docstatus": "Document status (0=Draft, 1=Submitted, 2=Cancelled)",
    "_user_tags": "User-defined tags",
    "_comments": "Comments and timeline",
    "_assign": "Document assignment",
    "_liked_by": "User likes tracking"
}
```

### Naming Conventions

**Table Naming**:
- All tables prefixed with `tab`: `tabCustomer`, `tabSales Invoice`
- DocType names converted to table names: "Sales Invoice" → `tabSales Invoice`
- Child tables: `tabSales Invoice Item`

**Field Naming**:
```python
# Field naming patterns
{
    "customer_name": "snake_case for field names",
    "grand_total": "Calculated fields use descriptive names",
    "per_billed": "Percentage fields prefixed with 'per_'",
    "is_return": "Boolean fields prefixed with 'is_'",
    "has_batch_no": "Boolean fields using 'has_' for capabilities"
}
```

## Core Entity Relationships

### Master Data Relationships

**Customer Entity Model**:
```sql
-- Customer master with hierarchical relationships
CREATE TABLE `tabCustomer` (
    `name` varchar(140) PRIMARY KEY,
    `customer_name` varchar(140) UNIQUE,
    `customer_group` varchar(140),  -- Link to Customer Group
    `territory` varchar(140),       -- Link to Territory
    `default_currency` varchar(3),  -- Link to Currency
    `default_price_list` varchar(140), -- Link to Price List
    `payment_terms` varchar(140),   -- Link to Payment Terms Template
    `credit_limit` decimal(18,6),
    `is_frozen` tinyint(1) DEFAULT 0,
    
    FOREIGN KEY (`customer_group`) REFERENCES `tabCustomer Group`(`name`),
    FOREIGN KEY (`territory`) REFERENCES `tabTerritory`(`name`),
    FOREIGN KEY (`default_currency`) REFERENCES `tabCurrency`(`name`)
);

-- Customer-Company specific settings
CREATE TABLE `tabParty Account` (
    `name` varchar(140) PRIMARY KEY,
    `parent` varchar(140),          -- Customer name
    `company` varchar(140),         -- Company name
    `account` varchar(140),         -- Receivable account
    `advance_account` varchar(140), -- Advance account
    
    FOREIGN KEY (`parent`) REFERENCES `tabCustomer`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    FOREIGN KEY (`account`) REFERENCES `tabAccount`(`name`)
);

-- Customer credit limits per company
CREATE TABLE `tabCustomer Credit Limit` (
    `name` varchar(140) PRIMARY KEY,
    `parent` varchar(140),      -- Customer name
    `company` varchar(140),     -- Company name
    `credit_limit` decimal(18,6),
    `bypass_credit_limit_check` tinyint(1),
    
    FOREIGN KEY (`parent`) REFERENCES `tabCustomer`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`)
);
```

**Item Entity Model**:
```sql
-- Item master with rich metadata
CREATE TABLE `tabItem` (
    `name` varchar(140) PRIMARY KEY,
    `item_code` varchar(140) UNIQUE,
    `item_name` varchar(140),
    `item_group` varchar(140),      -- Hierarchical grouping
    `stock_uom` varchar(140),       -- Base unit of measure
    `is_stock_item` tinyint(1),
    `is_sales_item` tinyint(1), 
    `is_purchase_item` tinyint(1),
    `has_batch_no` tinyint(1),
    `has_serial_no` tinyint(1),
    `has_variants` tinyint(1),
    `variant_of` varchar(140),      -- Parent item for variants
    `brand` varchar(140),
    `description` longtext,
    `image` text,
    `weight_per_unit` decimal(18,6),
    `weight_uom` varchar(140),
    
    FOREIGN KEY (`item_group`) REFERENCES `tabItem Group`(`name`),
    FOREIGN KEY (`stock_uom`) REFERENCES `tabUOM`(`name`),
    FOREIGN KEY (`brand`) REFERENCES `tabBrand`(`name`)
);

-- Item-Company specific defaults
CREATE TABLE `tabItem Default` (
    `name` varchar(140) PRIMARY KEY,
    `parent` varchar(140),          -- Item code
    `company` varchar(140),         -- Company name
    `default_warehouse` varchar(140),
    `expense_account` varchar(140),
    `income_account` varchar(140),
    `cost_center` varchar(140),
    `buying_cost_center` varchar(140),
    `selling_cost_center` varchar(140),
    
    FOREIGN KEY (`parent`) REFERENCES `tabItem`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    FOREIGN KEY (`default_warehouse`) REFERENCES `tabWarehouse`(`name`)
);
```

### Transaction Relationships

**Sales Invoice Structure**:
```sql
-- Sales Invoice (Parent Document)
CREATE TABLE `tabSales Invoice` (
    `name` varchar(140) PRIMARY KEY,
    `customer` varchar(140) NOT NULL,
    `customer_name` varchar(140),
    `posting_date` date,
    `due_date` date,
    `company` varchar(140),
    `currency` varchar(3),
    `conversion_rate` decimal(18,9) DEFAULT 1.0,
    `price_list` varchar(140),
    `taxes_and_charges` varchar(140),
    `payment_terms_template` varchar(140),
    
    -- Totals
    `total` decimal(18,6),
    `total_taxes_and_charges` decimal(18,6),
    `grand_total` decimal(18,6),
    `outstanding_amount` decimal(18,6),
    `paid_amount` decimal(18,6),
    
    -- Status tracking
    `status` varchar(140),
    `per_billed` decimal(18,6) DEFAULT 0.0,
    `per_delivered` decimal(18,6) DEFAULT 0.0,
    
    -- References
    `project` varchar(140),
    `cost_center` varchar(140),
    
    FOREIGN KEY (`customer`) REFERENCES `tabCustomer`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    FOREIGN KEY (`currency`) REFERENCES `tabCurrency`(`name`),
    
    INDEX `posting_date` (`posting_date`),
    INDEX `due_date` (`due_date`),
    INDEX `status` (`status`),
    INDEX `outstanding_amount` (`outstanding_amount`)
);

-- Sales Invoice Items (Child Table)
CREATE TABLE `tabSales Invoice Item` (
    `name` varchar(140) PRIMARY KEY,
    `parent` varchar(140),          -- Sales Invoice name
    `parenttype` varchar(140) DEFAULT 'Sales Invoice',
    `parentfield` varchar(140) DEFAULT 'items',
    `idx` int(8),                   -- Sort order
    
    `item_code` varchar(140),
    `item_name` varchar(140),
    `description` longtext,
    `qty` decimal(18,6),
    `uom` varchar(140),
    `rate` decimal(18,6),
    `amount` decimal(18,6),
    `warehouse` varchar(140),
    `cost_center` varchar(140),
    `income_account` varchar(140),
    
    -- Pricing details
    `price_list_rate` decimal(18,6),
    `discount_percentage` decimal(18,6),
    `discount_amount` decimal(18,6),
    
    -- Reference tracking
    `sales_order` varchar(140),
    `so_detail` varchar(140),       -- Sales Order Item reference
    `delivery_note` varchar(140),
    `dn_detail` varchar(140),       -- Delivery Note Item reference
    
    FOREIGN KEY (`parent`) REFERENCES `tabSales Invoice`(`name`) ON DELETE CASCADE,
    FOREIGN KEY (`item_code`) REFERENCES `tabItem`(`name`),
    FOREIGN KEY (`warehouse`) REFERENCES `tabWarehouse`(`name`)
);
```

### Hierarchical Data Model

**Tree Structure Implementation**:
```sql
-- Account Chart (Nested Sets Model)
CREATE TABLE `tabAccount` (
    `name` varchar(140) PRIMARY KEY,
    `account_name` varchar(140),
    `company` varchar(140),
    `parent_account` varchar(140),  -- Self-reference
    `account_type` varchar(140),
    `root_type` varchar(140),       -- Asset, Liability, Income, Expense, Equity
    `account_currency` varchar(3),
    `is_group` tinyint(1),         -- Parent vs Leaf node
    
    -- Nested sets model for hierarchical queries
    `lft` int(8),                  -- Left boundary
    `rgt` int(8),                  -- Right boundary
    
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    FOREIGN KEY (`parent_account`) REFERENCES `tabAccount`(`name`),
    
    INDEX `lft_rgt` (`lft`, `rgt`),
    INDEX `company_account_type` (`company`, `account_type`)
);

-- Efficient hierarchical queries using nested sets
-- Get all child accounts of a parent:
SELECT child.name, child.account_name
FROM `tabAccount` parent, `tabAccount` child
WHERE child.lft BETWEEN parent.lft AND parent.rgt
  AND parent.name = 'Assets - Company'
  AND child.company = parent.company;
```

## Financial Data Model

### Double-Entry Accounting System

**General Ledger Entry Model**:
```sql
-- General Ledger (Core accounting table)
CREATE TABLE `tabGL Entry` (
    `name` varchar(140) PRIMARY KEY,
    `posting_date` date NOT NULL,
    `transaction_date` date,
    `account` varchar(140) NOT NULL,
    `account_currency` varchar(3),
    `company` varchar(140) NOT NULL,
    
    -- Double-entry amounts
    `debit` decimal(18,6) DEFAULT 0.0,
    `credit` decimal(18,6) DEFAULT 0.0,
    `debit_in_account_currency` decimal(18,6) DEFAULT 0.0,
    `credit_in_account_currency` decimal(18,6) DEFAULT 0.0,
    
    -- Source document reference
    `voucher_type` varchar(140),
    `voucher_no` varchar(140),
    `voucher_detail_no` varchar(140),
    
    -- Party information
    `party_type` varchar(140),      -- Customer, Supplier, Employee
    `party` varchar(140),           -- Party name
    
    -- Analytical dimensions
    `cost_center` varchar(140),
    `project` varchar(140),
    `finance_book` varchar(140),
    
    -- Additional metadata
    `against` text,                 -- Comma-separated opposite accounts
    `against_voucher_type` varchar(140),
    `against_voucher` varchar(140),
    `remarks` text,
    `is_cancelled` tinyint(1) DEFAULT 0,
    
    FOREIGN KEY (`account`) REFERENCES `tabAccount`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    
    -- Performance indexes
    INDEX `posting_date` (`posting_date`),
    INDEX `account_posting_date` (`account`, `posting_date`),
    INDEX `voucher_no` (`voucher_type`, `voucher_no`),
    INDEX `party` (`party_type`, `party`, `posting_date`),
    INDEX `cost_center` (`cost_center`, `posting_date`),
    INDEX `project` (`project`, `posting_date`)
);
```

### Payment Tracking System

**Modern Payment Ledger** (New approach for outstanding management):
```sql
-- Payment Ledger Entry (Replaces legacy GL-based outstanding tracking)
CREATE TABLE `tabPayment Ledger Entry` (
    `name` varchar(140) PRIMARY KEY,
    `company` varchar(140) NOT NULL,
    `posting_date` date NOT NULL,
    `account` varchar(140) NOT NULL,        -- Party account
    `account_currency` varchar(3),
    
    -- Party information
    `party_type` varchar(140) NOT NULL,
    `party` varchar(140) NOT NULL,
    
    -- Amount tracking
    `amount` decimal(18,6) NOT NULL,        -- In account currency
    `amount_in_company_currency` decimal(18,6),
    
    -- Outstanding tracking
    `amount_type` varchar(140),             -- Receivable, Payable
    `due_date` date,
    
    -- Source document
    `voucher_type` varchar(140) NOT NULL,
    `voucher_no` varchar(140) NOT NULL,
    `voucher_detail_no` varchar(140),
    
    -- Payment allocation
    `against_voucher_type` varchar(140),
    `against_voucher_no` varchar(140),
    
    -- Dimensions
    `cost_center` varchar(140),
    `finance_book` varchar(140),
    
    FOREIGN KEY (`account`) REFERENCES `tabAccount`(`name`),
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`),
    
    INDEX `party_outstanding` (`party_type`, `party`, `posting_date`),
    INDEX `voucher_ref` (`voucher_type`, `voucher_no`),
    INDEX `due_date` (`due_date`),
    INDEX `amount_type` (`amount_type`, `posting_date`)
);
```

### Multi-Currency Support

**Currency Exchange Management**:
```sql
-- Currency Exchange Rates
CREATE TABLE `tabCurrency Exchange` (
    `name` varchar(140) PRIMARY KEY,
    `date` date NOT NULL,
    `from_currency` varchar(3) NOT NULL,
    `to_currency` varchar(3) NOT NULL,
    `exchange_rate` decimal(18,9) NOT NULL,
    `for_buying` decimal(18,9),     -- Different rates for buying/selling
    `for_selling` decimal(18,9),
    
    UNIQUE KEY `unique_exchange` (`date`, `from_currency`, `to_currency`),
    
    FOREIGN KEY (`from_currency`) REFERENCES `tabCurrency`(`name`),
    FOREIGN KEY (`to_currency`) REFERENCES `tabCurrency`(`name`)
);

-- Multi-currency transaction pattern
CREATE TABLE `tabSales Invoice` (
    -- ... other fields ...
    `currency` varchar(3) NOT NULL,
    `conversion_rate` decimal(18,9) DEFAULT 1.0,
    
    -- Amounts in transaction currency
    `total` decimal(18,6),
    `grand_total` decimal(18,6),
    
    -- Amounts in company currency
    `base_total` decimal(18,6),
    `base_grand_total` decimal(18,6),
    
    FOREIGN KEY (`currency`) REFERENCES `tabCurrency`(`name`)
);
```

## Inventory Management Model

### Stock Ledger System

**Stock Ledger Entry** (Core inventory tracking):
```sql
-- Stock Ledger Entry (All inventory movements)
CREATE TABLE `tabStock Ledger Entry` (
    `name` varchar(140) PRIMARY KEY,
    `item_code` varchar(140) NOT NULL,
    `warehouse` varchar(140) NOT NULL,
    `posting_date` date NOT NULL,
    `posting_time` time NOT NULL,
    
    -- Quantity tracking
    `actual_qty` decimal(18,6) NOT NULL,    -- Change in quantity
    `qty_after_transaction` decimal(18,6),  -- Running balance
    
    -- Valuation
    `stock_uom` varchar(140),
    `incoming_rate` decimal(18,6),
    `valuation_rate` decimal(18,6),
    `stock_value` decimal(18,6),
    `stock_value_difference` decimal(18,6),
    
    -- Source reference
    `voucher_type` varchar(140) NOT NULL,
    `voucher_no` varchar(140) NOT NULL,
    `voucher_detail_no` varchar(140),
    
    -- Batch/Serial tracking
    `batch_no` varchar(140),
    `serial_no` longtext,
    
    -- Company and project
    `company` varchar(140),
    `project` varchar(140),
    
    -- Quality and expiry
    `has_batch_no` tinyint(1),
    `has_serial_no` tinyint(1),
    
    FOREIGN KEY (`item_code`) REFERENCES `tabItem`(`name`),
    FOREIGN KEY (`warehouse`) REFERENCES `tabWarehouse`(`name`),
    
    -- Critical performance indexes
    INDEX `item_warehouse` (`item_code`, `warehouse`, `posting_date`, `posting_time`),
    INDEX `warehouse_posting` (`warehouse`, `posting_date`, `posting_time`),
    INDEX `voucher_ref` (`voucher_type`, `voucher_no`),
    INDEX `posting_sort` (`posting_date`, `posting_time`, `creation`)
);
```

### Real-Time Stock Balance

**Bin System** (Optimized stock balances):
```sql
-- Bin (Real-time stock balances)
CREATE TABLE `tabBin` (
    `name` varchar(140) PRIMARY KEY,
    `item_code` varchar(140) NOT NULL,
    `warehouse` varchar(140) NOT NULL,
    
    -- Stock quantities
    `actual_qty` decimal(18,6) DEFAULT 0.0,      -- Physical stock
    `planned_qty` decimal(18,6) DEFAULT 0.0,     -- Including planned receipts
    `indented_qty` decimal(18,6) DEFAULT 0.0,    -- Material requests
    `ordered_qty` decimal(18,6) DEFAULT 0.0,     -- Purchase orders
    `reserved_qty` decimal(18,6) DEFAULT 0.0,    -- Sales orders
    `projected_qty` decimal(18,6) DEFAULT 0.0,   -- Available for sale
    
    -- Valuation
    `stock_value` decimal(18,6) DEFAULT 0.0,
    `valuation_rate` decimal(18,6) DEFAULT 0.0,
    
    UNIQUE KEY `item_warehouse` (`item_code`, `warehouse`),
    
    FOREIGN KEY (`item_code`) REFERENCES `tabItem`(`name`),
    FOREIGN KEY (`warehouse`) REFERENCES `tabWarehouse`(`name`),
    
    INDEX `actual_qty` (`actual_qty`),
    INDEX `projected_qty` (`projected_qty`)
);
```

### Serial and Batch Tracking

**Modern Serial/Batch Bundle System**:
```sql
-- Serial and Batch Bundle (Unified tracking)
CREATE TABLE `tabSerial and Batch Bundle` (
    `name` varchar(140) PRIMARY KEY,
    `item_code` varchar(140) NOT NULL,
    `warehouse` varchar(140) NOT NULL,
    `type_of_transaction` varchar(140),     -- Inward, Outward
    `voucher_type` varchar(140),
    `voucher_no` varchar(140),
    `voucher_detail_no` varchar(140),
    `posting_date` date,
    `posting_time` time,
    `total_qty` decimal(18,6),
    `avg_rate` decimal(18,6),
    `total_amount` decimal(18,6),
    `has_batch_no` tinyint(1),
    `has_serial_no` tinyint(1),
    
    FOREIGN KEY (`item_code`) REFERENCES `tabItem`(`name`),
    FOREIGN KEY (`warehouse`) REFERENCES `tabWarehouse`(`name`)
);

-- Serial and Batch Bundle Entry (Individual items)
CREATE TABLE `tabSerial and Batch Entry` (
    `name` varchar(140) PRIMARY KEY,
    `parent` varchar(140),                  -- Bundle reference
    `serial_no` varchar(140),
    `batch_no` varchar(140),
    `qty` decimal(18,6) DEFAULT 1.0,
    `stock_value_difference` decimal(18,6),
    `incoming_rate` decimal(18,6),
    `outgoing_rate` decimal(18,6),
    `warehouse` varchar(140),
    
    FOREIGN KEY (`parent`) REFERENCES `tabSerial and Batch Bundle`(`name`) ON DELETE CASCADE,
    
    INDEX `serial_no` (`serial_no`),
    INDEX `batch_no` (`batch_no`),
    INDEX `warehouse` (`warehouse`)
);
```

## Multi-Tenancy Design

### Company-Based Data Segregation

**Company Entity Model**:
```sql
-- Company (Multi-tenancy root)
CREATE TABLE `tabCompany` (
    `name` varchar(140) PRIMARY KEY,
    `company_name` varchar(140) UNIQUE,
    `abbr` varchar(10) UNIQUE,             -- Company abbreviation
    `default_currency` varchar(3),
    `country` varchar(140),
    `date_of_establishment` date,
    `date_of_incorporation` date,
    `parent_company` varchar(140),          -- Group company structure
    
    -- Default accounts
    `default_bank_account` varchar(140),
    `default_cash_account` varchar(140),
    `default_receivable_account` varchar(140),
    `default_payable_account` varchar(140),
    `cost_center` varchar(140),             -- Default cost center
    
    -- Settings
    `enable_perpetual_inventory` tinyint(1) DEFAULT 1,
    `stock_adjustment_account` varchar(140),
    `expenses_included_in_valuation` varchar(140),
    
    FOREIGN KEY (`default_currency`) REFERENCES `tabCurrency`(`name`),
    FOREIGN KEY (`parent_company`) REFERENCES `tabCompany`(`name`)
);
```

### Data Isolation Patterns

**Company-Specific Data**:
```sql
-- Most master data includes company field
ALTER TABLE `tabAccount` ADD CONSTRAINT 
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`);

ALTER TABLE `tabWarehouse` ADD CONSTRAINT
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`);

ALTER TABLE `tabCost Center` ADD CONSTRAINT
    FOREIGN KEY (`company`) REFERENCES `tabCompany`(`name`);

-- User permissions for company access
CREATE TABLE `tabUser Permission` (
    `name` varchar(140) PRIMARY KEY,
    `user` varchar(140) NOT NULL,
    `allow` varchar(140) NOT NULL,          -- DocType name
    `for_value` varchar(140) NOT NULL,      -- Specific record
    `applicable_for` varchar(140),          -- Conditional access
    
    UNIQUE KEY `user_permission` (`user`, `allow`, `for_value`),
    
    FOREIGN KEY (`user`) REFERENCES `tabUser`(`name`)
);
```

## Audit and Versioning Patterns

### Document Lifecycle Tracking

**Version Control System**:
```sql
-- Version (Document change history)
CREATE TABLE `tabVersion` (
    `name` varchar(140) PRIMARY KEY,
    `ref_doctype` varchar(140),
    `docname` varchar(140),
    `data` longtext,                        -- JSON snapshot
    `changed` longtext,                     -- Changed fields
    `comment` text,
    
    INDEX `ref_doc` (`ref_doctype`, `docname`),
    INDEX `creation` (`creation`)
);
```

### Status Management Pattern

**StatusUpdater Implementation**:
```python
# Automatic status calculation pattern
class StatusUpdater:
    def update_status(self, update_modified=True):
        """Update status based on percentage completion"""
        status_map = [
            ["per_billed", "billing_status", "Billed"],
            ["per_delivered", "delivery_status", "Delivered"]
        ]
        
        for percent_field, status_field, complete_status in status_map:
            per_field = self.get(percent_field)
            
            if per_field < 0.001:
                status = "Not " + complete_status
            elif per_field >= 99.99:
                status = complete_status  
            else:
                status = "Partly " + complete_status
                
            self.set(status_field, status)
```

**Percentage Completion Calculation**:
```sql
-- Automatic calculation of completion percentages
UPDATE `tabSales Order` so
SET per_delivered = (
    SELECT COALESCE(
        (SUM(delivered_qty) * 100.0) / NULLIF(SUM(qty), 0), 
        0
    )
    FROM `tabSales Order Item` soi 
    WHERE soi.parent = so.name
),
per_billed = (
    SELECT COALESCE(
        (SUM(billed_amt) * 100.0) / NULLIF(SUM(amount), 0),
        0  
    )
    FROM `tabSales Order Item` soi
    WHERE soi.parent = so.name
);
```

## Performance Optimization

### Indexing Strategy

**Strategic Index Design**:
```sql
-- Transaction-heavy tables need compound indexes
ALTER TABLE `tabGL Entry` 
    ADD INDEX `account_posting_performance` 
    (`account`, `posting_date`, `is_cancelled`);

ALTER TABLE `tabStock Ledger Entry`
    ADD INDEX `item_warehouse_chronological`
    (`item_code`, `warehouse`, `posting_date`, `posting_time`, `creation`);

-- Party outstanding queries
ALTER TABLE `tabPayment Ledger Entry`
    ADD INDEX `party_outstanding_analysis`
    (`party_type`, `party`, `amount_type`, `posting_date`);

-- Reporting indexes
ALTER TABLE `tabSales Invoice`
    ADD INDEX `sales_reporting`
    (`company`, `posting_date`, `docstatus`, `customer`);
```

### Query Optimization Patterns

**Efficient Outstanding Calculation**:
```sql
-- Optimized outstanding amount calculation
SELECT 
    ple.party,
    SUM(CASE WHEN ple.amount_type = 'Receivable' THEN ple.amount ELSE 0 END) as receivable,
    SUM(CASE WHEN ple.amount_type = 'Payable' THEN ple.amount ELSE 0 END) as payable,
    (SUM(CASE WHEN ple.amount_type = 'Receivable' THEN ple.amount ELSE 0 END) -
     SUM(CASE WHEN ple.amount_type = 'Payable' THEN ple.amount ELSE 0 END)) as outstanding
FROM `tabPayment Ledger Entry` ple
WHERE ple.party_type = 'Customer' 
  AND ple.company = %s
  AND ple.posting_date <= %s
GROUP BY ple.party
HAVING outstanding != 0;
```

### Caching Patterns

**Computed Field Caching**:
```sql
-- Cached calculations in document
ALTER TABLE `tabCustomer` 
    ADD COLUMN `total_sales` decimal(18,6) DEFAULT 0.0,
    ADD COLUMN `outstanding_amount` decimal(18,6) DEFAULT 0.0,
    ADD COLUMN `last_sale_date` date;

-- Trigger-based cache updates
DELIMITER $$
CREATE TRIGGER update_customer_stats 
    AFTER INSERT ON `tabSales Invoice`
    FOR EACH ROW
BEGIN
    IF NEW.docstatus = 1 THEN
        UPDATE `tabCustomer` 
        SET total_sales = total_sales + NEW.grand_total,
            outstanding_amount = outstanding_amount + NEW.outstanding_amount,
            last_sale_date = GREATEST(COALESCE(last_sale_date, '1900-01-01'), NEW.posting_date)
        WHERE name = NEW.customer;
    END IF;
END$$
DELIMITER ;
```

## Data Integrity Patterns

### Referential Integrity

**Cascade Patterns**:
```sql
-- Parent-child cascade deletion
ALTER TABLE `tabSales Invoice Item`
    ADD CONSTRAINT `fk_sales_invoice_item_parent`
    FOREIGN KEY (`parent`) REFERENCES `tabSales Invoice`(`name`) 
    ON DELETE CASCADE;

-- Prevent deletion of referenced masters
ALTER TABLE `tabGL Entry`
    ADD CONSTRAINT `fk_gl_entry_account`
    FOREIGN KEY (`account`) REFERENCES `tabAccount`(`name`)
    ON DELETE RESTRICT;
```

### Transaction Integrity

**Document Status Workflow**:
```sql
-- Ensure valid status transitions
ALTER TABLE `tabSales Invoice`
    ADD CONSTRAINT `chk_docstatus`
    CHECK (docstatus IN (0, 1, 2));

-- Prevent editing of submitted documents
CREATE TRIGGER prevent_submitted_edit
    BEFORE UPDATE ON `tabSales Invoice`
    FOR EACH ROW
BEGIN
    IF OLD.docstatus = 1 AND NEW.docstatus = 1 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Cannot modify submitted document';
    END IF;
END;
```

### Business Rule Constraints

**Financial Controls**:
```sql
-- Prevent negative stock (if configured)
CREATE TRIGGER check_negative_stock
    BEFORE INSERT ON `tabStock Ledger Entry`
    FOR EACH ROW
BEGIN
    DECLARE current_qty DECIMAL(18,6);
    
    SELECT COALESCE(actual_qty, 0) INTO current_qty
    FROM `tabBin`
    WHERE item_code = NEW.item_code AND warehouse = NEW.warehouse;
    
    IF (current_qty + NEW.actual_qty) < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Insufficient stock quantity';
    END IF;
END;
```

This database design analysis provides a comprehensive foundation for building scalable, multi-tenant ERP systems with proper data modeling, performance optimization, and integrity constraints. The patterns shown here have been proven in production environments handling large-scale enterprise data.