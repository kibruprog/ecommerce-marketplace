# System Architecture - Enterprise E-Commerce Marketplace Platform

**Document Version**: 1.0  
**Status**: In Development  
**Last Updated**: 2026-06-12

---

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Layered Architecture](#layered-architecture)
3. [Design Patterns](#design-patterns)
4. [SOLID Principles](#solid-principles)
5. [Technology Stack Integration](#technology-stack-integration)
6. [Data Flow](#data-flow)
7. [Security Architecture](#security-architecture)
8. [Scalability Architecture](#scalability-architecture)

---

## Architecture Overview

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENT LAYER                               │
├─────────────────────────────────────────────────────────────────┤
│  React Frontend     │  Mobile App (Phase 2)  │  Admin Dashboard │
│  (Port 3000)        │                        │  (Port 3001)     │
└──────────┬──────────────────────────────────────────────────────┘
           │ HTTPS
           │ WebSocket (SignalR)
           ↓
┌─────────────────────────────────────────────────────────────────┐
│                    API GATEWAY / LOAD BALANCER                  │
│  (AWS ALB / CloudFlare)                                         │
├─────────────────────────────────────────────────────────────────┤
│  - HTTPS Termination  │  - Rate Limiting  │  - CORS Handling   │
│  - Request Routing    │  - Compression    │  - Web Application │
│  - SSL/TLS            │  - Caching        │    Firewall (WAF)  │
└──────────┬────────────────────────────────────────────────────┬─┘
           │                                                   │
           ↓                                                   ↓
┌────────────────────────────────────┐  ┌────────────────────────┐
│     ASP.NET CORE API SERVERS       │  │   SIGNALR HUB          │
│  (ECS/EKS - Auto Scaling)          │  │   (Real-time Updates)  │
├────────────────────────────────────┤  ├────────────────────────┤
│ Instance 1  Instance 2  Instance N │  │  WebSocket Handler     │
│  (Port 5000)                       │  │  (Port 5001)           │
│                                    │  │                        │
│  ┌──────────────────────────────┐ │  │  ┌──────────────────┐  │
│  │   Presentation Layer         │ │  │  │  Auction Hub     │  │
│  │   - API Controllers          │ │  │  │  - Bid Updates   │  │
│  │   - Request/Response         │ │  │  │  - Notifications │  │
│  │   - Input Validation         │ │  │  └──────────────────┘  │
│  └────────────┬─────────────────┘ │  └────────────────────────┘
│               ↓                    │
│  ┌──────────────────────────────┐ │
│  │   Service Layer              │ │
│  │   - Business Logic           │ │
│  │   - Transaction Handling     │ │
│  │   - Validation               │ │
│  │   - Auction Engine           │ │
│  │   - Payment Processing       │ │
│  └────────────┬─────────────────┘ │
│               ↓                    │
│  ┌──────────────────────────────┐ │
│  │   Repository Layer           │ │
│  │   - Unit of Work Pattern     │ │
│  │   - Data Access              │ │
│  │   - Query Building           │ │
│  └────────────┬─────────────────┘ │
│               ↓                    │
└───────────────┼────────────────────┘
                ↓
┌────────────────────────────────────────────────────────────────┐
│                    DATA ACCESS LAYER                           │
│              Entity Framework Core 10 (ORM)                    │
├────────────────────────────────────────────────────────────────┤
│  - DbContext  │  - Migrations  │  - Query Translation         │
└────────────────────┬─────────────────────────────────────────┬─┘
                     │                                         │
        ┌────────────┼─────────────────┐                       │
        ↓            ↓                 ↓                       ↓
┌──────────────┐ ┌────────────┐  ┌────────────┐  ┌──────────────────┐
│ SQL SERVER   │ │   REDIS    │  │  AWS S3    │  │  EXTERNAL APIs   │
│ (Production) │ │  (Cache)   │  │  (Images)  │  │  - Stripe        │
├──────────────┤ ├────────────┤  ├────────────┤  │  - PayPal        │
│ Multi-AZ     │ │ Session    │  │ CDN via    │  │  - Chapa         │
│ Read Replicas│ │ Caching    │  │ CloudFront │  │  - Telebirr      │
│ Backup       │ │ Real-time  │  │ Compression│  │  - SendGrid      │
│ Restore      │ │ Data       │  │ Versioning │  └──────────────────┘
└──────────────┘ └────────────┘  └────────────┘
```

---

## Layered Architecture

### 1. Presentation Layer

**Responsibility**: Handle HTTP requests/responses, input validation

**Components**:
- **API Controllers**: Handle REST endpoints
- **Request DTOs**: Input validation and serialization
- **Response DTOs**: Consistent response format
- **Middleware**: CORS, logging, error handling

**Key Classes**:
```csharp
// Example Controller
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    
    [HttpGet("{id}")]
    [Authorize]
    public async Task<IActionResult> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        return Ok(product);
    }
}
```

### 2. API Layer

**Responsibility**: Route requests, apply policies, handle cross-cutting concerns

**Components**:
- **Routing**: Route products, orders, auctions, etc.
- **Authorization**: Policy-based access control
- **Versioning**: API v1, v2, etc.
- **Error Handling**: Global exception handling middleware
- **Logging**: Request/response logging

**Middleware Stack** (in order):
1. HTTPS Redirect
2. CORS
3. Custom Exception Handler
4. Request Logging
5. Authentication (JWT)
6. Authorization (RBAC)
7. Rate Limiting

### 3. Service Layer

**Responsibility**: Implement business logic, orchestrate operations

**Components**:
- **Domain Services**: Core business logic
- **Application Services**: Coordinate operations
- **Validators**: Business rule validation
- **Transaction Handling**: Multi-step operations

**Service Examples**:
```csharp
public interface IProductService
{
    Task<ProductDto> GetProductByIdAsync(int id);
    Task<ProductDto> CreateProductAsync(CreateProductRequest request);
    Task<PagedResult<ProductDto>> SearchProductsAsync(SearchRequest search);
}

public interface IOrderService
{
    Task<OrderDto> CreateOrderAsync(CreateOrderRequest request);
    Task<OrderDto> GetOrderAsync(int orderId);
    Task UpdateOrderStatusAsync(int orderId, OrderStatus status);
}

public interface IAuctionService
{
    Task<AuctionDto> CreateAuctionAsync(CreateAuctionRequest request);
    Task<BidDto> PlaceBidAsync(int auctionId, decimal amount);
    Task<AuctionResultDto> ConcludeAuctionAsync(int auctionId);
}
```

### 4. Repository Layer

**Responsibility**: Abstract data access operations

**Pattern**: Repository Pattern + Unit of Work

```csharp
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task<T> AddAsync(T entity);
    Task<T> UpdateAsync(T entity);
    Task<bool> DeleteAsync(int id);
}

public interface IUnitOfWork : IDisposable
{
    IRepository<Product> Products { get; }
    IRepository<Order> Orders { get; }
    IRepository<Auction> Auctions { get; }
    IRepository<User> Users { get; }
    
    Task<int> SaveChangesAsync();
}
```

### 5. Database Layer

**Responsibility**: Persist and retrieve data

**Technology**: Entity Framework Core 10

**Components**:
- **DbContext**: Represents the database session
- **Entities**: Domain objects
- **Migrations**: Schema versioning
- **Queries**: LINQ-to-SQL translations

---

## Design Patterns

### 1. Repository Pattern

**Purpose**: Abstract data access from business logic

**Benefits**:
- Testable service layer
- Switchable data sources
- Centralized query logic

### 2. Unit of Work Pattern

**Purpose**: Manage transactions across multiple repositories

**Implementation**:
```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    
    public IRepository<Order> Orders { get; private set; }
    public IRepository<OrderItem> OrderItems { get; private set; }
    public IRepository<Payment> Payments { get; private set; }
    
    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }
}
```

### 3. Service Layer Pattern

**Purpose**: Encapsulate business logic

**Benefit**: Reusable across API, SignalR, background jobs

### 4. Strategy Pattern (Payment Processing)

**Purpose**: Support multiple payment methods

```csharp
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(Payment payment);
    Task<RefundResult> RefundAsync(Payment payment);
}

public class StripePaymentStrategy : IPaymentStrategy { }
public class PayPalPaymentStrategy : IPaymentStrategy { }
public class ChapaPaymentStrategy : IPaymentStrategy { }
public class TelebirrPaymentStrategy : IPaymentStrategy { }
```

### 5. Factory Pattern (Payment Gateway)

**Purpose**: Create appropriate payment strategy

```csharp
public class PaymentStrategyFactory
{
    public IPaymentStrategy CreateStrategy(PaymentMethod method)
    {
        return method switch
        {
            PaymentMethod.Stripe => new StripePaymentStrategy(),
            PaymentMethod.PayPal => new PayPalPaymentStrategy(),
            PaymentMethod.Chapa => new ChapaPaymentStrategy(),
            PaymentMethod.Telebirr => new TelebirrPaymentStrategy(),
            _ => throw new InvalidOperationException()
        };
    }
}
```

### 6. Mediator Pattern (Commands/Queries) - Optional

**Purpose**: Decouple command/query handlers from controllers

**Using MediatR library**

### 7. Observer Pattern (SignalR)

**Purpose**: Notify clients of real-time events

**Events**:
- Bid placed
- Auction ending
- Order status updated
- Notification received

---

## SOLID Principles

### Single Responsibility Principle (SRP)

**Application**:
- ProductService: Manages products only
- OrderService: Manages orders only
- AuctionService: Manages auctions only
- Each repository: Single entity type

### Open/Closed Principle (OCP)

**Application**:
- Payment strategies extend IPaymentStrategy without modification
- New payment methods added by implementing interface
- Services extensible without changing core code

### Liskov Substitution Principle (LSP)

**Application**:
- All payment strategies interchangeable
- All repositories implement same contract
- Services depend on abstractions

### Interface Segregation Principle (ISP)

**Application**:
```csharp
// Segregated interfaces instead of one large interface
public interface IProductReadService
{
    Task<ProductDto> GetByIdAsync(int id);
    Task<IEnumerable<ProductDto>> SearchAsync(SearchRequest request);
}

public interface IProductWriteService
{
    Task<ProductDto> CreateAsync(CreateProductRequest request);
    Task<ProductDto> UpdateAsync(UpdateProductRequest request);
}
```

### Dependency Inversion Principle (DIP)

**Application**:
```csharp
// Controllers depend on interfaces, not concrete classes
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }
}

