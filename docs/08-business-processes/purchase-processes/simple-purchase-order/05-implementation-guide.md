# Simple Purchase Order - Implementation Guide

## Overview

This guide provides step-by-step instructions for implementing the Simple Purchase Order module in any programming language or framework. It covers architecture decisions, code organization, implementation patterns, and best practices.

## Implementation Strategy

### Development Approach
1. **Database First**: Start with schema and relationships
2. **API First**: Design REST endpoints before UI
3. **Test Driven**: Write tests for business logic
4. **Incremental**: Build in phases with working deliverables
5. **Documentation**: Maintain code documentation throughout

### Technology Stack Recommendations

#### Backend Technologies
**Option 1: Python/Django**
- Django REST Framework for API
- PostgreSQL for database
- Celery for background tasks

**Option 2: Node.js/Express**
- Express.js with TypeScript
- Sequelize ORM
- MySQL/PostgreSQL database

**Option 3: Java/Spring Boot**
- Spring Boot with JPA
- MySQL/PostgreSQL database
- Spring Security for authentication

**Option 4: C#/.NET Core**
- ASP.NET Core Web API
- Entity Framework Core
- SQL Server/PostgreSQL database

#### Frontend Technologies
- **React** with TypeScript for web applications
- **Vue.js** for simpler component structure
- **Angular** for enterprise applications
- **Mobile**: React Native or Flutter

## Phase 1: Database Implementation

### Step 1.1: Create Database Schema

**PostgreSQL Example:**
```sql
-- Create database
CREATE DATABASE simple_purchase_order;

-- Create main tables
\i database-schema.sql

-- Create indexes
\i database-indexes.sql

-- Insert initial data
\i sample-data.sql
```

**MySQL Example:**
```sql
CREATE DATABASE simple_purchase_order 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

USE simple_purchase_order;
SOURCE database-schema.sql;
SOURCE database-indexes.sql;
SOURCE sample-data.sql;
```

### Step 1.2: Database Connection Configuration

**Python/Django settings.py:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'simple_purchase_order',
        'USER': 'app_user',
        'PASSWORD': 'secure_password',
        'HOST': 'localhost',
        'PORT': '5432',
        'OPTIONS': {
            'charset': 'utf8mb4',
        }
    }
}
```

**Node.js/Sequelize config:**
```javascript
const config = {
  development: {
    username: 'app_user',
    password: 'secure_password',
    database: 'simple_purchase_order',
    host: '127.0.0.1',
    dialect: 'postgresql',
    logging: console.log
  }
};
```

### Step 1.3: Create Model Classes

**Python/Django Models:**
```python
from django.db import models
from django.core.validators import MinValueValidator
from decimal import Decimal

class PurchaseOrder(models.Model):
    DRAFT = 'Draft'
    SUBMITTED = 'Submitted'
    CANCELLED = 'Cancelled'
    RECEIVED = 'Received'
    COMPLETED = 'Completed'
    
    STATUS_CHOICES = [
        (DRAFT, 'Draft'),
        (SUBMITTED, 'Submitted'),
        (CANCELLED, 'Cancelled'),
        (RECEIVED, 'Received'),
        (COMPLETED, 'Completed'),
    ]
    
    id = models.CharField(max_length=50, primary_key=True)
    naming_series = models.CharField(max_length=20, default='SPO-.YYYY.-')
    supplier_id = models.CharField(max_length=50)
    supplier_name = models.CharField(max_length=150, blank=True)
    company_id = models.CharField(max_length=50)
    transaction_date = models.DateField()
    required_by_date = models.DateField(null=True, blank=True)
    currency = models.CharField(max_length=3, default='USD')
    conversion_rate = models.DecimalField(
        max_digits=10, decimal_places=9, 
        validators=[MinValueValidator(Decimal('0.000000001'))]
    )
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default=DRAFT)
    docstatus = models.SmallIntegerField(default=0)
    net_total = models.DecimalField(max_digits=15, decimal_places=2, default=0)
    tax_rate = models.DecimalField(max_digits=5, decimal_places=2, default=0)
    tax_amount = models.DecimalField(max_digits=15, decimal_places=2, default=0)
    grand_total = models.DecimalField(max_digits=15, decimal_places=2, default=0)
    remarks = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.CharField(max_length=50)
    updated_by = models.CharField(max_length=50)
    
    class Meta:
        db_table = 'purchase_order'
        indexes = [
            models.Index(fields=['supplier_id', 'transaction_date']),
            models.Index(fields=['company_id', 'status']),
            models.Index(fields=['status', 'docstatus']),
        ]

