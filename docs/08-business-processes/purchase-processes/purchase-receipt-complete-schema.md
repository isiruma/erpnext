# Purchase Receipt - Complete Table Schema (From ERPNext Source Code)

## Overview

This document provides the complete table schema for Purchase Receipt based on the actual JSON definition files from the ERPNext codebase. This includes every field, property, and relationship exactly as implemented in the system.

**Source Files:**
- `/erpnext/stock/doctype/purchase_receipt/purchase_receipt.json`
- `/erpnext/stock/doctype/purchase_receipt_item/purchase_receipt_item.json`

---

## Purchase Receipt (Parent Document)
**Table:** `tabPurchase Receipt`
**Module:** Stock
**Document Type:** Submittable Document
**Auto Name:** `naming_series:`
**Engine:** InnoDB

### Document Properties
- **Is Submittable:** Yes (has docstatus workflow)
- **Allow Import:** Yes
- **Allow Auto Repeat:** Yes
- **Track Changes:** Yes
- **Grid Page Length:** 50
- **Sort Field:** creation
- **Sort Order:** DESC

### Complete Field Schema

#### 1. **Basic Document Information**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `name` | VARCHAR(140) | Document ID | Primary Key, Auto-generated | Unique document identifier |
| `naming_series` | Select | Series | Required, Options: "MAT-PRE-.YYYY.-", "MAT-PR-RET-.YYYY.-" | Document numbering series |
| `title` | Data | Title | Hidden, Default: "{supplier_name}", Allow on Submit | Document display title |
| `amended_from` | Link | Amended From | Hidden, Read Only, Options: "Purchase Receipt" | Reference to original if amended |

#### 2. **Supplier and Contact Information**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `supplier` | Link | Supplier | Required, Bold, Options: "Supplier", Search Index | Primary supplier reference |
| `supplier_name` | Data | Supplier Name | Bold, Read Only, Fetch From: "supplier.supplier_name" | Supplier display name |
| `supplier_delivery_note` | Data | Supplier Delivery Note | | Supplier's delivery reference |
| `supplier_address` | Link | Supplier Address | Options: "Address" | Supplier address reference |
| `contact_person` | Link | Contact Person | Options: "Contact" | Contact person reference |
| `address_display` | Text Editor | Address | Read Only | Formatted address display |
| `contact_display` | Small Text | Contact | Read Only | Contact details display |
| `contact_mobile` | Small Text | Mobile No | Read Only, Options: "Phone" | Contact mobile number |
| `contact_email` | Small Text | Contact Email | Read Only, Options: "Email" | Contact email address |

#### 3. **Date and Time Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `posting_date` | Date | Date | Required, Default: "Today", Search Index | Receipt posting date |
| `posting_time` | Time | Posting Time | Required | Receipt posting time |
| `set_posting_time` | Check | Edit Posting Date and Time | Default: 0, Depends on docstatus | Allow backdating |

#### 4. **Company and Organization**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `company` | Link | Company | Required, Options: "Company", Remember Last Selected | Company reference |
| `cost_center` | Link | Cost Center | Options: "Cost Center" | Default cost center |
| `project` | Link | Project | Options: "Project" | Project allocation |

#### 5. **Return and Reference Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `is_return` | Check | Is Return | Default: 0, Read Only | Return receipt flag |
| `return_against` | Link | Return Against Purchase Receipt | Options: "Purchase Receipt", Depends on: "is_return" | Original receipt reference |
| `subcontracting_receipt` | Link | Subcontracting Receipt | Options: "Subcontracting Receipt", Search Index | Subcontracting reference |
| `inter_company_reference` | Link | Inter Company Reference | Options: "Delivery Note", Read Only | Inter-company transaction reference |

#### 6. **Currency and Exchange Rate**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `currency` | Link | Currency | Required, Options: "Currency" | Transaction currency |
| `conversion_rate` | Float | Exchange Rate | Required, Precision: 9 | Currency conversion rate |
| `buying_price_list` | Link | Price List | Options: "Price List" | Price list reference |
| `price_list_currency` | Link | Price List Currency | Options: "Currency", Read Only | Price list currency |
| `plc_conversion_rate` | Float | Price List Exchange Rate | Precision: 9 | Price list conversion rate |
| `ignore_pricing_rule` | Check | Ignore Pricing Rule | Default: 0, Permission Level: 1 | Bypass pricing rules |