// Dependency Injection in Startup
services.AddScoped<IOrderService, OrderService>();
```

---

## Technology Stack Integration

### Backend Stack

```
ASP.NET Core 10
├── Controllers (API endpoints)
├── Middleware (Cross-cutting concerns)
├── Dependency Injection (Built-in)
├── Configuration Management
└── Logging Framework (Serilog)

Entity Framework Core 10
├── DbContext
├── Migrations
├── LINQ Queries
└── Change Tracking

SignalR
├── Real-time Hub
├── WebSocket Connection
├── Broadcast Messages
└── Group Management

JWT Authentication
├── Bearer Token Generation
├── Refresh Tokens
├── Claims-based Authorization
└── Role-based Access Control (RBAC)

FluentValidation
├── DTO Validation
├── Business Rule Validation
└── Fluent API

AutoMapper
├── DTO Mapping
├── Entity Mapping
└── Projection Support
```

### Frontend Stack

```
React 18+
├── Functional Components
├── Hooks (useState, useEffect, useContext)
├── React Router v6
└── Context API for State Management

Material UI (MUI)
├── Pre-built Components
├── Theming System
├── Responsive Grid
└── Icons & Typography

Axios
├── HTTP Client
├── Request Interceptors
├── Response Interceptors
└── Error Handling