class PurchaseOrderItem(models.Model):
    id = models.CharField(max_length=50, primary_key=True)
    parent = models.ForeignKey(
        PurchaseOrder, 
        on_delete=models.CASCADE, 
        related_name='items'
    )
    item_code = models.CharField(max_length=50)
    item_name = models.CharField(max_length=150, blank=True)
    description = models.TextField(blank=True)
    qty = models.DecimalField(
        max_digits=10, decimal_places=3,
        validators=[MinValueValidator(Decimal('0.001'))]
    )
    uom = models.CharField(max_length=20)
    rate = models.DecimalField(
        max_digits=15, decimal_places=4,
        validators=[MinValueValidator(Decimal('0.0001'))]
    )
    amount = models.DecimalField(max_digits=15, decimal_places=2, default=0)
    warehouse_id = models.CharField(max_length=50)
    required_by_date = models.DateField(null=True, blank=True)
    line_sequence = models.IntegerField(default=1)
    
    class Meta:
        db_table = 'purchase_order_item'
        unique_together = [['parent', 'line_sequence']]
        indexes = [
            models.Index(fields=['parent_id', 'line_sequence']),
            models.Index(fields=['item_code']),
        ]
```

**Node.js/Sequelize Models:**
```javascript
const { DataTypes } = require('sequelize');

const PurchaseOrder = sequelize.define('PurchaseOrder', {
  id: {
    type: DataTypes.STRING(50),
    primaryKey: true
  },
  namingSeries: {
    type: DataTypes.STRING(20),
    defaultValue: 'SPO-.YYYY.-'
  },
  supplierId: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  supplierName: {
    type: DataTypes.STRING(150)
  },
  companyId: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  transactionDate: {
    type: DataTypes.DATEONLY,
    allowNull: false
  },
  requiredByDate: {
    type: DataTypes.DATEONLY
  },
  currency: {
    type: DataTypes.STRING(3),
    defaultValue: 'USD'
  },
  conversionRate: {
    type: DataTypes.DECIMAL(10, 9),
    defaultValue: 1.0,
    validate: { min: 0.000000001 }
  },
  status: {
    type: DataTypes.ENUM(
      'Draft', 'Submitted', 'Cancelled', 'Received', 'Completed'
    ),
    defaultValue: 'Draft'
  },
  docstatus: {
    type: DataTypes.TINYINT,
    defaultValue: 0
  },
  netTotal: {
    type: DataTypes.DECIMAL(15, 2),
    defaultValue: 0
  },
  taxRate: {
    type: DataTypes.DECIMAL(5, 2),
    defaultValue: 0
  },
  taxAmount: {
    type: DataTypes.DECIMAL(15, 2),
    defaultValue: 0
  },
  grandTotal: {
    type: DataTypes.DECIMAL(15, 2),
    defaultValue: 0
  },
  remarks: {
    type: DataTypes.TEXT
  },
  createdBy: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  updatedBy: {
    type: DataTypes.STRING(50),
    allowNull: false
  }
}, {
  tableName: 'purchase_order',
  indexes: [
    { fields: ['supplierId', 'transactionDate'] },
    { fields: ['companyId', 'status'] },
    { fields: ['status', 'docstatus'] }
  ]
});
```

## Phase 2: Business Logic Implementation

### Step 2.1: Create Service Classes

**Purchase Order Service (Python):**
```python
from decimal import Decimal
from django.db import transaction
from django.core.exceptions import ValidationError
from .models import PurchaseOrder, PurchaseOrderItem

