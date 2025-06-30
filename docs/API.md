# API Documentation

## Overview
The unified API is a RESTful API built with FastAPI that serves both the landing site and web application. It provides endpoints for content management, collection operations, activity processing, and user management. All endpoints return JSON responses, with public endpoints available for the landing site and protected endpoints requiring authentication for the web application.

## Base URL
```
Development: http://localhost:5000/api
Production: https://app.yourdomain.com/api
```

## API Architecture
The API serves two distinct applications:
- **Landing Site** (`yourdomain.com`): Uses public endpoints for content display
- **Web Application** (`app.yourdomain.com`): Uses authenticated endpoints for full functionality

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

### Public Endpoints (Landing & Web)
These endpoints are accessible without authentication and are used by both the landing site and web application.

#### Get Featured Items
```http
GET /api/public/items/featured
```

Get featured items for homepage display.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "cuid",
      "name": "[Item Name]",
      "description": "[Description]",
      "metadata": "[Domain-specific data]",
      "image": "https://example.com/image.jpg",
      "category": {
        "name": "[Category]",
        "slug": "category-slug"
      }
    }
  ]
}
```

#### Get Public Categories
```http
GET /api/public/categories
```

Get all categories for navigation.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "cuid",
      "name": "[Category Name]",
      "slug": "category-slug",
      "displayOrder": 1,
      "itemCount": 15
    }
  ]
}
```

### Authentication Endpoints

Authentication is handled entirely by Clerk on the frontend. The backend only verifies tokens and syncs user data.

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
    "role": "USER",
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

Get all content categories.

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "cuid",
      "name": "[Type A]",
      "slug": "type-a",
      "description": "[Category description]",
      "displayOrder": 1,
      "_count": {
        "items": 15
      }
    }
  ]
}
```

#### Get Category Items
```http
GET /api/categories/:slug/items
```

Get all items in a specific category.

**Parameters:**
- `slug` (path): Category slug

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 12)
- `sort` (optional): Sort by price_asc, price_desc, name_asc, name_desc (default: name_asc)

---

### Content Endpoints

#### List Items
```http
GET /api/items
```

Get all items with optional filtering.

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 12)
- `category` (optional): Filter by category slug
- `featured` (optional): Filter featured items (true/false)
- `search` (optional): Search by name or description
- `sort` (optional): Sort by metadata_asc, metadata_desc, name_asc, name_desc, created_desc

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

#### Get Item Details
```http
GET /api/items/:id
```

Get detailed information about a specific item.

**Parameters:**
- `id` (path): Item ID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "name": "[Item Name]",
    "description": "[Item description]",
    "metadata": "[domain-specific data]",
    "image": "https://example.com/item-image.jpg",
    "category": {
      "id": "cuid",
      "name": "[Type A]",
      "slug": "type-a"
    },
    "featured": true,
    "available": true,
    "quantity": 20,
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Create Item (Admin)
```http
POST /api/items
```

Create a new item. Requires admin role.

**Body:**
```json
{
  "name": "[Item Name]",
  "description": "[Item description]",
  "metadata": "[domain-specific data]",
  "categoryId": "cuid",
  "featured": false,
  "quantity": 15
}
```

**Note:** Image upload is handled separately via `/api/upload` endpoint.

#### Update Item (Admin)
```http
PUT /api/items/:id
```

Update an existing item. Requires admin role.

**Parameters:**
- `id` (path): Item ID

**Body:** Same as create, all fields optional

#### Delete Item (Admin)
```http
DELETE /api/items/:id
```

Delete an item. Requires admin role.

**Parameters:**
- `id` (path): Item ID

---

### Collection Endpoints

#### Get Collection
```http
GET /api/collections
```

Get the current user's collection.

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
        "item": {
          "id": "cuid",
          "name": "[Item Name]",
          "metadata": "[domain-specific data]",
          "image": "https://example.com/item-image.jpg",
          "available": true
        }
      }
    ],
    "totalData": "[calculated data]",
    "itemCount": 2
  }
}
```

#### Add to Collection
```http
POST /api/collections/items
```

Add an item to the collection.

**Body:**
```json
{
  "itemId": "cuid",
  "quantity": 1
}
```

#### Update Collection Item
```http
PUT /api/collections/items/:id
```

Update quantity of a collection item.

**Parameters:**
- `id` (path): Collection item ID

**Body:**
```json
{
  "quantity": 3
}
```

#### Remove from Collection
```http
DELETE /api/collections/items/:id
```

Remove an item from the collection.

**Parameters:**
- `id` (path): Collection item ID

#### Clear Collection
```http
DELETE /api/collections
```

Remove all items from the collection.

---

### Activity Endpoints

#### Create Activity
```http
POST /api/activities
```

Create a new activity from the current collection.

