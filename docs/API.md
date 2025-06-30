# API Documentation

## Overview
The Bakery Website API is a RESTful API built with Express.js that provides endpoints for product management, shopping cart operations, order processing, and user management. All endpoints return JSON responses and require appropriate authentication.

## Base URL
```
Development: http://localhost:5000/api
Production: https://api.yourdomain.com/api
```

## Authentication
The API uses JWT token verification for authentication. The frontend uses Clerk to authenticate users and obtain JWT tokens. All protected endpoints require a valid Bearer token in the Authorization header.

```http
Authorization: Bearer <jwt_token>
```

### Frontend - Getting the Token
In the React app, use Clerk's `useAuth` hook:
```typescript
const { getToken } = useAuth();
const token = await getToken();

// Include in API requests
const response = await axios.get('/api/protected', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

### Backend - Token Verification
The FastAPI backend verifies JWT tokens using the JWKS endpoint:
```python
from jose import jwt, JWTError
from fastapi import HTTPException, Depends
from fastapi.security import HTTPBearer

security = HTTPBearer()

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        payload = jwt.decode(
            credentials.credentials,
            key=JWKS_CLIENT,  # Fetched from Clerk's JWKS endpoint
            algorithms=["RS256"]
        )
        return payload
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid authentication token")
```

## Common Response Formats

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": { ... }
  }
}
```

### Pagination Response
```json
{
  "success": true,
  "data": {
    "items": [ ... ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 100,
      "pages": 10
    }
  }
}
```

## API Endpoints

### Authentication Endpoints

Since authentication is handled entirely by Clerk on the frontend, the backend only needs to verify tokens and sync user data.

---

### User Endpoints

#### Get Current User
```http
GET /api/users/me
```

Get the current authenticated user's profile.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "clerkId": "user_xxx",
    "email": "user@example.com",
    "name": "John Doe",
    "role": "CUSTOMER",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Update User Profile
```http
PUT /api/users/me
```

Update the current user's profile information.

**Body:**
```json
{
  "name": "Jane Doe"
}
```

---

### Category Endpoints

#### List All Categories
```http
GET /api/categories
```

Get all product categories.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "cuid",
      "name": "Cake",
      "slug": "cake",
      "description": "Delicious cakes for all occasions",
      "displayOrder": 1,
      "_count": {
        "products": 15
      }
    }
  ]
}
```

#### Get Category Products
```http
GET /api/categories/:slug/products
```

Get all products in a specific category.

**Parameters:**
- `slug` (path): Category slug

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 12)
- `sort` (optional): Sort by price_asc, price_desc, name_asc, name_desc (default: name_asc)

---

### Product Endpoints

#### List Products
```http
GET /api/products
```

Get all products with optional filtering.

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 12)
- `category` (optional): Filter by category slug
- `featured` (optional): Filter featured products (true/false)
- `search` (optional): Search by name or description
- `sort` (optional): Sort by price_asc, price_desc, name_asc, name_desc, created_desc

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "cuid",
        "name": "Chocolate Cake",
        "description": "Rich chocolate cake with ganache",
        "price": 35.00,
        "image": "https://example.com/chocolate-cake.jpg",
        "category": {
          "id": "cuid",
          "name": "Cake",
          "slug": "cake"
        },
        "featured": true,
        "inStock": true,
        "stockQuantity": 20
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 12,
      "total": 48,
      "pages": 4
    }
  }
}
```

#### Get Product Details
```http
GET /api/products/:id
```

Get detailed information about a specific product.

**Parameters:**
- `id` (path): Product ID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "name": "Chocolate Cake",
    "description": "Rich chocolate cake with ganache",
    "price": 35.00,
    "image": "https://example.com/chocolate-cake.jpg",
    "category": {
      "id": "cuid",
      "name": "Cake",
      "slug": "cake"
    },
    "featured": true,
    "inStock": true,
    "stockQuantity": 20,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Create Product (Admin)
```http
POST /api/products
```

Create a new product. Requires admin role.