class PurchaseOrderService:
    
    @staticmethod
    def create_purchase_order(data, user):
        """Create a new purchase order with items"""
        with transaction.atomic():
            # Generate ID
            po_id = PurchaseOrderService._generate_id(data.get('naming_series'))
            
            # Create main record
            po = PurchaseOrder.objects.create(
                id=po_id,
                supplier_id=data['supplier_id'],
                company_id=data['company_id'],
                transaction_date=data['transaction_date'],
                required_by_date=data.get('required_by_date'),
                currency=data['currency'],
                conversion_rate=data.get('conversion_rate', 1.0),
                tax_rate=data.get('tax_rate', 0),
                remarks=data.get('remarks', ''),
                created_by=user.email,
                updated_by=user.email
            )
            
            # Add items
            for idx, item_data in enumerate(data['items'], 1):
                PurchaseOrderItem.objects.create(
                    id=f"{po_id}-{idx}",
                    parent=po,
                    item_code=item_data['item_code'],
                    qty=item_data['qty'],
                    rate=item_data['rate'],
                    warehouse_id=item_data['warehouse_id'],
                    required_by_date=item_data.get('required_by_date'),
                    line_sequence=idx
                )
            
            # Calculate totals
            PurchaseOrderService._calculate_totals(po)
            po.save()
            
            return po
    
    @staticmethod
    def _calculate_totals(purchase_order):
        """Calculate and update purchase order totals"""
        net_total = Decimal('0.00')
        
        for item in purchase_order.items.all():
            item.amount = item.qty * item.rate
            item.save()
            net_total += item.amount
        
        purchase_order.net_total = net_total
        purchase_order.tax_amount = net_total * purchase_order.tax_rate / 100
        purchase_order.grand_total = net_total + purchase_order.tax_amount
    
    @staticmethod
    def submit_purchase_order(po_id, user):
        """Submit purchase order for processing"""
        try:
            po = PurchaseOrder.objects.get(id=po_id)
            
            if po.docstatus != 0:
                raise ValidationError("Can only submit draft purchase orders")
            
            if not po.items.exists():
                raise ValidationError("Cannot submit purchase order without items")
            
            po.docstatus = 1
            po.status = 'Submitted'
            po.updated_by = user.email
            po.save()
            
            return po
            
        except PurchaseOrder.DoesNotExist:
            raise ValidationError("Purchase order not found")
    
    @staticmethod
    def _generate_id(naming_series):
        """Generate unique purchase order ID"""
        import datetime
        current_year = datetime.datetime.now().year
        
        # Get next sequence number for this year
        last_po = PurchaseOrder.objects.filter(
            id__startswith=f'SPO-{current_year}-'
        ).order_by('-id').first()
        
        if last_po:
            last_num = int(last_po.id.split('-')[-1])
            next_num = last_num + 1
        else:
            next_num = 1
        
        return f'SPO-{current_year}-{next_num:04d}'
```

**Node.js Service Example:**
```javascript
const { PurchaseOrder, PurchaseOrderItem } = require('../models');
const { ValidationError } = require('../utils/errors');

class PurchaseOrderService {
  
  static async createPurchaseOrder(data, user) {
    const transaction = await sequelize.transaction();
    
    try {
      // Generate ID
      const poId = await this.generateId();
      
      // Create main record
      const po = await PurchaseOrder.create({
        id: poId,
        supplierId: data.supplier_id,
        companyId: data.company_id,
        transactionDate: data.transaction_date,
        requiredByDate: data.required_by_date,
        currency: data.currency,
        conversionRate: data.conversion_rate || 1.0,
        taxRate: data.tax_rate || 0,
        remarks: data.remarks || '',
        createdBy: user.email,
        updatedBy: user.email
      }, { transaction });
      
      // Add items
      const items = data.items.map((item, index) => ({
        id: `${poId}-${index + 1}`,
        parentId: poId,
        itemCode: item.item_code,
        qty: item.qty,
        rate: item.rate,
        warehouseId: item.warehouse_id,
        requiredByDate: item.required_by_date,
        lineSequence: index + 1
      }));
      
      await PurchaseOrderItem.bulkCreate(items, { transaction });
      
      // Calculate totals
      await this.calculateTotals(poId, transaction);
      
      await transaction.commit();
      
      // Return with items
      return await PurchaseOrder.findByPk(poId, {
        include: [{ model: PurchaseOrderItem, as: 'items' }]
      });
      
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }
  
  static async calculateTotals(poId, transaction) {
    const items = await PurchaseOrderItem.findAll({
      where: { parentId: poId },
      transaction
    });
    
    let netTotal = 0;
    
    for (const item of items) {
      const amount = parseFloat(item.qty) * parseFloat(item.rate);
      await item.update({ amount }, { transaction });
      netTotal += amount;
    }
    
    const po = await PurchaseOrder.findByPk(poId, { transaction });
    const taxAmount = netTotal * parseFloat(po.taxRate) / 100;
    const grandTotal = netTotal + taxAmount;
    
    await po.update({
      netTotal,
      taxAmount,
      grandTotal
    }, { transaction });
  }
}
```

### Step 2.2: Create Validation Logic

**Python Validators:**
```python
from django.core.exceptions import ValidationError
from datetime import date

class PurchaseOrderValidator:
    