**Body:**
```json
{
  "userInfo": "John Doe",
  "contactInfo": "john@example.com",
  "method": "ONLINE",
  "location": "123 Main St, City, State 12345",
  "scheduledDate": "2024-01-15T10:00:00Z",
  "notes": "[Additional notes]"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "activityNumber": "ACT-20240101-001",
    "status": "PENDING",
    "data": "[calculated data]",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### List Activities
```http
GET /api/activities
```

Get all activities for the current user (or all activities for admin).

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
        "activityNumber": "ACT-20240101-001",
        "status": "COMPLETED",
        "data": "[calculated data]",
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

#### Get Activity Details
```http
GET /api/activities/:id
```

Get detailed information about a specific activity.

**Parameters:**
- `id` (path): Activity ID

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "cuid",
    "activityNumber": "ACT-20240101-001",
    "status": "COMPLETED",
    "items": [
      {
        "id": "cuid",
        "quantity": 2,
        "itemData": "[item-specific data]",
        "totalData": "[calculated total]",
        "item": {
          "id": "cuid",
          "name": "[Item Name]",
          "image": "https://example.com/item-image.jpg"
        }
      }
    ],
    "data": "[total calculated data]",
    "userInfo": "John Doe",
    "contactInfo": "john@example.com",
    "method": "ONLINE",
    "location": "123 Main St, City, State 12345",
    "scheduledDate": "2024-01-15T10:00:00Z",
    "notes": "[Additional notes]",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

#### Update Activity Status (Admin)
```http
PUT /api/activities/:id/status
```

Update the status of an activity. Requires admin role.

**Parameters:**
- `id` (path): Activity ID

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

### File Upload Endpoints

#### Upload Item Image
```http
POST /api/upload/item
```

Upload an item image using multipart form data.

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
    "url": "/uploads/items/1704067200000-item-image.jpg",
    "filename": "1704067200000-item-image.jpg",
    "originalName": "item-image.jpg",
    "mimetype": "image/jpeg",
    "size": 102400
  }
}
```

#### Upload Multiple Item Images
```http
POST /api/upload/items
```

Upload multiple item images (max 5 files).

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
        "url": "/uploads/items/1704067200000-image1.jpg",
        "filename": "1704067200000-image1.jpg",
        "size": 102400
      },
      {
        "url": "/uploads/items/1704067200001-image2.jpg",
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
- Storage location: `./uploads/items` (development)
- Files served from: `http://localhost:5000/uploads/items/`

---

### Landing Site Specific Endpoints

#### Submit Contact Form
```http
POST /api/public/contact
```

Submit contact form from landing page.

**Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "message": "Inquiry message",
  "subject": "General Inquiry"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Contact form submitted successfully"
}
```

#### Subscribe to Newsletter
```http
POST /api/public/newsletter
```

Subscribe email to newsletter.

**Body:**
```json
{
  "email": "user@example.com"
}
```

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
    "totalActivities": 150,
    "pendingActivities": 5,
    "totalData": "[aggregated metrics]",
    "totalItems": 48,
    "totalUsers": 230,
    "recentActivities": [ ... ],
    "popularItems": [ ... ]
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
| `UNAVAILABLE` | Item is unavailable |
| `COLLECTION_EMPTY` | Collection is empty |
| `INTERNAL_ERROR` | Server error |

## Rate Limiting

API endpoints are rate-limited to prevent abuse:
- Public endpoints: 100 requests per minute per IP
- Landing site endpoints: 200 requests per minute per IP
- Authenticated endpoints: 300 requests per minute per user
- Admin endpoints: 500 requests per minute per user

Rate limit headers:
```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 299
X-RateLimit-Reset: 1609459200
```

## Webhooks

### Activity Status Update
```http
POST https://your-webhook-url.com/activity-status
```

Triggered when an activity status changes.

**Payload:**
```json
{
  "event": "activity.status.updated",
  "timestamp": "2024-01-01T00:00:00Z",
  "data": {
    "activityId": "cuid",
    "activityNumber": "ACT-20240101-001",
    "oldStatus": "PENDING",
    "newStatus": "PROCESSING",
    "userId": "cuid"
  }
}
```

## Testing

### Using cURL

Get items:
```bash
curl -X GET http://localhost:5000/api/items
```

Add to collection (with auth):
```bash
curl -X POST http://localhost:5000/api/collections/items \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"itemId":"cuid","quantity":1}'
```

### Using Postman

Import the Postman collection from `/docs/postman-collection.json` for a complete set of API requests with examples.

## SDK Usage

### JavaScript/TypeScript

#### Landing Site SDK
```typescript
import { LandingAPI } from '@[project]/sdk';

const api = new LandingAPI({
  baseURL: 'http://localhost:5000/api'
});

// Get featured items
const featured = await api.public.getFeaturedItems();

// Submit contact form
await api.public.submitContact({
  name: 'John Doe',
  email: 'john@example.com',
  message: 'Inquiry'
});
```

#### Web Application SDK
```typescript
import { WebAppAPI } from '@[project]/sdk';

const api = new WebAppAPI({
  baseURL: 'http://localhost:5000/api',
  getToken: async () => await clerk.session?.getToken()
});

// Get items
const items = await api.items.list({ featured: true });

// Add to collection
await api.collections.addItem({ itemId: 'cuid', quantity: 2 });
```

## CORS Configuration

### Development
```python
CORS_ORIGINS = [
    "http://localhost:3000",  # Landing site
    "http://localhost:5173",  # Web application
]
```

### Production
```python
CORS_ORIGINS = [
    "https://yourdomain.com",      # Landing site
    "https://app.yourdomain.com",  # Web application
]
```