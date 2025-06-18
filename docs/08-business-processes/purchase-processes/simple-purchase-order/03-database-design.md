# Simple Purchase Order - Database Design Document

## Overview

This document provides comprehensive database design specifications for the Simple Purchase Order module, including table structures, relationships, constraints, and optimization strategies.

## Database Architecture

### Design Principles
1. **Normalization**: 3rd Normal Form (3NF) for data integrity
2. **Performance**: Strategic denormalization for frequently accessed data
3. **Scalability**: Indexed columns for fast queries
4. **Flexibility**: Support for multi-currency and multi-company scenarios
5. **Auditability**: Complete change tracking and user attribution

### Entity Relationship Diagram

```
┌─────────────────────┐      ┌─────────────────────┐
│      Company        │      │      Supplier       │
├─────────────────────┤      ├─────────────────────┤
│ id (PK)            │      │ id (PK)            │
│ company_name       │      │ supplier_name      │
│ base_currency      │      │ default_currency   │
│ ...                │      │ ...                │
└─────────────────────┘      └─────────────────────┘
           │                            │
           │                            │
           ▼                            ▼
┌─────────────────────────────────────────────────────────┐
│                Purchase Order                           │
├─────────────────────────────────────────────────────────┤
│ id (PK)                                                │
│ naming_series                                          │
│ supplier_id (FK) → Supplier.id                        │
│ company_id (FK) → Company.id                          │
│ transaction_date                                       │     
│ status                                                 │
│ net_total, tax_amount, grand_total                    │
│ currency, conversion_rate                              │
│ ...                                                    │
└─────────────────────────────────────────────────────────┘
           │
           │ 1:N
           ▼
┌─────────────────────────────────────────────────────────┐
│              Purchase Order Item                        │
├─────────────────────────────────────────────────────────┤
│ id (PK)                                                │
│ parent_id (FK) → Purchase Order.id                    │
│ item_code (FK) → Item.id                              │
│ warehouse_id (FK) → Warehouse.id                      │
│ qty, rate, amount                                      │
│ line_sequence                                          │
│ ...                                                    │
└─────────────────────────────────────────────────────────┘
           │                │                │
           ▼                ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│    Item     │  │  Warehouse  │  │    UOM      │
├─────────────┤  ├─────────────┤  ├─────────────┤
│ id (PK)    │  │ id (PK)    │  │ id (PK)    │
│ item_name  │  │ warehouse   │  │ uom_name   │
│ ...        │  │ ...        │  │ ...        │
└─────────────┘  └─────────────┘  └─────────────┘
```

## Table Definitions

### 1. Purchase Order (Main Transaction Table)

```sql
CREATE TABLE purchase_order (
    -- Primary Key
    id VARCHAR(50) PRIMARY KEY,
    
    -- Document Identification
    naming_series VARCHAR(20) NOT NULL DEFAULT 'SPO-.YYYY.-',
    
    -- Supplier Information
    supplier_id VARCHAR(50) NOT NULL,
    supplier_name VARCHAR(150), -- Denormalized for performance
    
    -- Company and Date Information
    company_id VARCHAR(50) NOT NULL,
    transaction_date DATE NOT NULL DEFAULT CURRENT_DATE,
    required_by_date DATE,
    
    -- Currency and Financial
    currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    conversion_rate DECIMAL(10,9) NOT NULL DEFAULT 1.000000000,
    
    -- Totals
    net_total DECIMAL(15,2) NOT NULL DEFAULT 0.00,
    tax_rate DECIMAL(5,2) DEFAULT 0.00,
    tax_amount DECIMAL(15,2) NOT NULL DEFAULT 0.00,
    grand_total DECIMAL(15,2) NOT NULL DEFAULT 0.00,
    
    -- Status and Workflow
    status VARCHAR(20) NOT NULL DEFAULT 'Draft',
    docstatus TINYINT NOT NULL DEFAULT 0, -- 0=Draft, 1=Submitted, 2=Cancelled
    
    -- Address and Contact (Optional)
    supplier_address_id VARCHAR(50),
    contact_person_id VARCHAR(50),
    
    -- Additional Information
    remarks TEXT,
    amended_from VARCHAR(50), -- Reference to original document
    
    -- Audit Fields
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by VARCHAR(50) NOT NULL,
    updated_by VARCHAR(50) NOT NULL,
    
    -- Constraints
    CONSTRAINT chk_po_conversion_rate CHECK (conversion_rate > 0),
    CONSTRAINT chk_po_totals CHECK (net_total >= 0 AND tax_amount >= 0 AND grand_total >= 0),
    CONSTRAINT chk_po_docstatus CHECK (docstatus IN (0, 1, 2)),
    CONSTRAINT chk_po_status CHECK (status IN ('Draft', 'Submitted', 'Partially Received', 'Received', 'Completed', 'Cancelled')),
    CONSTRAINT chk_po_dates CHECK (required_by_date IS NULL OR required_by_date >= transaction_date),
    
    -- Foreign Key Constraints
    CONSTRAINT fk_po_supplier FOREIGN KEY (supplier_id) REFERENCES supplier(id),
    CONSTRAINT fk_po_company FOREIGN KEY (company_id) REFERENCES company(id),
    CONSTRAINT fk_po_address FOREIGN KEY (supplier_address_id) REFERENCES address(id),
    CONSTRAINT fk_po_contact FOREIGN KEY (contact_person_id) REFERENCES contact(id),
    CONSTRAINT fk_po_amended FOREIGN KEY (amended_from) REFERENCES purchase_order(id)
);
```