    @staticmethod
    def validate_purchase_order(data):
        """Validate purchase order data"""
        errors = {}
        
        # Date validation
        if data.get('required_by_date'):
            if data['required_by_date'] < data['transaction_date']:
                errors['required_by_date'] = "Required by date cannot be before transaction date"
        
        # Items validation
        if not data.get('items'):
            errors['items'] = "At least one item is required"
        else:
            item_errors = PurchaseOrderValidator._validate_items(data['items'])
            if item_errors:
                errors['items'] = item_errors
        
        # Currency validation
        if data.get('conversion_rate', 0) <= 0:
            errors['conversion_rate'] = "Conversion rate must be greater than 0"
        
        if errors:
            raise ValidationError(errors)
    
    @staticmethod
    def _validate_items(items):
        """Validate purchase order items"""
        errors = []
        
        for idx, item in enumerate(items):
            item_errors = {}
            
            if not item.get('item_code'):
                item_errors['item_code'] = "Item code is required"
            
            if not item.get('qty') or float(item['qty']) <= 0:
                item_errors['qty'] = "Quantity must be greater than 0"
            
            if not item.get('rate') or float(item['rate']) <= 0:
                item_errors['rate'] = "Rate must be greater than 0"
            
            if not item.get('warehouse_id'):
                item_errors['warehouse_id'] = "Warehouse is required"
            
            if item_errors:
                errors.append({
                    'line': idx + 1,
                    'errors': item_errors
                })
        
        return errors if errors else None
```

## Phase 3: API Implementation

### Step 3.1: Create API Views

**Django REST Framework Views:**
```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from .models import PurchaseOrder
from .serializers import PurchaseOrderSerializer
from .services import PurchaseOrderService

class PurchaseOrderViewSet(viewsets.ModelViewSet):
    queryset = PurchaseOrder.objects.prefetch_related('items')
    serializer_class = PurchaseOrderSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['company_id', 'supplier_id', 'status']
    ordering = ['-transaction_date', '-id']
    
    def create(self, request):
        """Create new purchase order"""
        try:
            po = PurchaseOrderService.create_purchase_order(
                request.data, 
                request.user
            )
            serializer = self.get_serializer(po)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        except ValidationError as e:
            return Response(
                {'errors': e.message_dict}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    @action(detail=True, methods=['post'])
    def submit(self, request, pk=None):
        """Submit purchase order"""
        try:
            po = PurchaseOrderService.submit_purchase_order(pk, request.user)
            serializer = self.get_serializer(po)
            return Response(serializer.data)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    @action(detail=True, methods=['post'])
    def cancel(self, request, pk=None):
        """Cancel purchase order"""
        try:
            po = PurchaseOrderService.cancel_purchase_order(pk, request.user)
            serializer = self.get_serializer(po)
            return Response(serializer.data)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_400_BAD_REQUEST
            )
```

**Node.js/Express Routes:**
```javascript
const express = require('express');
const { PurchaseOrderService } = require('../services');
const { validatePurchaseOrder } = require('../validators');
const auth = require('../middleware/auth');

const router = express.Router();

// Create purchase order
router.post('/', auth, async (req, res) => {
  try {
    // Validate request
    const validation = validatePurchaseOrder(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        success: false,
        message: 'Validation failed',
        errors: validation.errors
      });
    }
    
    // Create purchase order
    const po = await PurchaseOrderService.createPurchaseOrder(
      req.body, 
      req.user
    );
    
    res.status(201).json({
      success: true,
      data: po,
      message: 'Purchase order created successfully'
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Internal server error',
      error: error.message
    });
  }
});

// Get purchase orders with filtering
router.get('/', auth, async (req, res) => {
  try {
    const {
      company_id,
      supplier_id,
      status,
      date_from,
      date_to,
      limit = 20,
      offset = 0
    } = req.query;
    
    const result = await PurchaseOrderService.getPurchaseOrders({
      company_id,
      supplier_id,
      status,
      date_from,
      date_to,
      limit: parseInt(limit),
      offset: parseInt(offset)
    });
    
    res.json({
      success: true,
      data: result,
      message: 'Purchase orders retrieved successfully'
    });
    
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Internal server error',
      error: error.message
    });
  }
});

