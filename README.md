# MuleSoft Purchase Order Integration APIs

This project implements a complete MuleSoft purchase order integration solution following strict layered architecture patterns with reusable components.

## Project Structure

```
Mule-Dev-1/
├── Flipkart-purchaseorder-sysapi/           # Flipkart System API (Port 8081)
├── Amazon-purchaseorder-sysapi/             # Amazon System API (Port 8082)  
├── purchaseorder-procapi/                   # Process API - Reusable (Port 8083)
├── purchaseorder-expapi/                    # Experience API (Port 8084)
├── postman-collections/                     # Postman test collections
└── README.md                                # This file
```

## Architecture Overview

### Use Case 1: Flipkart → IBM (Three-Layer Architecture)
```
External Consumer → Experience API → Process API → System API → Flipkart
                       ↓
                   IBM System
```

### Use Case 2: Amazon → HP (Two-Layer Architecture) 
```
External Consumer → Process API → System API → Amazon
                       ↓
                   HP System
```

## API Details

### 1. System APIs

#### Flipkart System API (`Flipkart-purchaseorder-sysapi`)
- **Port**: 8081
- **Purpose**: Connects to Flipkart's order service and exposes raw order details
- **Endpoints**:
  - `GET /api/v1/orders` - Get all orders with pagination
  - `GET /api/v1/orders/{orderId}` - Get specific order by ID
  - `GET /api/v1/health` - Health check

#### Amazon System API (`Amazon-purchaseorder-sysapi`)
- **Port**: 8082
- **Purpose**: Connects to Amazon's order service and exposes raw order details
- **Endpoints**:
  - `GET /api/v1/orders` - Get all orders with pagination
  - `GET /api/v1/orders/{orderNumber}` - Get specific order by number
  - `GET /api/v1/health` - Health check

### 2. Process API (`purchaseorder-procapi`)

- **Port**: 8083
- **Purpose**: Reusable orchestration layer for transformations
- **Supports**:
  - Flipkart → IBM format transformation
  - Amazon → HP format transformation
- **Endpoints**:
  - `POST /api/v1/process` - Process orders with transformation
  - `GET /api/v1/process/{transactionId}` - Get processing status
  - `GET /api/v1/health` - Health check

### 3. Experience API (`purchaseorder-expapi`)

- **Port**: 8084
- **Purpose**: Simplified interface for Flipkart to IBM integration
- **Endpoints**:
  - `POST /api/v1/orders` - Submit single order
  - `POST /api/v1/orders/bulk` - Submit bulk orders
  - `GET /api/v1/status/{requestId}` - Get order status
  - `GET /api/v1/status/batch/{batchId}` - Get batch status
  - `GET /api/v1/health` - Health check

## DataWeave Transformations

### Flipkart to IBM Transformation
- Maps Flipkart order schema to IBM's required format
- Handles field mapping: `orderId` → `ibm_order_id`
- Status conversion: `CONFIRMED` → `NEW`, `SHIPPED` → `IN_PROGRESS`, etc.

### Amazon to HP Transformation  
- Maps Amazon order schema to HP's required format
- Handles field mapping: `orderNumber` → `hp_reference_number`
- Status conversion: `PLACED` → `RECEIVED`, `PROCESSING` → `PROCESSING`, etc.

## Error Handling Strategy

All APIs implement comprehensive error handling:

1. **Input Validation**: Validates required fields and data formats
2. **Business Logic Errors**: Handles transformation and processing errors
3. **System Errors**: Manages connectivity and runtime issues
4. **Consistent Error Response Format**:
   ```json
   {
     "errorCode": "SYS_001",
     "errorMessage": "Descriptive error message",
     "timestamp": "2024-01-15T10:30:00.000Z"
   }
   ```

## Sample Payloads

### Flipkart Order Format
```json
{
  "orderId": "FK123456789",
  "customerInfo": {
    "customerId": "CUST001",
    "customerName": "John Doe",
    "email": "john.doe@email.com",
    "phone": "+91-9876543210"
  },
  "orderDetails": {
    "orderDate": "2024-01-15T10:30:00Z",
    "totalAmount": 45000.00,
    "currency": "INR",
    "status": "CONFIRMED"
  },
  "items": [
    {
      "itemId": "ITEM001", 
      "productName": "Laptop Dell Inspiron",
      "quantity": 1,
      "unitPrice": 45000.00,
      "category": "Electronics"
    }
  ],
  "shippingAddress": {
    "addressLine1": "123 Main Street",
    "city": "Bangalore",
    "state": "Karnataka",
    "postalCode": "560001",
    "country": "India"
  }
}
```