React Hook Form
├── Form Management
├── Validation Integration
├── Performance Optimization
└── Error Handling

Recharts
├── Chart Visualization
├── Real-time Data Updates
├── Responsive Charts
└── Customizable Themes
```

---

## Data Flow

### Typical Request Flow

```
1. Client Request (React)
   ↓
2. Axios Interceptor (Add JWT token)
   ↓
3. HTTPS/TLS Encryption
   ↓
4. Load Balancer (Route to available server)
   ↓
5. ASP.NET Core API
   ├─ CORS Middleware
   ├─ Authentication (Validate JWT)
   ├─ Authorization (Check roles)
   ├─ Controller (Route to action)
   ├─ Service Layer (Execute business logic)
   ├─ Repository (Data access)
   ├─ EF Core (Query generation)
   └─ SQL Server (Execute query)
   ↓
6. Response DTO
   ↓
7. JSON Serialization
   ↓
8. Response to Client
   ↓
9. React State Update
   ↓
10. UI Re-render
```

### Real-Time Auction Bid Flow

```
1. Buyer places bid in React component
   ↓
2. Bid validation on client
   ↓
3. SignalR WebSocket connection to HubBid method
   ↓
4. Auction Hub receives bid
   ↓
5. AuctionService validates bid
   ├─ Check auction active
   ├─ Check bid amount
   ├─ Check bidder eligibility
   └─ Check concurrent conflicts
   ↓