// Submit purchase order
router.post('/:id/submit', auth, async (req, res) => {
  try {
    const po = await PurchaseOrderService.submitPurchaseOrder(
      req.params.id,
      req.user
    );
    
    res.json({
      success: true,
      data: po,
      message: 'Purchase order submitted successfully'
    });
    
  } catch (error) {
    if (error.name === 'ValidationError') {
      return res.status(400).json({
        success: false,
        message: error.message
      });
    }
    
    res.status(500).json({
      success: false,
      message: 'Internal server error',
      error: error.message
    });
  }
});

module.exports = router;
```

### Step 3.2: Create Serializers/DTOs

**Django Serializers:**
```python
from rest_framework import serializers
from .models import PurchaseOrder, PurchaseOrderItem

class PurchaseOrderItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = PurchaseOrderItem
        fields = [
            'id', 'item_code', 'item_name', 'description',
            'qty', 'uom', 'rate', 'amount', 'warehouse_id',
            'required_by_date', 'line_sequence'
        ]

class PurchaseOrderSerializer(serializers.ModelSerializer):
    items = PurchaseOrderItemSerializer(many=True, read_only=True)
    
    class Meta:
        model = PurchaseOrder
        fields = [
            'id', 'naming_series', 'supplier_id', 'supplier_name',
            'company_id', 'transaction_date', 'required_by_date',
            'currency', 'conversion_rate', 'status', 'docstatus',
            'net_total', 'tax_rate', 'tax_amount', 'grand_total',
            'remarks', 'created_at', 'updated_at', 'created_by',
            'updated_by', 'items'
        ]
        read_only_fields = [
            'id', 'supplier_name', 'status', 'docstatus',
            'net_total', 'tax_amount', 'grand_total',
            'created_at', 'updated_at', 'created_by', 'updated_by'
        ]
```

## Phase 4: Frontend Implementation

### Step 4.1: React Components Structure

**Component Hierarchy:**
```
src/
├── components/
│   ├── PurchaseOrder/
│   │   ├── PurchaseOrderList.jsx
│   │   ├── PurchaseOrderForm.jsx
│   │   ├── PurchaseOrderView.jsx
│   │   └── PurchaseOrderItem.jsx
│   ├── Common/
│   │   ├── DataTable.jsx
│   │   ├── FormField.jsx
│   │   └── Modal.jsx
├── services/
│   ├── purchaseOrderService.js
│   └── apiClient.js
├── hooks/
│   ├── usePurchaseOrders.js
│   └── useSuppliers.js
└── utils/
    ├── validation.js
    └── formatting.js
```

**Purchase Order Form Component:**
```jsx
import React, { useState, useEffect } from 'react';
import { usePurchaseOrders } from '../hooks/usePurchaseOrders';
import { useSuppliers } from '../hooks/useSuppliers';
import PurchaseOrderItem from './PurchaseOrderItem';