**Body:**
```json
{
  "name": "Red Velvet Cake",
  "description": "Classic red velvet with cream cheese frosting",
  "price": 40.00,
  "categoryId": "cuid",
  "featured": false,
  "stockQuantity": 15
}
```

**Note:** Image upload is handled separately via `/api/upload` endpoint.

#### Update Product (Admin)
```http
PUT /api/products/:id
```

Update an existing product. Requires admin role.

**Parameters:**
- `id` (path): Product ID

**Body:** Same as create, all fields optional

#### Delete Product (Admin)
```http
DELETE /api/products/:id
```

Delete a product. Requires admin role.

**Parameters:**
- `id` (path): Product ID

---

### Cart Endpoints

#### Get Cart
```http
GET /api/cart
```

Get the current user's shopping cart.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "items": [
      {
        "id": "cuid",
        "quantity": 2,
        "product": {
          "id": "cuid",
          "name": "Chocolate Cake",
          "price": 35.00,
          "image": "https://example.com/chocolate-cake.jpg",
          "inStock": true
        }
      }
    ],
    "subtotal": 70.00,
    "itemCount": 2
  }
}
```

#### Add to Cart
```http
POST /api/cart/items
```

Add a product to the cart.

**Body:**
```json
{
  "productId": "cuid",
  "quantity": 1
}
```

#### Update Cart Item
```http
PUT /api/cart/items/:id
```

Update quantity of a cart item.

**Parameters:**
- `id` (path): Cart item ID

**Body:**
```json
{
  "quantity": 3
}
```

#### Remove from Cart
```http
DELETE /api/cart/items/:id
```

Remove an item from the cart.

**Parameters:**
- `id` (path): Cart item ID

#### Clear Cart
```http
DELETE /api/cart
```

Remove all items from the cart.

---

### Order Endpoints

#### Create Order
```http
POST /api/orders
```

Create a new order from the current cart.

**Body:**
```json
{
  "customerName": "John Doe",
  "customerEmail": "john@example.com",
  "customerPhone": "+1234567890",
  "deliveryMethod": "PICKUP",
  "deliveryAddress": "123 Main St, City, State 12345",
  "deliveryDate": "2024-01-15T10:00:00Z",
  "notes": "Please call when ready"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "orderNumber": "ORD-20240101-001",
    "status": "PENDING",
    "total": 75.00,
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### List Orders
```http
GET /api/orders
```

Get all orders for the current user (or all orders for admin).

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `status` (optional): Filter by status

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "cuid",
        "orderNumber": "ORD-20240101-001",
        "status": "COMPLETED",
        "total": 75.00,
        "itemCount": 2,
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 25,
      "pages": 3
    }
  }
}
```

#### Get Order Details
```http
GET /api/orders/:id
```

Get detailed information about a specific order.

**Parameters:**
- `id` (path): Order ID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "orderNumber": "ORD-20240101-001",
    "status": "COMPLETED",
    "items": [
      {
        "id": "cuid",
        "quantity": 2,
        "unitPrice": 35.00,
        "totalPrice": 70.00,
        "product": {
          "id": "cuid",
          "name": "Chocolate Cake",
          "image": "https://example.com/chocolate-cake.jpg"
        }
      }
    ],
    "subtotal": 70.00,
    "tax": 5.00,
    "total": 75.00,
    "customerName": "John Doe",
    "customerEmail": "john@example.com",
    "customerPhone": "+1234567890",
    "deliveryMethod": "PICKUP",
    "deliveryDate": "2024-01-15T10:00:00Z",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Update Order Status (Admin)
```http
PUT /api/orders/:id/status
```

Update the status of an order. Requires admin role.

**Parameters:**
- `id` (path): Order ID

**Body:**
```json
{
  "status": "PROCESSING"
}
```

**Valid Status Values:**
- `PENDING`
- `PROCESSING`
- `READY`
- `COMPLETED`
- `CANCELLED`

---

### File Upload Endpoints (Multer)

#### Upload Product Image
```http
POST /api/upload/product
```

Upload a product image using Multer.

**Headers:**
```
Content-Type: multipart/form-data
Authorization: Bearer <token>
```

**Body:**
- `image`: Image file (JPEG, PNG, WebP, max 5MB)

**Response:**
```json
{
  "success": true,
  "data": {
    "url": "/uploads/products/1704067200000-chocolate-cake.jpg",
    "filename": "1704067200000-chocolate-cake.jpg",
    "originalName": "chocolate-cake.jpg",
    "mimetype": "image/jpeg",
    "size": 102400
  }
}
```

#### Upload Multiple Product Images
```http
POST /api/upload/products
```

Upload multiple product images (max 5 files).

**Headers:**
```
Content-Type: multipart/form-data
Authorization: Bearer <token>
```

**Body:**
- `images`: Array of image files

**Response:**
```json
{
  "success": true,
  "data": {
    "files": [
      {
        "url": "/uploads/products/1704067200000-image1.jpg",
        "filename": "1704067200000-image1.jpg",
        "size": 102400
      },
      {
        "url": "/uploads/products/1704067200001-image2.jpg",
        "filename": "1704067200001-image2.jpg",
        "size": 98304
      }
    ]
  }
}
```

**File Upload Configuration:**
- Maximum file size: 5MB per file
- Accepted formats: JPEG, PNG, WebP
- Storage location: `./uploads/products` (development)
- Files served from: `http://localhost:5000/uploads/products/`