#### 7. **Warehouse Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `set_warehouse` | Link | Accepted Warehouse | Options: "Warehouse" | Default accepted warehouse |
| `rejected_warehouse` | Link | Rejected Warehouse | Options: "Warehouse", Ignore User Permissions | Rejected items warehouse |
| `set_from_warehouse` | Link | Set From Warehouse | Options: "Warehouse", Depends on: "is_internal_supplier" | Source warehouse for transfers |
| `supplier_warehouse` | Link | Supplier Warehouse | Options: "Warehouse", Depends on: "is_subcontracted" | Supplier's warehouse |
| `apply_putaway_rule` | Check | Apply Putaway Rule | Default: 0 | Enable putaway rules |

#### 8. **Barcode and Scanning**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `scan_barcode` | Data | Scan Barcode | Options: "Barcode" | Barcode scanning input |

#### 9. **Items and Line Items**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `items` | Table | Items | Required, Options: "Purchase Receipt Item", Allow Bulk Edit | Line items table |

#### 10. **Quantity Totals**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `total_qty` | Float | Total Quantity | Read Only | Sum of all item quantities |
| `total_net_weight` | Float | Total Net Weight | Read Only, Depends on: "total_net_weight" | Total weight of items |

#### 11. **Amount Totals (Company Currency)**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `base_total` | Currency | Total (Company Currency) | Read Only, Options: "Company:company:default_currency" | Total before taxes (company currency) |
| `base_net_total` | Currency | Net Total (Company Currency) | Required, Read Only, Options: "Company:company:default_currency" | Net total (company currency) |
| `base_grand_total` | Currency | Grand Total (Company Currency) | Read Only, Options: "Company:company:default_currency" | Final total (company currency) |
| `base_rounded_total` | Currency | Rounded Total (Company Currency) | Read Only, Options: "Company:company:default_currency" | Rounded final total (company currency) |
| `base_rounding_adjustment` | Currency | Rounding Adjustment (Company Currency) | Read Only, Options: "Company:company:default_currency" | Rounding adjustment (company currency) |
| `base_in_words` | Data | In Words (Company Currency) | Read Only, Length: 240 | Amount in words (company currency) |

#### 12. **Amount Totals (Transaction Currency)**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `total` | Currency | Total | Read Only, Options: "currency" | Total before taxes (transaction currency) |
| `net_total` | Currency | Net Total | Read Only, Options: "currency" | Net total (transaction currency) |
| `grand_total` | Currency | Grand Total | Read Only, Options: "currency", In List View | Final total (transaction currency) |
| `rounded_total` | Currency | Rounded Total | Read Only, Options: "currency" | Rounded final total (transaction currency) |
| `rounding_adjustment` | Currency | Rounding Adjustment | Read Only, Options: "currency" | Rounding adjustment (transaction currency) |
| `in_words` | Data | In Words | Read Only, Length: 240 | Amount in words (transaction currency) |
| `disable_rounded_total` | Check | Disable Rounded Total | Default: 0 | Disable rounding |

#### 13. **Tax Withholding**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `tax_withholding_net_total` | Currency | Tax Withholding Net Total | Hidden, Read Only, Options: "currency" | Net total after tax withholding |
| `base_tax_withholding_net_total` | Currency | Base Tax Withholding Net Total | Hidden, Read Only | Net total after withholding (company currency) |

#### 14. **Tax and Charges Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `tax_category` | Link | Tax Category | Options: "Tax Category" | Tax category reference |
| `taxes_and_charges` | Link | Purchase Taxes and Charges Template | Options: "Purchase Taxes and Charges Template" | Tax template reference |
| `taxes` | Table | Purchase Taxes and Charges | Options: "Purchase Taxes and Charges" | Tax line items |
| `shipping_rule` | Link | Shipping Rule | Options: "Shipping Rule" | Shipping rule reference |
| `incoterm` | Link | Incoterm | Options: "Incoterm" | International commercial terms |
| `named_place` | Data | Named Place | Depends on: "incoterm" | Named place for incoterm |