const PurchaseOrderForm = ({ orderId, onSave, onCancel }) => {
  const [formData, setFormData] = useState({
    supplier_id: '',
    company_id: '',
    transaction_date: new Date().toISOString().split('T')[0],
    required_by_date: '',
    currency: 'USD',
    tax_rate: 0,
    remarks: '',
    items: []
  });
  
  const [errors, setErrors] = useState({});
  const { createPurchaseOrder, updatePurchaseOrder } = usePurchaseOrders();
  const { suppliers } = useSuppliers();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const validationErrors = validateForm(formData);
      if (Object.keys(validationErrors).length > 0) {
        setErrors(validationErrors);
        return;
      }
      
      if (orderId) {
        await updatePurchaseOrder(orderId, formData);
      } else {
        await createPurchaseOrder(formData);
      }
      
      onSave();
    } catch (error) {
      setErrors(error.response?.data?.errors || {});
    }
  };
  
  const handleItemChange = (index, field, value) => {
    const newItems = [...formData.items];
    newItems[index] = { ...newItems[index], [field]: value };
    
    // Calculate amount
    if (field === 'qty' || field === 'rate') {
      newItems[index].amount = 
        (newItems[index].qty || 0) * (newItems[index].rate || 0);
    }
    
    setFormData({ ...formData, items: newItems });
  };
  
  const addItem = () => {
    setFormData({
      ...formData,
      items: [...formData.items, {
        item_code: '',
        qty: 0,
        rate: 0,
        amount: 0,
        warehouse_id: ''
      }]
    });
  };
  
  return (
    <form onSubmit={handleSubmit} className="purchase-order-form">
      <div className="form-header">
        <h2>{orderId ? 'Edit' : 'Create'} Purchase Order</h2>
      </div>
      
      <div className="form-section">
        <div className="form-row">
          <div className="form-field">
            <label>Supplier *</label>
            <select
              value={formData.supplier_id}
              onChange={(e) => setFormData({
                ...formData,
                supplier_id: e.target.value
              })}
              required
            >
              <option value="">Select Supplier</option>
              {suppliers.map(supplier => (
                <option key={supplier.id} value={supplier.id}>
                  {supplier.supplier_name}
                </option>
              ))}
            </select>
            {errors.supplier_id && (
              <span className="error">{errors.supplier_id}</span>
            )}
          </div>
          
          <div className="form-field">
            <label>Order Date *</label>
            <input
              type="date"
              value={formData.transaction_date}
              onChange={(e) => setFormData({
                ...formData,
                transaction_date: e.target.value
              })}
              required
            />
          </div>
        </div>
      </div>
      
      <div className="form-section">
        <h3>Items</h3>
        <div className="items-table">
          {formData.items.map((item, index) => (
            <PurchaseOrderItem
              key={index}
              item={item}
              index={index}
              onChange={handleItemChange}
              onRemove={() => {
                const newItems = formData.items.filter((_, i) => i !== index);
                setFormData({ ...formData, items: newItems });
              }}
            />
          ))}
        </div>
        <button type="button" onClick={addItem} className="btn-add-item">
          Add Item
        </button>
      </div>
      
      <div className="form-actions">
        <button type="button" onClick={onCancel} className="btn-cancel">
          Cancel
        </button>
        <button type="submit" className="btn-save">
          Save Purchase Order
        </button>
      </div>
    </form>
  );
};

export default PurchaseOrderForm;
```

### Step 4.2: State Management

**Custom Hooks for Data Management:**
```javascript
import { useState, useEffect } from 'react';
import { purchaseOrderService } from '../services/purchaseOrderService';

