# Simple Purchase Order - API Documentation

## Overview

This document provides comprehensive API specifications for the Simple Purchase Order module. The API follows REST principles and supports JSON data exchange for all operations.

## Base Configuration

### Base URL
```
https://api.yourcompany.com/v1
```

### Authentication
```http
Authorization: Bearer <access_token>
```

### Content Type
```http
Content-Type: application/json
```

### Response Format
All responses follow a consistent format:
```json
{
  "success": true,
  "data": {...},
  "message": "Operation completed successfully",
  "errors": []
}
```

## Purchase Order API Endpoints

### 1. Create Purchase Order

**Endpoint:** `POST /purchase-orders`

**Description:** Create a new purchase order with line items

**Request Body:**
```json
{
  "supplier_id": "SUPP-001",
  "company_id": "COMP-001",
  "transaction_date": "2025-06-18",
  "required_by_date": "2025-07-01",
  "currency": "USD",
  "conversion_rate": 1.0,
  "tax_rate": 10.0,
  "supplier_address_id": "ADDR-001",
  "contact_person_id": "CONT-001",
  "remarks": "Urgent delivery required",
  "items": [
    {
      "item_code": "ITEM-001",
      "qty": 10,
      "rate": 50.0,
      "warehouse_id": "WH-001",
      "required_by_date": "2025-07-01"
    },
    {
      "item_code": "ITEM-002", 
      "qty": 5,
      "rate": 100.0,
      "warehouse_id": "WH-001",
      "required_by_date": "2025-07-01"
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "SPO-2025-0001",
    "naming_series": "SPO-.YYYY.-",
    "supplier_id": "SUPP-001",
    "supplier_name": "ABC Suppliers Ltd",
    "company_id": "COMP-001",
    "transaction_date": "2025-06-18",
    "required_by_date": "2025-07-01",
    "currency": "USD",
    "conversion_rate": 1.0,
    "status": "Draft",
    "docstatus": 0,
    "net_total": 1000.00,
    "tax_rate": 10.0,
    "tax_amount": 100.00,
    "grand_total": 1100.00,
    "created_at": "2025-06-18T10:30:00Z",
    "created_by": "user@company.com",
    "items": [
      {
        "id": "SPO-2025-0001-1",
        "item_code": "ITEM-001",
        "item_name": "Widget A",
        "qty": 10,
        "uom": "Nos",
        "rate": 50.0,
        "amount": 500.00,
        "warehouse_id": "WH-001",
        "line_sequence": 1
      },
      {
        "id": "SPO-2025-0001-2", 
        "item_code": "ITEM-002",
        "item_name": "Widget B",
        "qty": 5,
        "uom": "Nos", 
        "rate": 100.0,
        "amount": 500.00,
        "warehouse_id": "WH-001",
        "line_sequence": 2
      }
    ]
  },
  "message": "Purchase Order created successfully"
}
```

**Error Response (400 Bad Request):**
```json
{
  "success": false,
  "data": null,
  "message": "Validation failed",
  "errors": [
    {
      "field": "items[0].qty",
      "message": "Quantity must be greater than 0"
    },
    {
      "field": "supplier_id",
      "message": "Supplier is required"
    }
  ]
}
```

### 2. Get Purchase Order List

**Endpoint:** `GET /purchase-orders`

**Description:** Retrieve paginated list of purchase orders with filtering

**Query Parameters:**
- `company_id` (required): Company filter
- `supplier_id` (optional): Filter by supplier
- `status` (optional): Filter by status (Draft, Submitted, Completed, etc.)
- `date_from` (optional): Start date filter (YYYY-MM-DD)
- `date_to` (optional): End date filter (YYYY-MM-DD)
- `limit` (optional): Page size (default: 20, max: 100)
- `offset` (optional): Page offset (default: 0)
- `sort_by` (optional): Sort field (transaction_date, grand_total, etc.)
- `sort_order` (optional): Sort direction (asc, desc)

**Example Request:**
```http
GET /purchase-orders?company_id=COMP-001&status=Draft&limit=10&offset=0&sort_by=transaction_date&sort_order=desc
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "orders": [
      {
        "id": "SPO-2025-0001",
        "supplier_id": "SUPP-001",
        "supplier_name": "ABC Suppliers Ltd",
        "transaction_date": "2025-06-18",
        "required_by_date": "2025-07-01",
        "currency": "USD",
        "grand_total": 1100.00,
        "status": "Draft",
        "created_at": "2025-06-18T10:30:00Z"
      }
    ],
    "pagination": {
      "total_count": 45,
      "limit": 10,
      "offset": 0,
      "has_next": true,
      "has_previous": false
    }
  },
  "message": "Purchase orders retrieved successfully"
}
```

### 3. Get Single Purchase Order

**Endpoint:** `GET /purchase-orders/{id}`

**Description:** Retrieve complete purchase order with all line items