#### 15. **Tax Totals (Company Currency)**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `base_taxes_and_charges_added` | Currency | Taxes and Charges Added (Company Currency) | Read Only, Options: "Company:company:default_currency" | Added taxes (company currency) |
| `base_taxes_and_charges_deducted` | Currency | Taxes and Charges Deducted (Company Currency) | Read Only, Options: "Company:company:default_currency" | Deducted taxes (company currency) |
| `base_total_taxes_and_charges` | Currency | Total Taxes and Charges (Company Currency) | Read Only, Options: "Company:company:default_currency" | Total taxes (company currency) |

#### 16. **Tax Totals (Transaction Currency)**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `taxes_and_charges_added` | Currency | Taxes and Charges Added | Read Only, Options: "currency" | Added taxes (transaction currency) |
| `taxes_and_charges_deducted` | Currency | Taxes and Charges Deducted | Read Only, Options: "currency" | Deducted taxes (transaction currency) |
| `total_taxes_and_charges` | Currency | Total Taxes and Charges | Read Only, Options: "currency" | Total taxes (transaction currency) |

#### 17. **Additional Discount**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `apply_discount_on` | Select | Apply Additional Discount On | Default: "Grand Total", Options: "Grand Total", "Net Total" | Discount application base |
| `additional_discount_percentage` | Float | Additional Discount Percentage | | Additional discount percentage |
| `base_discount_amount` | Currency | Additional Discount Amount (Company Currency) | Read Only, Options: "Company:company:default_currency" | Discount amount (company currency) |
| `discount_amount` | Currency | Additional Discount Amount | Options: "currency" | Discount amount (transaction currency) |

#### 18. **Calculation and Analysis**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `other_charges_calculation` | Text Editor | Taxes and Charges Calculation | Read Only | Tax calculation breakdown |
| `pricing_rules` | Table | Pricing Rule Detail | Read Only, Options: "Pricing Rule Detail" | Applied pricing rules |

#### 19. **Subcontracting Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `is_subcontracted` | Check | Is Subcontracted | Default: 0, Read Only | Subcontracting flag |
| `supplied_items` | Table | Consumed Items | Options: "Purchase Receipt Item Supplied" | Raw materials consumed |
| `get_current_stock` | Button | Get Current Stock | Depends on: "supplied_items" | Fetch current stock levels |
| `is_old_subcontracting_flow` | Check | Is Old Subcontracting Flow | Default: 0, Hidden, Read Only | Legacy subcontracting flag |

#### 20. **Address Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `shipping_address` | Link | Shipping Address Template | Options: "Address" | Shipping address reference |
| `shipping_address_display` | Text Editor | Shipping Address | Read Only | Formatted shipping address |
| `dispatch_address` | Link | Dispatch Address Template | Options: "Address" | Dispatch address reference |
| `dispatch_address_display` | Text Editor | Dispatch Address | Read Only | Formatted dispatch address |
| `billing_address` | Link | Billing Address | Options: "Address" | Billing address reference |
| `billing_address_display` | Text Editor | Billing Address | Read Only | Formatted billing address |

#### 21. **Terms and Conditions**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `tc_name` | Link | Terms | Options: "Terms and Conditions" | Terms template reference |
| `terms` | Text Editor | Terms and Conditions | | Terms and conditions text |

#### 22. **Status and Progress**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `status` | Select | Status | Required, Read Only, Default: "Draft", Options: "Draft", "Partly Billed", "To Bill", "Completed", "Return Issued", "Cancelled", "Closed" | Document status |
| `per_billed` | Percent | % Amount Billed | Read Only | Percentage billed |
| `per_returned` | Percent | % Returned | Read Only, In List View | Percentage returned |

#### 23. **Auto Repeat and Subscription**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `auto_repeat` | Link | Auto Repeat | Read Only, Options: "Auto Repeat" | Auto repeat configuration |

#### 24. **Printing and Display**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `letter_head` | Link | Letter Head | Allow on Submit, Options: "Letter Head" | Letter head reference |
| `select_print_heading` | Link | Print Heading | Allow on Submit, Options: "Print Heading" | Print heading reference |
| `language` | Data | Print Language | Read Only | Print language |
| `group_same_items` | Check | Group same items | Default: 0, Allow on Submit | Group similar items in print |

#### 25. **Transportation Details**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `transporter_name` | Data | Transporter Name | | Transporter company name |
| `lr_no` | Data | Vehicle Number | | Vehicle/lorry receipt number |
| `lr_date` | Date | Vehicle Date | | Vehicle/lorry receipt date |