export const usePurchaseOrders = () => {
  const [orders, setOrders] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchOrders = async (filters = {}) => {
    setLoading(true);
    try {
      const response = await purchaseOrderService.getOrders(filters);
      setOrders(response.data.orders);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const createPurchaseOrder = async (data) => {
    const response = await purchaseOrderService.create(data);
    await fetchOrders(); // Refresh list
    return response.data;
  };
  
  const submitPurchaseOrder = async (id) => {
    const response = await purchaseOrderService.submit(id);
    await fetchOrders(); // Refresh list
    return response.data;
  };
  
  useEffect(() => {
    fetchOrders();
  }, []);
  
  return {
    orders,
    loading,
    error,
    fetchOrders,
    createPurchaseOrder,
    submitPurchaseOrder
  };
};
```

## Phase 5: Testing Implementation

### Step 5.1: Unit Tests

**Python/Django Tests:**
```python
from django.test import TestCase
from django.contrib.auth.models import User
from decimal import Decimal
from .models import PurchaseOrder, PurchaseOrderItem
from .services import PurchaseOrderService

class PurchaseOrderServiceTest(TestCase):
    
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com'
        )
        
        self.po_data = {
            'supplier_id': 'SUPP-001',
            'company_id': 'COMP-001',
            'transaction_date': '2025-06-18',
            'currency': 'USD',
            'tax_rate': 10,
            'items': [
                {
                    'item_code': 'ITEM-001',
                    'qty': 10,
                    'rate': 50.0,
                    'warehouse_id': 'WH-001'
                }
            ]
        }
    
    def test_create_purchase_order_success(self):
        """Test successful purchase order creation"""
        po = PurchaseOrderService.create_purchase_order(
            self.po_data, 
            self.user
        )
        
        self.assertIsNotNone(po.id)
        self.assertEqual(po.supplier_id, 'SUPP-001')
        self.assertEqual(po.net_total, Decimal('500.00'))
        self.assertEqual(po.tax_amount, Decimal('50.00'))
        self.assertEqual(po.grand_total, Decimal('550.00'))
        self.assertEqual(po.items.count(), 1)
    
    def test_create_purchase_order_invalid_data(self):
        """Test purchase order creation with invalid data"""
        invalid_data = self.po_data.copy()
        invalid_data['items'][0]['qty'] = 0  # Invalid quantity
        
        with self.assertRaises(ValidationError):
            PurchaseOrderService.create_purchase_order(
                invalid_data, 
                self.user
            )
    
    def test_submit_purchase_order(self):
        """Test purchase order submission"""
        po = PurchaseOrderService.create_purchase_order(
            self.po_data, 
            self.user
        )
        
        submitted_po = PurchaseOrderService.submit_purchase_order(
            po.id, 
            self.user
        )
        
        self.assertEqual(submitted_po.status, 'Submitted')
        self.assertEqual(submitted_po.docstatus, 1)
```

**JavaScript/Jest Tests:**
```javascript
const { PurchaseOrderService } = require('../services/PurchaseOrderService');
const { PurchaseOrder, PurchaseOrderItem } = require('../models');

describe('PurchaseOrderService', () => {
  let testUser;
  let testData;
  
  beforeEach(() => {
    testUser = { email: 'test@example.com' };
    testData = {
      supplier_id: 'SUPP-001',
      company_id: 'COMP-001',
      transaction_date: '2025-06-18',
      currency: 'USD',
      tax_rate: 10,
      items: [
        {
          item_code: 'ITEM-001',
          qty: 10,
          rate: 50.0,
          warehouse_id: 'WH-001'
        }
      ]
    };
  });
  
  test('should create purchase order successfully', async () => {
    const po = await PurchaseOrderService.createPurchaseOrder(
      testData,
      testUser
    );
    
    expect(po.id).toBeDefined();
    expect(po.supplierId).toBe('SUPP-001');
    expect(parseFloat(po.netTotal)).toBe(500.00);
    expect(parseFloat(po.taxAmount)).toBe(50.00);
    expect(parseFloat(po.grandTotal)).toBe(550.00);
    expect(po.items).toHaveLength(1);
  });
  
  test('should validate required fields', async () => {
    const invalidData = { ...testData };
    delete invalidData.supplier_id;
    
    await expect(
      PurchaseOrderService.createPurchaseOrder(invalidData, testUser)
    ).rejects.toThrow('Supplier is required');
  });
});
```

### Step 5.2: Integration Tests

**API Integration Tests:**
```python
from django.test import TestCase
from django.urls import reverse
from rest_framework.test import APIClient
from rest_framework import status
from django.contrib.auth.models import User

class PurchaseOrderAPITest(TestCase):
    
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass'
        )
        self.client.force_authenticate(user=self.user)
    
    def test_create_purchase_order_api(self):
        """Test purchase order creation via API"""
        data = {
            'supplier_id': 'SUPP-001',
            'company_id': 'COMP-001',
            'transaction_date': '2025-06-18',
            'currency': 'USD',
            'tax_rate': 10,
            'items': [
                {
                    'item_code': 'ITEM-001',
                    'qty': 10,
                    'rate': 50.0,
                    'warehouse_id': 'WH-001'
                }
            ]
        }
        
        url = reverse('purchaseorder-list')
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['supplier_id'], 'SUPP-001')
        self.assertEqual(float(response.data['grand_total']), 550.00)
```

## Phase 6: Deployment

### Step 6.1: Environment Configuration

**Docker Configuration:**
```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "simple_po.wsgi:application", "--bind", "0.0.0.0:8000"]
```

**Docker Compose:**
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: simple_purchase_order
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: secure_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    
  web:
    build: .
    command: gunicorn simple_po.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DEBUG=False
      - DATABASE_URL=postgresql://app_user:secure_password@db:5432/simple_purchase_order

volumes:
  postgres_data:
```

### Step 6.2: Production Checklist

1. **Security**
   - [ ] Enable HTTPS/SSL
   - [ ] Configure secure headers
   - [ ] Set up rate limiting
   - [ ] Validate all inputs
   - [ ] Use environment variables for secrets

2. **Performance**
   - [ ] Enable database query optimization
   - [ ] Configure caching (Redis/Memcached)
   - [ ] Set up CDN for static files
   - [ ] Enable gzip compression

3. **Monitoring**
   - [ ] Set up application logging
   - [ ] Configure error tracking (Sentry)
   - [ ] Monitor database performance
   - [ ] Set up health checks

4. **Backup**
   - [ ] Configure automated database backups
   - [ ] Test backup restoration
   - [ ] Document recovery procedures

This implementation guide provides a comprehensive roadmap for building the Simple Purchase Order module while maintaining flexibility for different technology stacks and business requirements.