---

### Admin Dashboard Endpoints

#### Get Dashboard Stats (Admin)
```http
GET /api/admin/dashboard
```

Get dashboard statistics. Requires admin role.

**Response:**
```json
{
  "success": true,
  "data": {
    "totalOrders": 150,
    "pendingOrders": 5,
    "totalRevenue": 12500.00,
    "totalProducts": 48,
    "totalUsers": 230,
    "recentOrders": [ ... ],
    "popularProducts": [ ... ]
  }
}
```

## Error Codes

| Code | Description |
|------|-------------|
| `UNAUTHORIZED` | Missing or invalid authentication token |
| `FORBIDDEN` | Insufficient permissions |
| `NOT_FOUND` | Resource not found |
| `VALIDATION_ERROR` | Invalid input data |
| `DUPLICATE_ENTRY` | Resource already exists |
| `OUT_OF_STOCK` | Product is out of stock |
| `CART_EMPTY` | Cart is empty |
| `INTERNAL_ERROR` | Server error |

## Rate Limiting

API endpoints are rate-limited to prevent abuse:
- Public endpoints: 100 requests per minute
- Authenticated endpoints: 300 requests per minute
- Admin endpoints: 500 requests per minute

Rate limit headers:
```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 299
X-RateLimit-Reset: 1609459200
```

## Webhooks

### Order Status Update
```http
POST https://your-webhook-url.com/order-status
```

Triggered when an order status changes.

**Payload:**
```json
{
  "event": "order.status.updated",
  "timestamp": "2024-01-01T00:00:00Z",
  "data": {
    "orderId": "cuid",
    "orderNumber": "ORD-20240101-001",
    "oldStatus": "PENDING",
    "newStatus": "PROCESSING",
    "userId": "cuid"
  }
}
```

## Testing

### Using cURL

Get products:
```bash
curl -X GET http://localhost:5000/api/products
```

Add to cart (with auth):
```bash
curl -X POST http://localhost:5000/api/cart/items \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"productId":"cuid","quantity":1}'
```

### Using Postman

Import the Postman collection from `/docs/postman-collection.json` for a complete set of API requests with examples.

## SDK Usage

### JavaScript/TypeScript
```typescript
import { BakeryAPI } from '@bakery/sdk';

const api = new BakeryAPI({
  baseURL: 'http://localhost:5000/api',
  getToken: async () => await clerk.session?.getToken()
});

// Get products
const products = await api.products.list({ featured: true });

// Add to cart
await api.cart.addItem({ productId: 'cuid', quantity: 2 });
```