#### 26. **Internal and Inter-Company**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `is_internal_supplier` | Check | Is Internal Supplier | Default: 0, Read Only, Fetch From: "supplier.is_internal_supplier" | Internal supplier flag |
| `represents_company` | Link | Represents Company | Read Only, Options: "Company", Fetch From: "supplier.represents_company" | Company represented by supplier |

#### 27. **Additional Information**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `instructions` | Small Text | Instructions | | Special instructions |
| `remarks` | Small Text | Remarks | | Additional remarks |
| `range` | Data | Range | Hidden | Legacy field |
| `other_details` | HTML | Other Details | Hidden | Legacy HTML field |

---

## Purchase Receipt Item (Child Document)
**Table:** `tabPurchase Receipt Item`
**Module:** Stock
**Auto Name:** hash
**Is Table:** Yes (Child table)

### Complete Field Schema

#### 1. **Parent Document Reference**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `parent` | VARCHAR(140) | Parent | Foreign Key to Purchase Receipt | Parent document reference |
| `parenttype` | VARCHAR(50) | Parent Type | Always "Purchase Receipt" | Parent document type |
| `parentfield` | VARCHAR(50) | Parent Field | Always "items" | Parent field name |
| `idx` | INT | Index | Line sequence number | Display order |

#### 2. **Item Identification and Scanning**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `barcode` | Data | Barcode | | Item barcode |
| `has_item_scanned` | Check | Has Item Scanned | Default: 0, Read Only, Depends on: "barcode" | Scanning status flag |
| `item_code` | Link | Item Code | Required, Bold, Options: "Item", Search Index, In List View | Item master reference |
| `supplier_part_no` | Data | Supplier Part Number | Hidden, Read Only | Supplier's part number |
| `item_name` | Data | Item Name | Required | Item display name |
| `product_bundle` | Link | Product Bundle | Read Only, Options: "Product Bundle" | Product bundle reference |

#### 3. **Item Details and Description**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `description` | Text Editor | Description | | Item description |
| `brand` | Link | Brand | Hidden, Read Only, Options: "Brand", Fetch From: "item_code.brand" | Brand reference |
| `item_group` | Link | Item Group | Read Only, Options: "Item Group", Fetch From: "item_code.item_group" | Item group reference |
| `image` | Attach | Image | Hidden, Fetch From: "item_code.image" | Item image |
| `image_view` | Image | Image View | Options: "image" | Image display |

#### 4. **Quantity Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `received_qty` | Float | Received Quantity | Required, Default: 0, Read Only, Bold | Total quantity received |
| `qty` | Float | Accepted Quantity | In List View | Accepted quantity |
| `rejected_qty` | Float | Rejected Quantity | In List View | Rejected quantity |
| `returned_qty` | Float | Returned Qty in Stock UOM | Read Only, Depends on: "returned_qty" | Previously returned quantity |
| `stock_qty` | Float | Accepted Qty in Stock UOM | Read Only, Depends on: "eval:doc.uom != doc.stock_uom" | Quantity in stock UOM |
| `received_stock_qty` | Float | Received Qty in Stock UOM | Read Only, Depends on: "eval:doc.uom != doc.stock_uom" | Received quantity in stock UOM |

#### 5. **Unit of Measure (UOM)**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `uom` | Link | UOM | Required, Options: "UOM" | Transaction unit of measure |
| `stock_uom` | Link | Stock UOM | Required, Read Only, Options: "UOM", Depends on: "eval:doc.uom != doc.stock_uom" | Stock unit of measure |
| `conversion_factor` | Float | Conversion Factor | Required, Depends on: "eval:doc.uom != doc.stock_uom" | UOM conversion factor |

#### 6. **Sample Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `retain_sample` | Check | Retain Sample | Default: 0, Read Only, Fetch From: "item_code.retain_sample" | Sample retention flag |
| `sample_quantity` | Int | Sample Quantity | Depends on: "retain_sample", Fetch From: "item_code.sample_quantity" | Sample quantity to retain |