### 2. Purchase Order Item (Child Table)

```sql
CREATE TABLE purchase_order_item (
    -- Primary Key
    id VARCHAR(50) PRIMARY KEY,
    
    -- Parent Reference
    parent_id VARCHAR(50) NOT NULL,
    
    -- Item Information
    item_code VARCHAR(50) NOT NULL,
    item_name VARCHAR(150), -- Denormalized for performance
    description TEXT,
    
    -- Quantity and Pricing
    qty DECIMAL(10,3) NOT NULL,
    uom VARCHAR(20) NOT NULL,
    rate DECIMAL(15,4) NOT NULL,
    amount DECIMAL(15,2) NOT NULL DEFAULT 0.00,
    
    -- Warehouse and Delivery
    warehouse_id VARCHAR(50) NOT NULL,
    required_by_date DATE,
    
    -- Line Management
    line_sequence INTEGER NOT NULL DEFAULT 1,
    
    -- Audit Fields
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT chk_poi_qty CHECK (qty > 0),
    CONSTRAINT chk_poi_rate CHECK (rate > 0),
    CONSTRAINT chk_poi_amount CHECK (amount >= 0),
    CONSTRAINT chk_poi_sequence CHECK (line_sequence > 0),
    
    -- Foreign Key Constraints
    CONSTRAINT fk_poi_parent FOREIGN KEY (parent_id) REFERENCES purchase_order(id) ON DELETE CASCADE,
    CONSTRAINT fk_poi_item FOREIGN KEY (item_code) REFERENCES item(id),
    CONSTRAINT fk_poi_warehouse FOREIGN KEY (warehouse_id) REFERENCES warehouse(id),
    CONSTRAINT fk_poi_uom FOREIGN KEY (uom) REFERENCES uom(id),
    
    -- Unique Constraints
    CONSTRAINT uk_poi_parent_sequence UNIQUE (parent_id, line_sequence)
);
```

### 3. Supporting Master Tables

#### Supplier Master
```sql
CREATE TABLE supplier (
    id VARCHAR(50) PRIMARY KEY,
    supplier_name VARCHAR(150) NOT NULL,
    supplier_group_id VARCHAR(50),
    default_currency VARCHAR(3) DEFAULT 'USD',
    payment_terms_id VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_supplier_name UNIQUE (supplier_name)
);
```

#### Item Master
```sql
CREATE TABLE item (
    id VARCHAR(50) PRIMARY KEY,
    item_name VARCHAR(150) NOT NULL,
    item_group_id VARCHAR(50),
    description TEXT,
    stock_uom VARCHAR(20) NOT NULL,
    is_purchase_item BOOLEAN DEFAULT TRUE,
    is_active BOOLEAN DEFAULT TRUE,
    last_purchase_rate DECIMAL(15,4),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_item_name UNIQUE (item_name)
);
```

#### Company Master
```sql
CREATE TABLE company (
    id VARCHAR(50) PRIMARY KEY,
    company_name VARCHAR(150) NOT NULL,
    abbr VARCHAR(10) NOT NULL,
    base_currency VARCHAR(3) NOT NULL DEFAULT 'USD',
    country VARCHAR(50),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT uk_company_name UNIQUE (company_name),
    CONSTRAINT uk_company_abbr UNIQUE (abbr)
);
```

#### Warehouse Master
```sql
CREATE TABLE warehouse (
    id VARCHAR(50) PRIMARY KEY,
    warehouse_name VARCHAR(150) NOT NULL,
    company_id VARCHAR(50) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_warehouse_company FOREIGN KEY (company_id) REFERENCES company(id),
    CONSTRAINT uk_warehouse_name_company UNIQUE (warehouse_name, company_id)
);
```

## Indexes and Performance Optimization

