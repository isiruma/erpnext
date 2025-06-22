# Purchase Receipt and Purchase Order Relationship Analysis

## Table of Contents
1. [Business Overview](#business-overview)
2. [Purchase Receipt and Purchase Order Relationship](#purchase-receipt-and-purchase-order-relationship)
3. [Complete Table Schema](#complete-table-schema)
4. [Database Relationships](#database-relationships)
5. [Business Process Flow](#business-process-flow)
6. [Key Business Rules](#key-business-rules)
7. [Integration Points](#integration-points)

## Business Overview

**Purchase Receipt** is a critical document in the **Procure-to-Pay** workflow that confirms physical receipt of goods ordered through a Purchase Order. It serves as proof of delivery and triggers inventory updates and financial accounting entries.

### Core Business Purpose
- **Inventory Control**: Records actual goods received into inventory
- **Financial Accuracy**: Triggers accounting entries for received goods
- **Process Control**: Enables three-way matching (PO → Receipt → Invoice)
- **Quality Assurance**: Documents quality inspection and rejected quantities
- **Audit Trail**: Provides complete procurement documentation

### Position in Procure-to-Pay Workflow
```
Material Request → RFQ → Supplier Quotation → Purchase Order → **Purchase Receipt** → Purchase Invoice → Payment Entry
```

## Purchase Receipt and Purchase Order Relationship

### 1. Document Linkage Structure

**One-to-Many Relationship**:
- One Purchase Order can generate **multiple** Purchase Receipts
- Supports partial deliveries and staged receipts
- Each Purchase Receipt line item references specific Purchase Order items

**Reference Fields**:
```
Purchase Receipt Item.purchase_order → Purchase Order.name
Purchase Receipt Item.purchase_order_item → Purchase Order Item.name
```

### 2. Business Process Integration

#### **Step 1: Purchase Order Foundation**
- Defines what needs to be purchased
- Specifies quantities, rates, delivery schedules
- Establishes contractual terms with supplier
- Status: `Draft` → `Submitted` → `To Receive`

#### **Step 2: Purchase Receipt Execution**
- Records actual goods received vs. ordered
- Validates against Purchase Order specifications
- Handles quality inspection and rejection processes
- Updates inventory and financial records
- Status: `Draft` → `Submitted` → `Completed`

#### **Step 3: Status Synchronization**
Purchase Order status updates based on receipt progress:
- `To Receive` → `To Receive and Bill` (partial receipt)
- `To Receive and Bill` → `To Bill` (fully received, pending invoice)
- `Completed` (fully received and invoiced)

### 3. Quantity Management

**Ordered vs. Received Tracking**:
```sql
-- Purchase Order Item
ordered_qty = 100

-- Purchase Receipt Item (First delivery)
received_qty = 60
qty = 55           -- Accepted quantity
rejected_qty = 5   -- Rejected quantity

-- Purchase Order Item Updated
received_qty = 60
pending_qty = 40   -- Still to be delivered
```

**Business Rules**:
- Cannot receive more than ordered (configurable tolerance)
- Tracks cumulative receipts across multiple deliveries
- Maintains pending quantity calculations
- Supports over-receipt scenarios with approval workflows

## Complete Table Schema

### Purchase Receipt (Parent Document)
**Table**: `tabPurchase Receipt`

#### Core Identification
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `name` | VARCHAR(140) PK | Unique document ID | Auto-generated via naming series |
| `naming_series` | VARCHAR(50) | Document numbering | PR-YYYY-##### pattern |
| `title` | VARCHAR(255) | Display title | Auto-generated: Supplier - Date |
| `docstatus` | INT | Document status | 0=Draft, 1=Submitted, 2=Cancelled |

#### Supplier Information
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `supplier` | VARCHAR(140) FK | Supplier reference | Must match PO supplier |
| `supplier_name` | VARCHAR(255) | Supplier display name | Auto-fetched from supplier master |
| `supplier_delivery_note` | VARCHAR(255) | Supplier's delivery ref | Optional tracking reference |

#### Date and Timing
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `posting_date` | DATE | Receipt date | Default: Today, >= PO date |
| `posting_time` | TIME | Receipt time | Default: Current time |
| `set_posting_time` | TINYINT | Allow backdating | Requires special permission |

#### Company and Organization
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `company` | VARCHAR(140) FK | Company reference | Must match PO company |
| `cost_center` | VARCHAR(140) FK | Default cost center | For accounting allocation |
| `project` | VARCHAR(140) FK | Project reference | For project costing |

#### Currency and Financial
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `currency` | VARCHAR(3) FK | Transaction currency | Must match PO currency |
| `conversion_rate` | DECIMAL(9,6) | Exchange rate | Auto-fetched if multi-currency |
| `buying_price_list` | VARCHAR(140) FK | Price list reference | Default from supplier |
| `price_list_currency` | VARCHAR(3) FK | Price list currency | May differ from transaction |
| `plc_conversion_rate` | DECIMAL(9,6) | Price list conversion | For price calculations |

#### Warehouse Management
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `set_warehouse` | VARCHAR(140) FK | Default warehouse | Override per line item |
| `rejected_warehouse` | VARCHAR(140) FK | Rejected goods warehouse | Required if rejections exist |
| `supplier_warehouse` | VARCHAR(140) FK | Supplier warehouse | For subcontracting scenarios |
| `apply_putaway_rule` | TINYINT | Enable putaway rules | Auto-assign storage locations |

#### Totals and Amounts
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `total_qty` | DECIMAL(18,6) | Total received quantity | Sum of all line items |
| `base_total` | DECIMAL(18,6) | Total in company currency | Before taxes and charges |
| `total` | DECIMAL(18,6) | Total in transaction currency | Before taxes and charges |
| `base_grand_total` | DECIMAL(18,6) | Grand total (company) | After all taxes and charges |
| `grand_total` | DECIMAL(18,6) | Grand total (transaction) | After all taxes and charges |
| `base_net_total` | DECIMAL(18,6) | Net total (company) | After item-level discounts |
| `net_total` | DECIMAL(18,6) | Net total (transaction) | After item-level discounts |

#### Tax and Charges
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `tax_category` | VARCHAR(140) FK | Tax category | Determines applicable taxes |
| `taxes_and_charges` | VARCHAR(140) FK | Tax template | Pre-defined tax structure |
| `base_total_taxes_and_charges` | DECIMAL(18,6) | Total tax (company) | Sum of all tax amounts |
| `total_taxes_and_charges` | DECIMAL(18,6) | Total tax (transaction) | Sum of all tax amounts |

#### Returns and Special Cases
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `is_return` | TINYINT | Return receipt flag | For goods return scenarios |
| `return_against` | VARCHAR(140) FK | Original receipt ref | Required for returns |
| `subcontracting_receipt` | TINYINT | Subcontracting flag | Special handling for subcontracting |

### Purchase Receipt Item (Child Document)
**Table**: `tabPurchase Receipt Item`

#### Parent Reference
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `parent` | VARCHAR(140) FK | Purchase Receipt ref | Links to parent document |
| `parenttype` | VARCHAR(50) | Parent document type | Always "Purchase Receipt" |
| `parentfield` | VARCHAR(50) | Parent field name | Always "items" |
| `idx` | INT | Line item sequence | Determines display order |

#### Item Identification
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `item_code` | VARCHAR(140) FK | Item master reference | Must be valid purchasable item |
| `item_name` | VARCHAR(255) | Item display name | Auto-fetched from item master |
| `description` | TEXT | Item description | Auto-fetched, editable |
| `brand` | VARCHAR(140) FK | Brand reference | For brand-specific reporting |
| `item_group` | VARCHAR(140) FK | Item category | For classification |
| `supplier_part_no` | VARCHAR(255) | Supplier's part number | Cross-reference tracking |

#### Quantity Management
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `qty` | DECIMAL(18,6) | Accepted quantity | Actual quantity accepted |
| `received_qty` | DECIMAL(18,6) | Total received | Qty + Rejected Qty |
| `rejected_qty` | DECIMAL(18,6) | Rejected quantity | Goods not accepted |
| `stock_qty` | DECIMAL(18,6) | Stock UOM quantity | For inventory posting |
| `returned_qty` | DECIMAL(18,6) | Previously returned | For return tracking |
| `uom` | VARCHAR(140) FK | Transaction UOM | Unit of measure for transaction |
| `stock_uom` | VARCHAR(140) FK | Stock UOM | Item's base unit |
| `conversion_factor` | DECIMAL(18,6) | UOM conversion | Stock UOM = Qty × Factor |

#### Pricing and Amounts
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `rate` | DECIMAL(18,6) | Unit rate (transaction) | Price per unit |
| `amount` | DECIMAL(18,6) | Line amount (transaction) | Qty × Rate |
| `base_rate` | DECIMAL(18,6) | Unit rate (company) | Converted to company currency |
| `base_amount` | DECIMAL(18,6) | Line amount (company) | Converted line total |
| `valuation_rate` | DECIMAL(18,6) | Stock valuation rate | For inventory valuation |
| `price_list_rate` | DECIMAL(18,6) | Standard price | From price list |
| `discount_percentage` | DECIMAL(18,6) | Discount % | Line item discount |
| `discount_amount` | DECIMAL(18,6) | Discount amount | Absolute discount |

#### Warehouse and Location
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `warehouse` | VARCHAR(140) FK | Receiving warehouse | Where goods are stored |
| `rejected_warehouse` | VARCHAR(140) FK | Rejected items warehouse | For quality failures |
| `from_warehouse` | VARCHAR(140) FK | Source warehouse | For inter-warehouse transfers |

#### **Critical Reference Fields**
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `purchase_order` | VARCHAR(140) FK | **Source Purchase Order** | Links to originating PO |
| `purchase_order_item` | VARCHAR(140) FK | **Source PO line item** | Links to specific PO item |
| `material_request` | VARCHAR(140) FK | Original material request | Tracks demand source |
| `material_request_item` | VARCHAR(140) FK | Original MR item | Specific demand line |
| `purchase_invoice` | VARCHAR(140) FK | Related invoice | Three-way matching |
| `purchase_invoice_item` | VARCHAR(140) FK | Related invoice item | Line-level matching |

#### Quality and Compliance
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `quality_inspection` | VARCHAR(140) FK | QI reference | Links to quality check |
| `schedule_date` | DATE | Expected delivery | From Purchase Order |
| `manufacturer` | VARCHAR(140) FK | Manufacturer ref | For traceability |
| `manufacturer_part_no` | VARCHAR(255) | Manufacturer part | OEM part number |

#### Serial and Batch Tracking
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `serial_and_batch_bundle` | VARCHAR(140) FK | Serial/batch bundle | New tracking system |
| `rejected_serial_and_batch_bundle` | VARCHAR(140) FK | Rejected items bundle | For rejected goods tracking |
| `serial_no` | TEXT | Serial numbers | Legacy serial tracking |
| `batch_no` | VARCHAR(255) | Batch number | Legacy batch tracking |

#### Accounting Integration
| Field | Type | Purpose | Business Rule |
|-------|------|---------|---------------|
| `expense_account` | VARCHAR(140) FK | Expense account | For GL posting |
| `cost_center` | VARCHAR(140) FK | Cost center | For cost allocation |
| `project` | VARCHAR(140) FK | Project reference | For project costing |
| `item_tax_rate` | TEXT | Item-specific taxes | JSON format tax rates |

## Database Relationships

### Core Relationship Diagram
```
┌─────────────────────┐     ┌─────────────────────┐
│   Purchase Order    │────▷│  Purchase Receipt   │
├─────────────────────┤     ├─────────────────────┤
│ name (PK)          │     │ name (PK)          │
│ supplier (FK)      │     │ supplier (FK)      │
│ company (FK)       │     │ company (FK)       │
│ status             │     │ docstatus          │
└─────────────────────┘     └─────────────────────┘
          │                           │
          ▼                           ▼
┌─────────────────────┐     ┌─────────────────────┐
│ Purchase Order Item │────▷│Purchase Receipt Item│
├─────────────────────┤     ├─────────────────────┤
│ parent (FK)        │     │ parent (FK)        │
│ item_code (FK)     │     │ item_code (FK)     │
│ qty                │     │ qty                │
│ rate               │     │ rate               │
└─────────────────────┘     │ purchase_order (FK)│
                            │ purchase_order_item │
                            └─────────────────────┘
```

### Key Foreign Key Constraints

#### **Purchase Receipt Level**
```sql
-- Master Data References
FOREIGN KEY (supplier) REFERENCES tabSupplier(name)
FOREIGN KEY (company) REFERENCES tabCompany(name)
FOREIGN KEY (currency) REFERENCES tabCurrency(name)
FOREIGN KEY (cost_center) REFERENCES `tabCost Center`(name)
FOREIGN KEY (project) REFERENCES tabProject(name)

-- Warehouse References
FOREIGN KEY (set_warehouse) REFERENCES tabWarehouse(name)
FOREIGN KEY (rejected_warehouse) REFERENCES tabWarehouse(name)

-- Return References
FOREIGN KEY (return_against) REFERENCES `tabPurchase Receipt`(name)
```

#### **Purchase Receipt Item Level**
```sql
-- Parent-Child Relationship
FOREIGN KEY (parent) REFERENCES `tabPurchase Receipt`(name)

-- Item and UOM References
FOREIGN KEY (item_code) REFERENCES tabItem(name)
FOREIGN KEY (uom) REFERENCES tabUOM(name)
FOREIGN KEY (stock_uom) REFERENCES tabUOM(name)

-- **Critical Purchase Order Links**
FOREIGN KEY (purchase_order) REFERENCES `tabPurchase Order`(name)
FOREIGN KEY (purchase_order_item) REFERENCES `tabPurchase Order Item`(name)

-- Upstream Document References
FOREIGN KEY (material_request) REFERENCES `tabMaterial Request`(name)
FOREIGN KEY (material_request_item) REFERENCES `tabMaterial Request Item`(name)

-- Downstream Document References
FOREIGN KEY (purchase_invoice) REFERENCES `tabPurchase Invoice`(name)
FOREIGN KEY (purchase_invoice_item) REFERENCES `tabPurchase Invoice Item`(name)

-- Warehouse and Location
FOREIGN KEY (warehouse) REFERENCES tabWarehouse(name)
FOREIGN KEY (rejected_warehouse) REFERENCES tabWarehouse(name)

-- Accounting References
FOREIGN KEY (expense_account) REFERENCES tabAccount(name)
FOREIGN KEY (cost_center) REFERENCES `tabCost Center`(name)
FOREIGN KEY (project) REFERENCES tabProject(name)
```

### Index Strategy for Performance
```sql
-- Purchase Receipt Performance Indexes
CREATE INDEX idx_pr_supplier_date ON `tabPurchase Receipt` (supplier, posting_date);
CREATE INDEX idx_pr_company_status ON `tabPurchase Receipt` (company, docstatus, posting_date);
CREATE INDEX idx_pr_posting_date ON `tabPurchase Receipt` (posting_date DESC);

-- Purchase Receipt Item Performance Indexes
CREATE INDEX idx_pri_parent_idx ON `tabPurchase Receipt Item` (parent, idx);
CREATE INDEX idx_pri_item_warehouse ON `tabPurchase Receipt Item` (item_code, warehouse);
CREATE INDEX idx_pri_po_reference ON `tabPurchase Receipt Item` (purchase_order, purchase_order_item);
CREATE INDEX idx_pri_mr_reference ON `tabPurchase Receipt Item` (material_request, material_request_item);
```

## Business Process Flow

### 1. Receipt Creation Process

#### **Automatic Creation from Purchase Order**
```python
# Pseudo-code for receipt creation
purchase_receipt = frappe.new_doc("Purchase Receipt")
purchase_receipt.supplier = purchase_order.supplier
purchase_receipt.company = purchase_order.company

for po_item in purchase_order.items:
    pr_item = purchase_receipt.append("items")
    pr_item.item_code = po_item.item_code
    pr_item.qty = po_item.qty  # Can be modified
    pr_item.rate = po_item.rate
    pr_item.purchase_order = purchase_order.name
    pr_item.purchase_order_item = po_item.name
```

#### **Manual Quantity Adjustments**
- **Full Receipt**: Qty = Ordered Qty
- **Partial Receipt**: Qty < Ordered Qty (remaining qty stays pending)
- **Over Receipt**: Qty > Ordered Qty (requires approval based on tolerance)
- **Quality Rejection**: Split between `qty` (accepted) and `rejected_qty`

### 2. Validation and Business Logic

#### **Pre-Submission Validations**
1. **Supplier Consistency**: Receipt supplier must match PO supplier
2. **Item Validation**: All items must exist in PO
3. **Quantity Limits**: Cannot exceed ordered quantity (unless configured)
4. **Warehouse Validation**: Warehouses must belong to receipt company
5. **Date Logic**: Receipt date should be >= PO date

#### **Quality Inspection Integration**
```
If Item.inspection_required_before_purchase = 1:
    1. Create Quality Inspection document
    2. Link QI to Purchase Receipt Item
    3. Cannot submit receipt until QI is approved
    4. Rejected quantities automatically populated from QI
```

### 3. Impact on Purchase Order Status

#### **Status Calculation Logic**
```python
# Purchase Order status updates
def update_purchase_order_status(po):
    total_ordered = sum(item.qty for item in po.items)
    total_received = sum(item.received_qty for item in po.items)
    total_billed = sum(item.billed_amt for item in po.items)
    
    if total_received == 0:
        po.status = "To Receive"
    elif total_received < total_ordered:
        po.status = "To Receive and Bill"
    elif total_billed < po.grand_total:
        po.status = "To Bill"
    else:
        po.status = "Completed"
```

#### **Line Item Status Tracking**
```python
# Purchase Order Item updates
def update_po_item_status(po_item, pr_item):
    po_item.received_qty += pr_item.qty
    po_item.pending_qty = po_item.qty - po_item.received_qty
    
    if po_item.received_qty >= po_item.qty:
        po_item.status = "Received"
    else:
        po_item.status = "Partially Received"
```

## Key Business Rules

### 1. Quantity Management Rules

#### **Receipt Quantity Validations**
- **Standard Rule**: `received_qty <= ordered_qty * (1 + over_receipt_allowance%)`
- **Zero Quantity**: Cannot create receipt with zero accepted quantity
- **Negative Quantities**: Only allowed for return receipts (`is_return = 1`)
- **UOM Consistency**: Transaction UOM must be valid for the item

#### **Quality Control Rules**
- **Inspection Required**: Some items require mandatory quality inspection
- **Rejection Handling**: Rejected quantities stored in separate warehouse
- **Sample Retention**: Option to retain samples for testing
- **Return Process**: Rejected goods can be returned to supplier

### 2. Financial Control Rules

#### **Valuation Logic**
```python
# Stock valuation calculation
def calculate_valuation_rate(pr_item):
    # Include landed costs, taxes, and additional charges
    base_amount = pr_item.base_amount
    landed_costs = pr_item.landed_cost_voucher_amount or 0
    tax_amount = pr_item.item_tax_amount or 0
    
    valuation_rate = (base_amount + landed_costs + tax_amount) / pr_item.stock_qty
    return valuation_rate
```

#### **Currency Handling**
- **Multi-Currency**: Supports different transaction currencies
- **Exchange Rates**: Auto-fetched or manually entered
- **Conversion**: All amounts stored in both transaction and company currency
- **Rate Fluctuation**: Tracks exchange rate differences

### 3. Inventory Impact Rules

#### **Stock Ledger Posting**
```python
# Stock ledger entry creation
stock_entry = {
    "item_code": pr_item.item_code,
    "warehouse": pr_item.warehouse,
    "actual_qty": pr_item.stock_qty,
    "incoming_rate": pr_item.valuation_rate,
    "voucher_type": "Purchase Receipt",
    "voucher_no": pr.name
}
```

#### **Bin Updates**
- **Actual Qty**: Increases by received quantity
- **Valuation Rate**: Updated based on moving average/FIFO
- **Reserved Qty**: May be reduced if linked to sales orders
- **Projected Qty**: Recalculated based on pending transactions

### 4. Approval Workflows

#### **Over-Receipt Scenarios**
```python
# Over-receipt handling
if received_qty > (ordered_qty * (1 + tolerance_percentage)):
    if requires_approval:
        send_for_approval()
    else:
        raise_validation_error()
```

#### **Return Authorization**
- **Return Receipts**: Require original receipt reference
- **Credit Notes**: May require approval for high-value returns
- **Quality Issues**: Link to non-conformance reports

## Integration Points

### 1. Inventory Management Integration

#### **Stock Ledger Impact**
- **Positive Entries**: Increase inventory for received goods
- **Negative Entries**: Decrease inventory for returns
- **Serial Numbers**: Track individual items with serial numbers
- **Batch Management**: Group items by batch for expiry tracking

#### **Warehouse Operations**
- **Putaway Rules**: Auto-assign storage locations
- **Bin Allocation**: Distribute stock across multiple bins
- **Stock Reservations**: May trigger reservation releases

### 2. Financial Accounting Integration

#### **General Ledger Entries**
```python
# GL entries for purchase receipt
gl_entries = [
    {
        "account": item.expense_account,  # Stock in Hand / Asset Account
        "debit": item.base_amount,
        "cost_center": item.cost_center,
        "project": item.project
    },
    {
        "account": "Goods Received Not Invoiced",  # GRNI Liability
        "credit": item.base_amount,
        "cost_center": item.cost_center
    }
]
```

#### **Tax Accounting**
- **Input Tax Credit**: For VAT/GST eligible purchases  
- **Withholding Tax**: For TDS/TCS scenarios
- **Landed Costs**: Integration with Landed Cost Voucher

### 3. Quality Management Integration

#### **Quality Inspection Workflow**
```
Purchase Receipt → Quality Inspection → Acceptance/Rejection → Stock Update
```

#### **Non-Conformance Management**
- **Supplier Quality Issues**: Track quality problems by supplier
- **Corrective Actions**: Link to supplier feedback and improvement plans
- **Quality Metrics**: Calculate supplier quality ratings

### 4. Project Management Integration

#### **Project Costing**
- **Direct Costs**: Allocate receipt amounts to projects
- **Indirect Costs**: Distribute overheads based on allocation rules
- **Budget Control**: Check against project budgets
- **Progress Tracking**: Update project material consumption

### 5. Manufacturing Integration

#### **Subcontracting Process**
```
Purchase Order (Subcontract) → Purchase Receipt → Raw Material Consumption → Finished Goods Receipt
```

#### **BOM Integration**
- **Raw Material Supply**: Track materials sent to subcontractors
- **Finished Goods Receipt**: Receive completed products
- **Scrap Accounting**: Handle material wastage and scrap

---

*This comprehensive analysis provides the complete understanding of Purchase Receipt functionality, its intricate relationship with Purchase Order, and the detailed database schema required for implementation. The document serves as both business logic reference and technical specification for ERP development.*