#### 7. **Pricing and Rates**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `price_list_rate` | Currency | Price List Rate | Options: "currency" | Standard price from price list |
| `base_price_list_rate` | Currency | Price List Rate (Company Currency) | Read Only, Options: "Company:company:default_currency" | Price list rate in company currency |
| `rate` | Currency | Rate | Bold, In List View, Options: "currency" | Unit rate |
| `base_rate` | Currency | Rate (Company Currency) | Required, Read Only, Options: "Company:company:default_currency" | Unit rate in company currency |
| `amount` | Currency | Amount | Read Only, In List View, Options: "currency" | Line amount |
| `base_amount` | Currency | Amount (Company Currency) | Read Only, Options: "Company:company:default_currency" | Line amount in company currency |
| `stock_uom_rate` | Currency | Rate of Stock UOM | Read Only, Options: "currency", Depends on: "eval: doc.uom != doc.stock_uom" | Rate per stock UOM |

#### 8. **Discount and Margin**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `margin_type` | Select | Margin Type | Options: "Percentage", "Amount", Depends on: "price_list_rate" | Margin calculation type |
| `margin_rate_or_amount` | Float | Margin Rate or Amount | Depends on: "eval:doc.margin_type && doc.price_list_rate" | Margin value |
| `rate_with_margin` | Currency | Rate With Margin | Read Only, Options: "currency" | Rate including margin |
| `base_rate_with_margin` | Currency | Rate With Margin (Company Currency) | Options: "Company:company:default_currency" | Rate with margin in company currency |
| `discount_percentage` | Percent | Discount on Price List Rate (%) | Depends on: "price_list_rate" | Discount percentage |
| `discount_amount` | Currency | Discount Amount | Options: "currency", Depends on: "price_list_rate" | Absolute discount amount |
| `distributed_discount_amount` | Currency | Distributed Discount Amount | Options: "currency" | Distributed discount from header |

#### 9. **Net Amounts**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `net_rate` | Currency | Net Rate | Read Only, Options: "currency" | Net rate after discounts |
| `base_net_rate` | Currency | Net Rate (Company Currency) | Read Only, Options: "Company:company:default_currency" | Net rate in company currency |
| `net_amount` | Currency | Net Amount | Read Only, In List View, Options: "currency" | Net amount after discounts |
| `base_net_amount` | Currency | Net Amount (Company Currency) | Read Only, Options: "Company:company:default_currency" | Net amount in company currency |

#### 10. **Valuation and Costing**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `valuation_rate` | Currency | Valuation Rate | Read Only, Hidden, Precision: 6, Options: "Company:company:default_currency" | Stock valuation rate |
| `sales_incoming_rate` | Currency | Sales Incoming Rate | Hidden, Read Only, Options: "Company:company:default_currency" | Rate for internal transfers |
| `landed_cost_voucher_amount` | Currency | Landed Cost Voucher Amount | Read Only, Allow on Submit, Options: "Company:company:default_currency" | Landed cost allocation |
| `rm_supp_cost` | Currency | Raw Materials Supplied Cost | Hidden, Read Only, Options: "Company:company:default_currency" | Subcontracted material cost |

#### 11. **Tax Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `item_tax_template` | Link | Item Tax Template | Options: "Item Tax Template" | Item-specific tax template |
| `item_tax_rate` | Code | Item Tax Rate | Hidden, Read Only | Item tax rate JSON |
| `item_tax_amount` | Currency | Item Tax Amount Included in Value | Hidden, Read Only, Options: "Company:company:default_currency" | Tax amount included |
| `apply_tds` | Check | Apply TDS | Default: 1, Hidden, Read Only | TDS application flag |

#### 12. **Warehouse and Location**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `warehouse` | Link | Accepted Warehouse | Bold, In List View, Options: "Warehouse" | Receiving warehouse |
| `rejected_warehouse` | Link | Rejected Warehouse | Options: "Warehouse" | Rejected items warehouse |
| `from_warehouse` | Link | From Warehouse | Hidden, Options: "Warehouse", Depends on: "eval:parent.is_internal_supplier" | Source warehouse for transfers |
| `putaway_rule` | Link | Putaway Rule | Read Only, Options: "Putaway Rule" | Applied putaway rule |

#### 13. **Quality and Inspection**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `quality_inspection` | Link | Quality Inspection | Options: "Quality Inspection", Depends on: "eval:!doc.__islocal" | Quality inspection reference |
| `schedule_date` | Date | Required By | Read Only | Expected delivery date |
| `allow_zero_valuation_rate` | Check | Allow Zero Valuation Rate | Default: 0 | Allow zero valuation |