### Primary Indexes (Already created with PRIMARY KEY)

### Secondary Indexes for Purchase Order
```sql
-- Most common query patterns
CREATE INDEX idx_po_supplier_date ON purchase_order (supplier_id, transaction_date DESC);
CREATE INDEX idx_po_company_status ON purchase_order (company_id, status, transaction_date DESC);
CREATE INDEX idx_po_status_docstatus ON purchase_order (status, docstatus);
CREATE INDEX idx_po_date_range ON purchase_order (transaction_date DESC, required_by_date);
CREATE INDEX idx_po_grand_total ON purchase_order (grand_total DESC);
CREATE INDEX idx_po_created_by ON purchase_order (created_by, created_at DESC);

-- Support for audit queries
CREATE INDEX idx_po_amended_from ON purchase_order (amended_from) WHERE amended_from IS NOT NULL;
```

### Secondary Indexes for Purchase Order Item
```sql
-- Parent-child navigation
CREATE INDEX idx_poi_parent_sequence ON purchase_order_item (parent_id, line_sequence);

-- Item analysis queries
CREATE INDEX idx_poi_item_date ON purchase_order_item poi 
    JOIN purchase_order po ON poi.parent_id = po.id (poi.item_code, po.transaction_date DESC);

-- Warehouse-based queries
CREATE INDEX idx_poi_warehouse ON purchase_order_item (warehouse_id, required_by_date);

-- Amount-based analysis
CREATE INDEX idx_poi_amount ON purchase_order_item (amount DESC);
```

### Composite Indexes for Reporting
```sql
-- Supplier performance analysis
CREATE INDEX idx_po_supplier_perf ON purchase_order (supplier_id, status, grand_total, transaction_date);

-- Company financial reporting
CREATE INDEX idx_po_company_financial ON purchase_order (company_id, currency, transaction_date, grand_total);
```

## Data Integrity and Constraints

### Referential Integrity
1. **Cascade Deletes**: Purchase Order Items deleted when parent Purchase Order is deleted
2. **Restrict Deletes**: Cannot delete Supplier/Item/Company if referenced in orders
3. **Null Handling**: Optional fields allow NULL, required fields have NOT NULL constraint

### Business Logic Constraints
1. **Date Logic**: Required by date must be >= transaction date
2. **Financial Logic**: All monetary amounts must be >= 0
3. **Status Logic**: Status transitions follow defined workflow
4. **Quantity Logic**: Quantities and rates must be > 0

### Custom Triggers
```sql
-- Automatically update totals when items change
DELIMITER //
CREATE TRIGGER trg_poi_update_totals 
    AFTER INSERT ON purchase_order_item
    FOR EACH ROW
BEGIN
    UPDATE purchase_order 
    SET net_total = (
        SELECT COALESCE(SUM(amount), 0) 
        FROM purchase_order_item 
        WHERE parent_id = NEW.parent_id
    ),
    updated_at = CURRENT_TIMESTAMP
    WHERE id = NEW.parent_id;
    
    UPDATE purchase_order 
    SET tax_amount = net_total * tax_rate / 100,
        grand_total = net_total + (net_total * tax_rate / 100)
    WHERE id = NEW.parent_id;
END//

CREATE TRIGGER trg_poi_update_totals_upd
    AFTER UPDATE ON purchase_order_item
    FOR EACH ROW
BEGIN
    UPDATE purchase_order 
    SET net_total = (
        SELECT COALESCE(SUM(amount), 0) 
        FROM purchase_order_item 
        WHERE parent_id = NEW.parent_id
    ),
    updated_at = CURRENT_TIMESTAMP
    WHERE id = NEW.parent_id;
    
    UPDATE purchase_order 
    SET tax_amount = net_total * tax_rate / 100,
        grand_total = net_total + (net_total * tax_rate / 100)
    WHERE id = NEW.parent_id;
END//

CREATE TRIGGER trg_poi_update_totals_del
    AFTER DELETE ON purchase_order_item
    FOR EACH ROW
BEGIN
    UPDATE purchase_order 
    SET net_total = (
        SELECT COALESCE(SUM(amount), 0) 
        FROM purchase_order_item 
        WHERE parent_id = OLD.parent_id
    ),
    updated_at = CURRENT_TIMESTAMP
    WHERE id = OLD.parent_id;
    
    UPDATE purchase_order 
    SET tax_amount = net_total * tax_rate / 100,
        grand_total = net_total + (net_total * tax_rate / 100)
    WHERE id = OLD.parent_id;
END//
DELIMITER ;
```

## Sample Data

