# Complete REST API Documentation
# Enterprise E-Commerce Marketplace Platform

**Document Version**: 1.0  
**Status**: In Development  
**Base URL**: https://api.marketplace.com/api/v1  
**Last Updated**: 2026-06-12

---

## Table of Contents
1. [API Overview](#api-overview)
2. [Authentication APIs](#authentication-apis)
3. [Product APIs](#product-apis)
4. [Cart APIs](#cart-apis)
5. [Order APIs](#order-apis)
6. [Auction APIs](#auction-apis)
7. [Payment APIs](#payment-apis)
8. [Review APIs](#review-apis)
9. [User APIs](#user-apis)
10. [Seller APIs](#seller-apis)
11. [Admin APIs](#admin-apis)
12. [Error Handling](#error-handling)
13. [Rate Limiting](#rate-limiting)

---

## API Overview

### API Standards

- **Protocol**: HTTPS (TLS 1.3)
- **Authentication**: JWT Bearer Tokens
- **Format**: JSON
- **Versioning**: URL-based (/api/v1)
- **Response Format**: Consistent envelope
- **Pagination**: Cursor or offset-based
- **Rate Limiting**: 1000 requests/minute per user

### Standard Response Format

#### Success Response
```json
{
  "success": true,
  "data": {},
  "message": "Operation successful",
  "timestamp": "2026-06-12T12:00:00Z"
}
```

#### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

#### Paginated Response
```json
{
  "success": true,
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 100,
    "totalPages": 5,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

## Authentication APIs

### 1. User Registration

**Endpoint**: `POST /auth/register`

**Authentication**: None (Public)

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "phone": "+1234567890",
  "userType": "BUYER" // or "SELLER"
}
```

**Validation Rules**:
- Email: Valid email format, unique in system
- Password: Min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char
- FirstName: Max 100 chars
- LastName: Max 100 chars
- Phone: Optional, valid format if provided

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "userId": 1,
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "message": "Verification email sent. Please check your inbox."
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

**Error Responses**:
- `400 Bad Request`: Validation failed
- `409 Conflict`: Email already exists

---

### 2. User Login

**Endpoint**: `POST /auth/login`

**Authentication**: None (Public)

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "rememberMe": false
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 86400,
    "user": {
      "userId": 1,
      "email": "user@example.com",
      "firstName": "John",
      "roles": ["BUYER"]
    }
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid credentials
- `429 Too Many Requests`: Account locked (after 5 failed attempts)

---

### 3. Refresh Token

**Endpoint**: `POST /auth/refresh-token`

**Authentication**: Bearer token (Refresh Token)

**Request Body**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 86400
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 4. Logout

**Endpoint**: `POST /auth/logout`

**Authentication**: Bearer token (Required)

**Response (200 OK)**:
```json
{
  "success": true,
  "data": null,
  "message": "Logged out successfully",
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 5. Forgot Password

**Endpoint**: `POST /auth/forgot-password`

**Authentication**: None (Public)

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": null,
  "message": "Password reset link sent to your email",
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 6. Reset Password

**Endpoint**: `POST /auth/reset-password`

**Authentication**: None (Public, but requires valid token)

**Request Body**:
```json
{
  "token": "reset-token-from-email",
  "newPassword": "NewSecurePass123!"
}
```

**Validation**: Password must meet strength requirements

**Response (200 OK)**:
```json
{
  "success": true,
  "data": null,
  "message": "Password reset successfully",
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

## Product APIs

### 1. Get All Products

**Endpoint**: `GET /products`

**Authentication**: Optional (public read)

**Query Parameters**:
```
page: int (default: 1)
pageSize: int (default: 20, max: 100)
search: string (optional, full-text search)
categoryId: int (optional)
minPrice: decimal (optional)
maxPrice: decimal (optional)
rating: int (optional, 1-5)
sortBy: string (optional: "price_asc", "price_desc", "rating", "newest")
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": [
    {
      "productId": 1,
      "name": "Product Name",
      "description": "Product description",
      "price": 99.99,
      "averageRating": 4.5,
      "reviewCount": 25,
      "seller": {
        "sellerId": 1,
        "businessName": "Business Name",
        "rating": 4.8
      },
      "images": [
        {
          "imageId": 1,
          "url": "https://cdn.example.com/image.jpg",
          "isPrimary": true
        }
      ],
      "status": "ACTIVE",
      "createdAt": "2026-06-12T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 100,
    "totalPages": 5
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 2. Get Product by ID

**Endpoint**: `GET /products/{id}`

**Authentication**: Optional

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "productId": 1,
    "name": "Product Name",
    "description": "Detailed product description",
    "price": 99.99,
    "sku": "SKU-001",
    "category": {
      "categoryId": 1,
      "name": "Electronics"
    },
    "seller": {
      "sellerId": 1,
      "businessName": "Business Name",
      "rating": 4.8,
      "totalSales": 1500
    },
    "images": [],
    "variants": [],
    "inventory": {
      "quantityOnHand": 50,
      "quantityReserved": 5,
      "quantityAvailable": 45
    },
    "averageRating": 4.5,
    "reviewCount": 25,
    "reviews": [
      {
        "reviewId": 1,
        "rating": 5,
        "title": "Great product",
        "comment": "Exactly as described",
        "reviewer": "John D.",
        "createdAt": "2026-06-12T12:00:00Z"
      }
    ],
    "status": "ACTIVE",
    "createdAt": "2026-06-12T12:00:00Z"
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 3. Create Product

**Endpoint**: `POST /products`

**Authentication**: Bearer token (Seller role required)

**Request Body**:
```json
{
  "name": "Product Name",
  "description": "Product description",
  "price": 99.99,
  "categoryId": 1,
  "sku": "SKU-001",
  "tags": ["electronics", "gadget"],
  "variants": [
    {
      "name": "Color",
      "value": "Black",
      "priceModifier": 0
    }
  ]
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "productId": 1,
    "name": "Product Name",
    "status": "DRAFT",
    "message": "Product created. Pending admin review before publication."
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

**Authorization**: Only sellers can create products

---

### 4. Update Product

**Endpoint**: `PUT /products/{id}`

**Authentication**: Bearer token (Seller role required)

**Authorization**: Only product owner or admin can update

**Request Body**: Same as create product

**Response (200 OK)**: Updated product object

---

### 5. Delete Product

**Endpoint**: `DELETE /products/{id}`

**Authentication**: Bearer token (Seller role required)

**Authorization**: Only product owner or admin

**Response (204 No Content)**

---

### 6. Upload Product Images

**Endpoint**: `POST /products/{id}/images`

**Authentication**: Bearer token (Seller role required)

**Content-Type**: `multipart/form-data`

**Request Body**:
```
files: File[] (max 10 files, max 5MB each)
```

**Response (201 Created)**:
```json
{
  "success": true,
  "data": [
    {
      "imageId": 1,
      "url": "https://cdn.example.com/image1.jpg",
      "thumbnailUrl": "https://cdn.example.com/thumb1.jpg",
      "isPrimary": true
    }
  ],
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

## Cart APIs

### 1. Get Cart

**Endpoint**: `GET /carts`

**Authentication**: Bearer token (Required)

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "cartId": 1,
    "items": [
      {
        "cartItemId": 1,
        "product": {
          "productId": 1,
          "name": "Product Name",
          "price": 99.99
        },
        "quantity": 2,
        "variants": {
          "color": "Black",
          "size": "Large"
        },
        "subtotal": 199.98
      }
    ],
    "subtotal": 199.98,
    "tax": 19.99,
    "total": 219.97,
    "updatedAt": "2026-06-12T12:00:00Z"
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 2. Add to Cart

**Endpoint**: `POST /carts/items`

**Authentication**: Bearer token (Required)

**Request Body**:
```json
{
  "productId": 1,
  "quantity": 2,
  "variants": {
    "color": "Black",
    "size": "Large"
  }
}
```

**Validation**: 
- Product must exist and be active
- Quantity must be available
- Variants must be valid for product

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "cartItemId": 1,
    "message": "Item added to cart"
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 3. Update Cart Item

**Endpoint**: `PUT /carts/items/{cartItemId}`

**Authentication**: Bearer token (Required)

**Request Body**:
```json
{
  "quantity": 3
}
```

**Response (200 OK)**: Updated cart item

---

### 4. Remove from Cart

**Endpoint**: `DELETE /carts/items/{cartItemId}`

**Authentication**: Bearer token (Required)

**Response (204 No Content)**

---

## Order APIs

### 1. Create Order (Checkout)

**Endpoint**: `POST /orders`

**Authentication**: Bearer token (Required)

**Request Body**:
```json
{
  "shippingAddress": {
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "country": "USA"
  },
  "shippingMethod": "STANDARD",
  "paymentMethod": "STRIPE"
}
```

**Processing**:
1. Validate cart not empty
2. Validate stock availability
3. Create order(s) by seller
4. Reserve stock
5. Generate payment request
6. Clear cart

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "orderIds": [1, 2],
    "totalAmount": 219.97,
    "paymentUrl": "https://payment.gateway.com/pay?token=...",
    "message": "Order created. Redirecting to payment..."
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 2. Get Orders

**Endpoint**: `GET /orders`

**Authentication**: Bearer token (Required)

**Query Parameters**:
```
status: string (optional: PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED)
page: int (default: 1)
pageSize: int (default: 20)
```

**Response (200 OK)**: Paginated list of orders

---

### 3. Get Order by ID

**Endpoint**: `GET /orders/{id}`

**Authentication**: Bearer token (Required)

**Authorization**: Only order buyer, seller, or admin can view

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "orderId": 1,
    "orderNumber": "ORD-2026-000001",
    "status": "CONFIRMED",
    "items": [],
    "buyer": {
      "userId": 1,
      "email": "buyer@example.com",
      "name": "John Doe"
    },
    "seller": {
      "sellerId": 1,
      "businessName": "Seller Business"
    },
    "subtotal": 199.98,
    "tax": 19.99,
    "shipping": 10.00,
    "total": 229.97,
    "shippingAddress": {},
    "trackingNumber": "TRACK123456",
    "createdAt": "2026-06-12T12:00:00Z",
    "shippedAt": null,
    "deliveredAt": null
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 4. Update Order Status

**Endpoint**: `PUT /orders/{id}/status`

**Authentication**: Bearer token (Required)

**Authorization**: Only seller or admin can update

**Request Body**:
```json
{
  "status": "SHIPPED",
  "trackingNumber": "TRACK123456"
}
```

**Valid Transitions**:
- PENDING → CONFIRMED
- CONFIRMED → PROCESSING
- PROCESSING → SHIPPED
- SHIPPED → DELIVERED
- Any → CANCELLED

**Response (200 OK)**: Updated order

---

## Auction APIs

### 1. Get Active Auctions

**Endpoint**: `GET /auctions`

**Authentication**: Optional

**Query Parameters**:
```
status: string (optional: DRAFT, ACTIVE, ENDED)
page: int (default: 1)
pageSize: int (default: 20)
```

**Response (200 OK)**: Paginated list of auctions

---

### 2. Get Auction by ID

**Endpoint**: `GET /auctions/{id}`

**Authentication**: Optional

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "auctionId": 1,
    "product": {
      "productId": 1,
      "name": "Product Name",
      "images": []
    },
    "seller": {
      "sellerId": 1,
      "businessName": "Seller Business"
    },
    "startPrice": 50.00,
    "reservePrice": 75.00,
    "currentHighestBid": 85.00,
    "bidIncrement": 5.00,
    "status": "ACTIVE",
    "startTime": "2026-06-12T12:00:00Z",
    "endTime": "2026-06-13T12:00:00Z",
    "timeRemaining": 3600,
    "bidCount": 12,
    "currentHighestBidder": "John D.",
    "winner": null,
    "bids": [
      {
        "bidId": 1,
        "bidder": "Anonymous",
        "amount": 85.00,
        "bidTime": "2026-06-12T11:30:00Z"
      }
    ]
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 3. Create Auction

**Endpoint**: `POST /auctions`

**Authentication**: Bearer token (Seller role required)

**Request Body**:
```json
{
  "productId": 1,
  "startPrice": 50.00,
  "reservePrice": 75.00,
  "bidIncrement": 5.00,
  "startTime": "2026-06-13T12:00:00Z",
  "endTime": "2026-06-14T12:00:00Z"
}
```

**Validation**:
- Product must be owned by seller
- Start time must be in future
- End time must be after start time
- Reserve price ≥ start price
- Cannot have active auction for same product

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "auctionId": 1,
    "status": "SCHEDULED"
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 4. Place Bid

**Endpoint**: `POST /auctions/{id}/bids`

**Authentication**: Bearer token (Required)

**Request Body**:
```json
{
  "bidAmount": 90.00
}
```

**Bid Validation**:
1. Auction must be active
2. Bid > current highest bid + bid increment
3. Bidder cannot be auction creator
4. Bidder must be verified buyer
5. Concurrent bid conflict resolution

**Real-Time Broadcasting**: All connected clients receive bid update via SignalR

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "bidId": 1,
    "amount": 90.00,
    "message": "Bid placed successfully",
    "newHighestBid": 90.00,
    "timeRemaining": 3600
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

**Error Responses**:
- `400 Bad Request`: Bid too low
- `409 Conflict`: Concurrent bid conflict
- `410 Gone`: Auction already ended

---

### 5. Get Auction Bids

**Endpoint**: `GET /auctions/{id}/bids`

**Authentication**: Optional

**Query Parameters**:
```
page: int (default: 1)
pageSize: int (default: 20)
```

**Response (200 OK)**: Paginated bid history

---

## Review APIs

### 1. Create Review

**Endpoint**: `POST /reviews`

**Authentication**: Bearer token (Required)

**Request Body**:
```json
{
  "productId": 1,
  "orderId": 1,
  "rating": 5,
  "title": "Excellent product",
  "comment": "Exceeded my expectations",
  "images": []
}
```

**Validation**:
- User must have purchased product (verified order)
- One review per order only
- Rating 1-5
- Title max 255 chars
- Comment max 1000 chars

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "reviewId": 1,
    "status": "PENDING"
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

### 2. Get Product Reviews

**Endpoint**: `GET /products/{id}/reviews`

**Authentication**: Optional

**Query Parameters**:
```
rating: int (optional, 1-5 to filter)
sortBy: string (optional: "helpful", "newest", "rating_high", "rating_low")
page: int (default: 1)
pageSize: int (default: 20)
```

**Response (200 OK)**: Paginated reviews

---

## Error Handling

### HTTP Status Codes

| Code | Meaning |
|------|----------|
| 200 | OK - Successful GET/PUT request |
| 201 | Created - Successful POST request |
| 204 | No Content - Successful DELETE request |
| 400 | Bad Request - Invalid input |
| 401 | Unauthorized - Missing/invalid authentication |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource conflict (duplicate, state) |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |
| 503 | Service Unavailable - Maintenance/overload |

### Common Error Codes

```json
{
  "VALIDATION_ERROR": "Input validation failed",
  "AUTHENTICATION_FAILED": "Invalid credentials",
  "UNAUTHORIZED": "Insufficient permissions",
  "RESOURCE_NOT_FOUND": "Resource doesn't exist",
  "DUPLICATE_RESOURCE": "Resource already exists",
  "INVALID_STATE": "Invalid operation for current state",
  "PAYMENT_FAILED": "Payment processing failed",
  "AUCTION_ENDED": "Auction is no longer active",
  "BID_TOO_LOW": "Bid amount too low",
  "OUT_OF_STOCK": "Product out of stock"
}
```

---

## Rate Limiting

### Rate Limits

- **Authenticated Users**: 1000 requests/minute
- **Anonymous Users**: 100 requests/minute
- **Auth Endpoints**: 5 login attempts/minute per IP

### Rate Limit Headers

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1623509400
```

### Exceeding Rate Limit

Returns `429 Too Many Requests`:
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again later.",
    "retryAfter": 60
  },
  "timestamp": "2026-06-12T12:00:00Z"
}
```

---

## Additional Endpoints

### Admin Endpoints

**Approve Seller**:
```
PUT /admin/sellers/{id}/verify
Body: { "status": "APPROVED", "notes": "Verified" }
```

**Moderate Product**:
```
PUT /admin/products/{id}/status
Body: { "status": "APPROVED", "notes": "Passed review" }
```

**Get System Analytics**:
```
GET /admin/analytics
  ?startDate=2026-06-01&endDate=2026-06-30
```

### Seller Endpoints

**Get Seller Analytics**:
```
GET /sellers/analytics
  ?startDate=2026-06-01&endDate=2026-06-30
```

**Update Store Settings**:
```
PUT /sellers/{id}/store
Body: { "storeName": "", "description": "" }
```

---

**Document Version**: 1.0  
**Status**: In Development  
**Last Updated**: 2026-06-12