#### 14. **Document References and Traceability**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `material_request` | Link | Material Request | Read Only, Options: "Material Request" | Source material request |
| `material_request_item` | Data | Material Request Item | Hidden, Read Only | Source MR item reference |
| `purchase_order` | Link | Purchase Order | Read Only, Search Index, Options: "Purchase Order" | **Source purchase order** |
| `purchase_order_item` | Data | Purchase Order Item | Hidden, Read Only, Search Index | **Source PO item reference** |
| `purchase_invoice` | Link | Purchase Invoice | Read Only, Options: "Purchase Invoice" | Related purchase invoice |
| `purchase_invoice_item` | Data | Purchase Invoice Item | Hidden, Read Only, Search Index | Related PI item reference |
| `purchase_receipt_item` | Data | Purchase Receipt Item | Hidden, Read Only, Search Index | Return reference |
| `delivery_note_item` | Data | Delivery Note Item | Read Only, Search Index | Inter-company delivery reference |
| `sales_order` | Link | Sales Order | Read Only, Search Index, Options: "Sales Order" | Related sales order |
| `sales_order_item` | Data | Sales Order Item | Hidden, Read Only, Search Index | Related SO item reference |

#### 15. **Serial and Batch Tracking**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `use_serial_batch_fields` | Check | Use Serial No / Batch Fields | Default: 0 | Use legacy serial/batch fields |
| `serial_and_batch_bundle` | Link | Serial and Batch Bundle | Search Index, Options: "Serial and Batch Bundle" | Serial/batch bundle (new system) |
| `rejected_serial_and_batch_bundle` | Link | Rejected Serial and Batch Bundle | Options: "Serial and Batch Bundle" | Rejected items bundle |
| `add_serial_batch_bundle` | Button | Add Serial / Batch No | Depends on: "eval:doc.use_serial_batch_fields === 0" | Add serial/batch button |
| `add_serial_batch_for_rejected_qty` | Button | Add Serial / Batch No (Rejected Qty) | Depends on: "eval:doc.use_serial_batch_fields === 0" | Add rejected serial/batch |
| `serial_no` | Text | Serial No | | Serial numbers (legacy) |
| `rejected_serial_no` | Text | Rejected Serial No | | Rejected serial numbers (legacy) |
| `batch_no` | Link | Batch No | Search Index, Options: "Batch" | Batch number (legacy) |

#### 16. **Weight Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `weight_per_unit` | Float | Weight Per Unit | | Weight per unit |
| `total_weight` | Float | Total Weight | Read Only | Total weight calculation |
| `weight_uom` | Link | Weight UOM | Options: "UOM" | Weight unit of measure |

#### 17. **Manufacturing and BOM**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `bom` | Link | BOM | Read Only, Options: "BOM", Depends on: "eval:parent.is_old_subcontracting_flow" | Bill of materials reference |
| `include_exploded_items` | Check | Include Exploded Items | Default: 0, Read Only, Depends on: "eval:parent.is_subcontracted" | Include BOM components |

#### 18. **Manufacturer Information**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `manufacturer` | Link | Manufacturer | Options: "Manufacturer" | Manufacturer reference |
| `manufacturer_part_no` | Data | Manufacturer Part Number | | Manufacturer's part number |

#### 19. **Asset Management**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `is_fixed_asset` | Check | Is Fixed Asset | Default: 0, Hidden, Read Only | Fixed asset flag |
| `asset_category` | Link | Asset Category | Read Only, Options: "Asset Category", Depends on: "is_fixed_asset" | Asset category |
| `asset_location` | Link | Asset Location | Options: "Location", Depends on: "is_fixed_asset" | Asset location |
| `wip_composite_asset` | Link | WIP Composite Asset | Options: "Asset" | Work-in-progress asset |

#### 20. **Accounting Integration**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `expense_account` | Link | Expense Account | Options: "Account" | Expense account for posting |
| `provisional_expense_account` | Link | Provisional Expense Account | Options: "Account" | Provisional expense account |
| `cost_center` | Link | Cost Center | Options: "Cost Center", Default: ":Company" | Cost center allocation |
| `project` | Link | Project | Options: "Project" | Project allocation |

#### 21. **Pricing Rules and Free Items**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `pricing_rules` | Small Text | Pricing Rules | Hidden, Read Only | Applied pricing rules |
| `is_free_item` | Check | Is Free Item | Default: 0, Read Only | Free item flag |