**Path Parameters:**
- `id`: Purchase order ID

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "SPO-2025-0001",
    "naming_series": "SPO-.YYYY.-",
    "supplier_id": "SUPP-001",
    "supplier_name": "ABC Suppliers Ltd",
    "company_id": "COMP-001",
    "transaction_date": "2025-06-18",
    "required_by_date": "2025-07-01",
    "currency": "USD",
    "conversion_rate": 1.0,
    "status": "Draft",
    "docstatus": 0,
    "net_total": 1000.00,
    "tax_rate": 10.0,
    "tax_amount": 100.00,
    "grand_total": 1100.00,
    "supplier_address_id": "ADDR-001",
    "contact_person_id": "CONT-001",
    "remarks": "Urgent delivery required",
    "created_at": "2025-06-18T10:30:00Z",
    "updated_at": "2025-06-18T10:30:00Z",
    "created_by": "user@company.com",
    "updated_by": "user@company.com",
    "items": [
      {
        "id": "SPO-2025-0001-1",
        "item_code": "ITEM-001",
        "item_name": "Widget A",
        "description": "High quality widget",
        "qty": 10,
        "uom": "Nos",
        "rate": 50.0,
        "amount": 500.00,
        "warehouse_id": "WH-001",
        "required_by_date": "2025-07-01",
        "line_sequence": 1
      }
    ]
  },
  "message": "Purchase order retrieved successfully"
}
```

### 4. Update Purchase Order

**Endpoint:** `PUT /purchase-orders/{id}`

**Description:** Update existing purchase order (only allowed in Draft status)

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:** Same as create request

**Response (200 OK):** Same as create response with updated data

**Error Response (409 Conflict):**
```json
{
  "success": false,
  "data": null,
  "message": "Cannot update submitted purchase order",
  "errors": [
    {
      "field": "status",
      "message": "Order is already submitted"
    }
  ]
}
```

### 5. Submit Purchase Order

**Endpoint:** `POST /purchase-orders/{id}/submit`

**Description:** Submit purchase order for processing (changes status from Draft to Submitted)

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:** Empty

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "SPO-2025-0001",
    "status": "Submitted",
    "docstatus": 1,
    "updated_at": "2025-06-18T11:00:00Z"
  },
  "message": "Purchase order submitted successfully"
}
```

### 6. Cancel Purchase Order

**Endpoint:** `POST /purchase-orders/{id}/cancel`

**Description:** Cancel submitted purchase order

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:**
```json
{
  "reason": "Supplier unavailable"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "SPO-2025-0001",
    "status": "Cancelled",
    "docstatus": 2,
    "updated_at": "2025-06-18T11:30:00Z"
  },
  "message": "Purchase order cancelled successfully"
}
```

### 7. Update Status

**Endpoint:** `PATCH /purchase-orders/{id}/status`

**Description:** Update purchase order status (for warehouse operations)

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:**
```json
{
  "status": "Received",
  "notes": "All items received in good condition"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "SPO-2025-0001",
    "status": "Received",
    "updated_at": "2025-06-18T15:00:00Z"
  },
  "message": "Status updated successfully"
}
```

### 8. Delete Purchase Order

**Endpoint:** `DELETE /purchase-orders/{id}`

**Description:** Delete purchase order (only allowed in Draft status)

**Path Parameters:**
- `id`: Purchase order ID

**Response (204 No Content):** Empty response body

## Related Document Generation APIs

### 9. Create Purchase Receipt

**Endpoint:** `POST /purchase-orders/{id}/create-receipt`

**Description:** Generate purchase receipt from purchase order

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:**
```json
{
  "posting_date": "2025-06-20",
  "items": [
    {
      "po_item_id": "SPO-2025-0001-1",
      "received_qty": 10
    },
    {
      "po_item_id": "SPO-2025-0001-2", 
      "received_qty": 5
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "receipt_id": "PR-2025-0001",
    "purchase_order_id": "SPO-2025-0001",
    "posting_date": "2025-06-20",
    "total_received": 1100.00
  },
  "message": "Purchase receipt created successfully"
}
```

### 10. Create Purchase Invoice

**Endpoint:** `POST /purchase-orders/{id}/create-invoice`

**Description:** Generate purchase invoice from purchase order

**Path Parameters:**
- `id`: Purchase order ID

**Request Body:**
```json
{
  "posting_date": "2025-06-22",
  "due_date": "2025-07-22",
  "items": [
    {
      "po_item_id": "SPO-2025-0001-1",
      "billed_qty": 10
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "invoice_id": "PI-2025-0001",
    "purchase_order_id": "SPO-2025-0001", 
    "posting_date": "2025-06-22",
    "total_amount": 550.00
  },
  "message": "Purchase invoice created successfully"
}
```

## Master Data APIs

### 11. Get Suppliers

**Endpoint:** `GET /suppliers`