### Sample Purchase Order
```sql
INSERT INTO purchase_order (
    id, naming_series, supplier_id, supplier_name, company_id, 
    transaction_date, currency, conversion_rate, net_total, 
    tax_rate, tax_amount, grand_total, status, docstatus, 
    created_by, updated_by
) VALUES (
    'SPO-2025-0001', 'SPO-.YYYY.-', 'SUPP-001', 'ABC Suppliers Ltd', 
    'COMP-001', '2025-06-18', 'USD', 1.000000000, 1000.00, 
    10.00, 100.00, 1100.00, 'Draft', 0, 
    'user@company.com', 'user@company.com'
);
```

### Sample Purchase Order Items
```sql
INSERT INTO purchase_order_item (
    id, parent_id, item_code, item_name, description, 
    qty, uom, rate, amount, warehouse_id, line_sequence
) VALUES 
('SPO-2025-0001-1', 'SPO-2025-0001', 'ITEM-001', 'Widget A', 'High quality widget', 
 10.000, 'Nos', 50.0000, 500.00, 'WH-001', 1),
('SPO-2025-0001-2', 'SPO-2025-0001', 'ITEM-002', 'Widget B', 'Premium widget', 
 5.000, 'Nos', 100.0000, 500.00, 'WH-001', 2);
```

## Query Patterns and Optimization

### Common Query Patterns

#### 1. List Purchase Orders with Filters
```sql
SELECT po.id, po.transaction_date, po.supplier_name, po.grand_total, po.status
FROM purchase_order po
WHERE po.company_id = ? 
  AND po.transaction_date BETWEEN ? AND ?
  AND po.status IN ('Draft', 'Submitted')
ORDER BY po.transaction_date DESC, po.id
LIMIT 50 OFFSET ?;
```

#### 2. Get Purchase Order with Items
```sql
SELECT po.*, poi.item_code, poi.item_name, poi.qty, poi.rate, poi.amount
FROM purchase_order po
LEFT JOIN purchase_order_item poi ON po.id = poi.parent_id
WHERE po.id = ?
ORDER BY poi.line_sequence;
```

#### 3. Supplier Performance Analysis
```sql
SELECT 
    po.supplier_id,
    po.supplier_name,
    COUNT(*) as order_count,
    SUM(po.grand_total) as total_value,
    AVG(po.grand_total) as avg_order_value,
    COUNT(CASE WHEN po.status = 'Completed' THEN 1 END) as completed_orders
FROM purchase_order po
WHERE po.transaction_date >= DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH)
  AND po.docstatus = 1
GROUP BY po.supplier_id, po.supplier_name
ORDER BY total_value DESC;
```

#### 4. Item Purchase History
```sql
SELECT 
    poi.item_code,
    poi.item_name,
    po.supplier_name,
    po.transaction_date,
    poi.qty,
    poi.rate,
    poi.amount
FROM purchase_order_item poi
JOIN purchase_order po ON poi.parent_id = po.id
WHERE poi.item_code = ?
  AND po.docstatus = 1
ORDER BY po.transaction_date DESC
LIMIT 10;
```

## Database Maintenance

### Regular Maintenance Tasks

#### 1. Index Maintenance
```sql
-- Rebuild indexes monthly
OPTIMIZE TABLE purchase_order;
OPTIMIZE TABLE purchase_order_item;

-- Update statistics
ANALYZE TABLE purchase_order;
ANALYZE TABLE purchase_order_item;
```

#### 2. Archival Strategy
```sql
-- Archive old draft orders (older than 1 year)
CREATE TABLE purchase_order_archive AS 
SELECT * FROM purchase_order 
WHERE docstatus = 0 AND created_at < DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR);

CREATE TABLE purchase_order_item_archive AS
SELECT poi.* FROM purchase_order_item poi
JOIN purchase_order_archive poa ON poi.parent_id = poa.id;

-- Delete archived records
DELETE poi FROM purchase_order_item poi
JOIN purchase_order_archive poa ON poi.parent_id = poa.id;

DELETE FROM purchase_order 
WHERE docstatus = 0 AND created_at < DATE_SUB(CURRENT_DATE, INTERVAL 1 YEAR);
```

#### 3. Data Cleanup
```sql
-- Clean up orphaned records
DELETE FROM purchase_order_item 
WHERE parent_id NOT IN (SELECT id FROM purchase_order);

-- Reset sequences for consistent ordering
SET @row_number = 0;
UPDATE purchase_order_item 
SET line_sequence = (@row_number := @row_number + 1)
WHERE parent_id = ?
ORDER BY created_at;
```

This database design provides a solid foundation for the Simple Purchase Order module while maintaining flexibility for future enhancements and ensuring optimal performance for typical business operations.