### Amazon Order Format
```json
{
  "orderNumber": "AMZ987654321",
  "buyerInfo": {
    "buyerId": "BUYER001",
    "fullName": "Jane Smith",
    "emailAddress": "jane.smith@email.com",
    "phoneNumber": "+1-555-123-4567"
  },
  "purchaseInfo": {
    "purchaseDate": "2024-01-15T14:45:00Z",
    "grandTotal": 899.98,
    "currencyCode": "USD",
    "orderStatus": "PROCESSING"
  },
  "productList": [
    {
      "productId": "PROD001",
      "title": "HP Laptop ProBook 450",
      "qty": 2,
      "price": 449.99,
      "productCategory": "Computers"
    }
  ],
  "deliveryAddress": {
    "street": "456 Oak Avenue",
    "cityName": "Seattle",
    "stateCode": "WA",
    "zipCode": "98101",
    "countryCode": "US"
  }
}
```

### IBM Order Format (Target)
```json
{
  "ibm_order_id": "IBM_FK123456789",
  "client_details": {
    "client_id": "CUST001",
    "client_name": "John Doe",
    "contact_email": "john.doe@email.com",
    "contact_phone": "+91-9876543210"
  },
  "order_summary": {
    "created_date": "2024-01-15T10:30:00.000Z",
    "total_value": 45000.00,
    "currency_type": "INR",
    "processing_status": "NEW"
  },
  "line_items": [
    {
      "line_id": "LINE_001",
      "product_code": "ITEM001",
      "product_description": "Laptop Dell Inspiron",
      "order_quantity": 1,
      "unit_cost": 45000.00,
      "item_category": "Electronics"
    }
  ],
  "delivery_info": {
    "address_line_1": "123 Main Street",
    "city_name": "Bangalore",
    "state_province": "Karnataka",
    "postal_code": "560001",
    "country_name": "India"
  }
}
```

### HP Order Format (Target)
```json
{
  "hp_reference_number": "HP_AMZ987654321",
  "customer_info": {
    "customer_code": "BUYER001",
    "customer_full_name": "Jane Smith",
    "email_id": "jane.smith@email.com",
    "phone_number": "+1-555-123-4567"
  },
  "transaction_details": {
    "transaction_date": "2024-01-15T14:45:00.000Z",
    "gross_amount": 899.98,
    "transaction_currency": "USD",
    "order_state": "PROCESSING"
  },
  "product_lines": [
    {
      "line_number": "001",
      "sku_code": "PROD001",
      "product_name": "HP Laptop ProBook 450",
      "ordered_qty": 2,
      "list_price": 449.99,
      "product_type": "Computers"
    }
  ],
  "shipping_details": {
    "ship_to_address1": "456 Oak Avenue",
    "ship_to_address2": "Suite 200",
    "ship_to_city": "Seattle",
    "ship_to_state": "WA",
    "ship_to_zip": "98101",
    "ship_to_country": "US"
  }
}
```

## Getting Started

### Prerequisites
- MuleSoft Anypoint Studio or VS Code with MuleSoft extension
- Maven 3.6+
- Java 17+
- Postman (for API testing)

### Running the APIs

1. **Start each API individually**:
   ```bash
   # Flipkart System API
   cd Flipkart-purchaseorder-sysapi/flipkart-purchaseorder-sysapi
   mvn mule:run
   
   # Amazon System API  
   cd Amazon-purchaseorder-sysapi/amazon-purchaseorder-sysapi
   mvn mule:run
   
   # Process API
   cd purchaseorder-procapi
   mvn mule:run
   
   # Experience API
   cd purchaseorder-expapi  
   mvn mule:run
   ```

2. **Import Postman Collection**:
   - Open Postman
   - Import `postman-collections/Purchase-Order-Integration-APIs.postman_collection.json`

### Testing Integration Flows

#### Test Sequence 1: Flipkart → IBM (3-Layer)
1. Test Flipkart System API endpoints
2. Submit order via Experience API
3. Verify transformation through Process API
4. Check status endpoints

#### Test Sequence 2: Amazon → HP (2-Layer)  
1. Test Amazon System API endpoints
2. Submit order directly to Process API
3. Verify HP format transformation
4. Check transaction status

## Key Features

✅ **Strict Layered Architecture**: Proper separation of System, Process, and Experience layers  
✅ **Reusable Components**: Process API handles multiple source-target combinations  
✅ **Independent Deployments**: Each API in separate folders for independent lifecycle  
✅ **Comprehensive Error Handling**: Consistent error responses across all APIs  
✅ **DataWeave Transformations**: Robust schema mapping between different systems  
✅ **Health Monitoring**: Health check endpoints on all APIs  
✅ **Complete Testing Suite**: Postman collection with end-to-end test scenarios  

## Configuration

Each API has its own `config.properties` file for environment-specific configuration:

- **Flipkart System API**: Port 8081
- **Amazon System API**: Port 8082  
- **Process API**: Port 8083
- **Experience API**: Port 8084

## Next Steps

1. **Import Postman Collection** and start testing
2. **Deploy to CloudHub 2.0** using Anypoint Studio
3. **Connect to Real Systems** by updating connector configurations
4. **Add Security Policies** via API Manager
5. **Enable Monitoring** through Anypoint Monitoring

## Support

For questions or issues with this integration solution, refer to:
- MuleSoft Documentation
- Anypoint Studio Help
- DataWeave Reference Guide