6. Save bid to database
   ↓
7. Update auction highest bid
   ↓
8. Broadcast bid to all connected clients via SignalR
   ↓
9. All clients receive real-time update
   ↓
10. UI updates in real-time
```

---

## Security Architecture

### Authentication Flow

```
1. User enters email/password
   ↓
2. API validates credentials
   ├─ Query user by email
   ├─ Compare password hash (BCrypt)
   └─ Check account status
   ↓
3. Generate tokens
   ├─ Access Token (JWT, 24 hours)
   ├─ Refresh Token (stored, 7 days)
   └─ Return both to client
   ↓
4. Client stores tokens
   ├─ Access Token: Memory or LocalStorage
   ├─ Refresh Token: Secure HttpOnly Cookie
   └─ (Prevent XSS attacks)
   ↓
5. Subsequent requests include Access Token
   ├─ Token in Authorization header
   ├─ Format: Bearer <token>
   └─ Server validates on each request
   ↓
6. Token expiration handling
   ├─ Check token expiry
   ├─ If expired, use Refresh Token
   ├─ Get new Access Token
   └─ Continue operation
```

### Authorization Architecture

```
Role-Based Access Control (RBAC)
├─ Buyer
│  ├─ Browse Products
│  ├─ Purchase Orders
│  ├─ Place Auction Bids
│  └─ Leave Reviews
│
├─ Seller
│  ├─ Manage Products
│  ├─ Create Auctions
│  ├─ Manage Orders
│  └─ View Analytics
│
└─ Admin
   ├─ Manage Users
   ├─ Manage Sellers
   ├─ Moderate Products
   ├─ View System Analytics
   └─ Configure Settings

Implementation:
├─ JWT Claims include roles
├─ Policies define permissions
├─ Attributes on controllers enforce
└─ Middleware validates access
```

### Data Security

```
Encryption
├─ In Transit: TLS 1.3
├─ At Rest: AES-256
├─ Passwords: BCrypt (salt rounds: 12)
└─ Sensitive fields encrypted in database

Input Validation
├─ Client-side: React validation
├─ API-level: DTO validation
├─ Business-level: Service validation
└─ Database-level: Constraints

SQL Injection Prevention
├─ Parameterized queries via EF Core
├─ No string concatenation in queries
├─ LINQ-to-SQL translation
└─ Input sanitization
```

---

## Scalability Architecture

### Horizontal Scaling

```
Load Balancer (AWS ALB)
├─ API Server 1 (ECS Task)
├─ API Server 2 (ECS Task)
├─ API Server 3 (ECS Task)
└─ Auto-scaling group (min 2, max 10)

Benefits:
├─ Distribute load across instances
├─ Auto-scale based on CPU/Memory
├─ Stateless API (horizontal scalable)
└─ No session affinity needed
```

### Caching Layer

```
Redis Cache
├─ Session Cache (TTL: token expiry)
├─ Product Cache (TTL: 1 hour)
├─ Category Cache (TTL: 24 hours)
├─ Real-time Auction Data
└─ Rate Limiting Counters

Benefits:
├─ Reduce database load
├─ Faster response times
├─ Support high concurrency
└─ Real-time data availability
```

### Database Scaling

```
SQL Server
├─ Write Master (Multi-AZ)
│  └─ Automatic Failover
├─ Read Replicas
│  ├─ Replica 1 (Region 2)
│  └─ Replica 2 (Region 3)
└─ Sharding ready (by seller/time)

Benefits:
├─ Read-heavy queries to replicas
├─ Write consistency maintained
├─ Disaster recovery capability
└─ Horizontal scalability path
```

---

**Document Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2026-06-12