**Query Parameters:**
- `active_only` (optional): Filter active suppliers (default: true)
- `search` (optional): Search by name
- `limit` (optional): Page size

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "SUPP-001",
      "supplier_name": "ABC Suppliers Ltd",
      "default_currency": "USD",
      "is_active": true
    }
  ]
}
```

### 12. Get Items

**Endpoint:** `GET /items`

**Query Parameters:**
- `purchase_only` (optional): Filter purchasable items (default: true)
- `search` (optional): Search by name or code
- `limit` (optional): Page size

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "ITEM-001",
      "item_name": "Widget A",
      "description": "High quality widget",
      "stock_uom": "Nos",
      "last_purchase_rate": 45.0
    }
  ]
}
```

### 13. Get Warehouses

**Endpoint:** `GET /warehouses`

**Query Parameters:**
- `company_id` (required): Company filter
- `active_only` (optional): Filter active warehouses

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "WH-001",
      "warehouse_name": "Main Warehouse",
      "company_id": "COMP-001"
    }
  ]
}
```

## Business Logic APIs

### 14. Get Supplier Details

**Endpoint:** `GET /suppliers/{id}/details`

**Description:** Get supplier details with default values for purchase order

**Path Parameters:**
- `id`: Supplier ID

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "SUPP-001",
    "supplier_name": "ABC Suppliers Ltd",
    "default_currency": "USD",
    "payment_terms_id": "NET-30",
    "addresses": [
      {
        "id": "ADDR-001",
        "address_type": "Billing",
        "address_line1": "123 Supplier Street"
      }
    ],
    "contacts": [
      {
        "id": "CONT-001",
        "contact_name": "John Doe",
        "email": "john@abcsuppliers.com"
      }
    ]
  }
}
```

### 15. Calculate Order Totals

**Endpoint:** `POST /purchase-orders/calculate-totals`

**Description:** Calculate totals for purchase order items (used for real-time calculations)

**Request Body:**
```json
{
  "items": [
    {
      "qty": 10,
      "rate": 50.0
    },
    {
      "qty": 5,
      "rate": 100.0
    }
  ],
  "tax_rate": 10.0,
  "currency": "USD"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "net_total": 1000.00,
    "tax_amount": 100.00,
    "grand_total": 1100.00,
    "item_totals": [
      {"line": 1, "amount": 500.00},
      {"line": 2, "amount": 500.00}
    ]
  }
}
```

## Error Handling

### HTTP Status Codes
- `200` - OK: Request successful
- `201` - Created: Resource created successfully
- `204` - No Content: Resource deleted successfully
- `400` - Bad Request: Invalid request data
- `401` - Unauthorized: Authentication required
- `403` - Forbidden: Insufficient permissions
- `404` - Not Found: Resource not found
- `409` - Conflict: Business rule violation
- `422` - Unprocessable Entity: Validation errors
- `500` - Internal Server Error: System error

### Error Response Structure
```json
{
  "success": false,
  "data": null,
  "message": "Validation failed",
  "errors": [
    {
      "field": "supplier_id",
      "code": "REQUIRED",
      "message": "Supplier is required"
    },
    {
      "field": "items[0].qty",
      "code": "MIN_VALUE",
      "message": "Quantity must be greater than 0"
    }
  ]
}
```

### Common Error Codes
- `REQUIRED` - Required field missing
- `INVALID_FORMAT` - Invalid data format
- `MIN_VALUE` - Value below minimum
- `MAX_VALUE` - Value above maximum
- `NOT_FOUND` - Referenced resource not found
- `ALREADY_EXISTS` - Duplicate resource
- `INVALID_STATUS` - Invalid status transition
- `PERMISSION_DENIED` - Insufficient permissions

## Rate Limiting

### Limits
- `100 requests per minute` for read operations
- `50 requests per minute` for write operations
- `10 requests per minute` for bulk operations

### Rate Limit Headers
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 85
X-RateLimit-Reset: 1640995200
```

## Pagination

### Request Parameters
- `limit`: Number of records per page (max: 100)
- `offset`: Number of records to skip

### Response Format
```json
{
  "data": [...],
  "pagination": {
    "total_count": 245,
    "limit": 20,
    "offset": 40,
    "has_next": true,
    "has_previous": true
  }
}
```

## Webhooks

### Purchase Order Events
Configure webhooks to receive notifications for:
- `purchase_order.created`
- `purchase_order.submitted`
- `purchase_order.cancelled`
- `purchase_order.status_changed`

### Webhook Payload
```json
{
  "event": "purchase_order.submitted",
  "data": {
    "id": "SPO-2025-0001",
    "supplier_id": "SUPP-001",
    "grand_total": 1100.00,
    "status": "Submitted"
  },
  "timestamp": "2025-06-18T11:00:00Z"
}
```

This API documentation provides complete specifications for implementing the Simple Purchase Order module with RESTful APIs that support all business requirements while maintaining simplicity and performance.