#### 22. **Billing and Payment Tracking**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `billed_amt` | Currency | Billed Amt | Read Only, Options: "currency" | Amount already billed |
| `amount_difference_with_purchase_invoice` | Currency | Amount Difference with Purchase Invoice | Read Only | Difference with invoice amount |

#### 23. **Returns and Adjustments**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `return_qty_from_rejected_warehouse` | Check | Return Qty from Rejected Warehouse | Default: 0, Read Only | Return from rejected warehouse |

#### 24. **Subcontracting Details**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `subcontracting_receipt_item` | Data | Subcontracting Receipt Item | Hidden, Read Only, Search Index | Subcontracting receipt reference |

#### 25. **Print and Display**

| Field Name | Field Type | Label | Properties | Business Purpose |
|------------|------------|-------|------------|------------------|
| `page_break` | Check | Page Break | Default: 0, Allow on Submit | Page break in print |

---

## Critical Relationships

### **Purchase Order Integration**
The most important relationship fields linking Purchase Receipt to Purchase Order:

| Purchase Receipt Item Field | Target | Purpose |
|----------------------------|--------|---------|
| `purchase_order` | `tabPurchase Order.name` | Links to source Purchase Order |
| `purchase_order_item` | `tabPurchase Order Item.name` | Links to specific PO line item |

### **Material Request Traceability**
| Purchase Receipt Item Field | Target | Purpose |
|----------------------------|--------|---------|
| `material_request` | `tabMaterial Request.name` | Links to original demand |
| `material_request_item` | `tabMaterial Request Item.name` | Links to specific MR line item |

### **Invoice Matching (Three-Way Matching)**
| Purchase Receipt Item Field | Target | Purpose |
|----------------------------|--------|---------|
| `purchase_invoice` | `tabPurchase Invoice.name` | Links to related invoice |
| `purchase_invoice_item` | `tabPurchase Invoice Item.name` | Links to specific invoice line |

---

## Database Constraints and Indexes

### **Primary Keys**
- Purchase Receipt: `name` (VARCHAR(140))
- Purchase Receipt Item: Auto-generated hash

### **Foreign Key Relationships**
```sql
-- Core References
FOREIGN KEY (supplier) REFERENCES tabSupplier(name)
FOREIGN KEY (company) REFERENCES tabCompany(name)
FOREIGN KEY (currency) REFERENCES tabCurrency(name)

-- Item Level References  
FOREIGN KEY (parent) REFERENCES `tabPurchase Receipt`(name)
FOREIGN KEY (item_code) REFERENCES tabItem(name)
FOREIGN KEY (warehouse) REFERENCES tabWarehouse(name)

-- Critical Purchase Order Links
FOREIGN KEY (purchase_order) REFERENCES `tabPurchase Order`(name)
FOREIGN KEY (purchase_order_item) REFERENCES `tabPurchase Order Item`(name)
```

### **Search Indexes**
Critical indexes for performance:
- `purchase_order` (Search Index)
- `purchase_order_item` (Search Index) 
- `material_request_item` (Search Index)
- `item_code` (Search Index)
- `batch_no` (Search Index)

---

## Field Dependencies and Business Logic

### **Conditional Field Display**
Many fields have `depends_on` conditions that control when they appear:

- **UOM Fields**: Show only when `eval:doc.uom != doc.stock_uom`
- **Return Fields**: Show only when `is_return = 1`
- **Subcontracting**: Show only when `is_subcontracted = 1`
- **Asset Fields**: Show only when `is_fixed_asset = 1`
- **Quality Fields**: Show only when quality inspection is required

### **Auto-Calculation Fields**
Several fields are automatically calculated:
- `stock_qty` = `qty` × `conversion_factor`
- `amount` = `qty` × `rate`
- `base_amount` = `amount` × `conversion_rate`
- `total_weight` = `qty` × `weight_per_unit`

### **Read-Only Fields**
Many fields are automatically populated and read-only:
- All `base_*` fields (company currency amounts)
- `supplier_name`, `item_name` (fetched from masters)
- Status and percentage fields
- Calculated totals and amounts

---

This comprehensive schema provides the exact field structure as implemented in ERPNext, enabling accurate database design and development of Purchase Receipt functionality.