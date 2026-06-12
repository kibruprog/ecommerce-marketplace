# Database Design - Enterprise E-Commerce Marketplace Platform

**Document Version**: 1.0  
**Status**: In Development  
**Last Updated**: 2026-06-12

---

## Table of Contents
1. [Database Overview](#database-overview)
2. [ER Diagram](#er-diagram)
3. [Database Schema](#database-schema)
4. [Normalization Analysis](#normalization-analysis)
5. [Indexing Strategy](#indexing-strategy)
6. [Relationships](#relationships)
7. [Design Decisions](#design-decisions)
8. [Performance Considerations](#performance-considerations)

---

## Database Overview

### Database Choice

- **Development**: SQLite (file-based, no setup required)
- **Production**: SQL Server 2019+ (enterprise-grade, scalable)
- **Caching Layer**: Redis (session, product cache)
- **Search**: Full-text search (SQL Server native)

### Design Principles

1. **Normalization**: 3NF (Third Normal Form)
2. **Data Integrity**: Foreign keys, constraints, defaults
3. **Performance**: Strategic indexing, partitioning ready
4. **Audit Trail**: Soft deletes, created/updated timestamps
5. **Scalability**: Support for millions of records

---

## ER Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATABASE ENTITIES                           │
└─────────────────────────────────────────────────────────────────┘

                          USERS
                     ┌────────────────┐
                     │ UserID (PK)    │
                     │ Email          │
                     │ PasswordHash   │
                     │ FirstName      │
                     │ LastName       │
                     │ Phone          │
                     │ CreatedAt      │
                     │ UpdatedAt      │
                     │ IsDeleted      │
                     └────────────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
         ┌──┴──┐      ┌─────┴─────┐    ┌──┴──┐
         │     │      │           │    │     │
        ROLES REFRESH SELLERS ADMINS BUYERS
        TOKENS

                        SELLERS
                   ┌────────────────┐
                   │ SellerID (PK)  │
                   │ UserID (FK)    │
                   │ BusinessName   │
                   │ VerificationStatus│
                   │ KYCDocPath     │
                   │ BankAccount    │
                   │ CommissionRate │
                   └────────────────┘
                            │
                            ├────────┬──────────┐
                            │        │          │
                          STORES PRODUCTS AUCTIONS

                       CATEGORIES
                   ┌────────────────┐
                   │ CategoryID (PK)│
                   │ Name           │
                   │ Description    │
                   │ ParentID (FK)  │
                   │ DisplayOrder   │
                   └────────────────┘
                            │
                            ├────────┐
                            │        │
                        PRODUCTS  TAGS

                       PRODUCTS
                   ┌────────────────┐
                   │ ProductID (PK) │
                   │ SellerID (FK)  │
                   │ CategoryID(FK) │
                   │ Name           │
                   │ Description    │
                   │ Price          │
                   │ Status         │
                   │ CreatedAt      │
                   └────────────────┘
                            │
            ┌───────────┬───┼───┬──────────┐
            │           │   │   │          │
      PRODUCT_  PRODUCT_  │ WISHLIST REVIEWS
      IMAGES  VARIANTS   │
                     INVENTORY

                       ORDERS
                   ┌────────────────┐
                   │ OrderID (PK)   │
                   │ BuyerID (FK)   │
                   │ SellerID (FK)  │
                   │ Status         │
                   │ TotalAmount    │
                   │ TaxAmount      │
                   │ ShippingAddress│
                   │ CreatedAt      │
                   └────────────────┘
                            │
            ┌───────────────┼──────────────┐
            │               │              │
      ORDER_ITEMS     PAYMENTS      SHIPMENTS

                      AUCTIONS
                   ┌────────────────┐
                   │ AuctionID (PK) │
                   │ ProductID (FK) │
                   │ SellerID (FK)  │
                   │ StartPrice     │
                   │ ReservePrice   │
                   │ BidIncrement   │
                   │ StartTime      │
                   │ EndTime        │
                   │ Status         │
                   └────────────────┘
                            │
                          BIDS
```

---

## Database Schema

### Core Tables

#### 1. Users
```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY IDENTITY(1,1),
    Email NVARCHAR(255) NOT NULL UNIQUE,
    PasswordHash NVARCHAR(255) NOT NULL,
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    Phone NVARCHAR(20),
    ProfileImagePath NVARCHAR(500),
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    LastLoginAt DATETIME,
    IsActive BIT DEFAULT 1,
    IsDeleted BIT DEFAULT 0,
    INDEX IX_Email (Email),
    INDEX IX_CreatedAt (CreatedAt)
);
```

#### 2. Roles
```sql
CREATE TABLE Roles (
    RoleID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(50) NOT NULL UNIQUE,
    Description NVARCHAR(255),
    CreatedAt DATETIME DEFAULT GETUTCDATE()
);

-- Insert default roles
INSERT INTO Roles (Name, Description) VALUES
('BUYER', 'Buyer role for purchasing products'),
('SELLER', 'Seller role for managing store'),
('ADMIN', 'Administrator role for system management');
```

#### 3. UserRoles (Junction Table)
```sql
CREATE TABLE UserRoles (
    UserRoleID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    RoleID INT NOT NULL FOREIGN KEY REFERENCES Roles(RoleID),
    AssignedAt DATETIME DEFAULT GETUTCDATE(),
    UNIQUE (UserID, RoleID),
    INDEX IX_UserID (UserID),
    INDEX IX_RoleID (RoleID)
);
```

#### 4. RefreshTokens
```sql
CREATE TABLE RefreshTokens (
    RefreshTokenID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    Token NVARCHAR(500) NOT NULL UNIQUE,
    ExpiresAt DATETIME NOT NULL,
    RevokedAt DATETIME,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    CreatedByIP NVARCHAR(50),
    IsExpired AS (CASE WHEN RevokedAt IS NOT NULL THEN 1 ELSE 0 END),
    INDEX IX_UserID (UserID),
    INDEX IX_Token (Token)
);
```

#### 5. Sellers
```sql
CREATE TABLE Sellers (
    SellerID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL UNIQUE FOREIGN KEY REFERENCES Users(UserID),
    BusinessName NVARCHAR(255) NOT NULL,
    BusinessDescription NVARCHAR(1000),
    VerificationStatus NVARCHAR(50) DEFAULT 'PENDING', -- PENDING, APPROVED, REJECTED
    KYCDocumentPath NVARCHAR(500),
    BankAccountName NVARCHAR(255),
    BankAccountNumber NVARCHAR(50),
    BankCode NVARCHAR(20),
    CommissionRate DECIMAL(5,2) DEFAULT 5.00,
    TotalSales DECIMAL(18,2) DEFAULT 0,
    AverageRating DECIMAL(3,2) DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    IsActive BIT DEFAULT 1,
    IsDeleted BIT DEFAULT 0,
    INDEX IX_VerificationStatus (VerificationStatus),
    INDEX IX_CreatedAt (CreatedAt)
);
```

#### 6. Stores
```sql
CREATE TABLE Stores (
    StoreID INT PRIMARY KEY IDENTITY(1,1),
    SellerID INT NOT NULL UNIQUE FOREIGN KEY REFERENCES Sellers(SellerID),
    StoreName NVARCHAR(255) NOT NULL,
    StoreDescription NVARCHAR(MAX),
    LogoPath NVARCHAR(500),
    BannerPath NVARCHAR(500),
    ContactEmail NVARCHAR(255),
    ContactPhone NVARCHAR(20),
    ReturnPolicy NVARCHAR(MAX),
    ShippingPolicy NVARCHAR(MAX),
    StoreHours NVARCHAR(500),
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_SellerID (SellerID)
);
```

#### 7. Categories
```sql
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(255) NOT NULL UNIQUE,
    Description NVARCHAR(1000),
    ParentCategoryID INT FOREIGN KEY REFERENCES Categories(CategoryID),
    DisplayOrder INT DEFAULT 0,
    ImagePath NVARCHAR(500),
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_Name (Name),
    INDEX IX_ParentCategoryID (ParentCategoryID)
);
```

#### 8. Products
```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    SellerID INT NOT NULL FOREIGN KEY REFERENCES Sellers(SellerID),
    CategoryID INT NOT NULL FOREIGN KEY REFERENCES Categories(CategoryID),
    Name NVARCHAR(255) NOT NULL,
    Description NVARCHAR(MAX),
    Price DECIMAL(18,2) NOT NULL,
    Status NVARCHAR(50) DEFAULT 'DRAFT', -- DRAFT, PENDING_REVIEW, ACTIVE, INACTIVE
    SKU NVARCHAR(100) UNIQUE,
    Weight DECIMAL(10,2),
    Dimensions NVARCHAR(100),
    AverageRating DECIMAL(3,2) DEFAULT 0,
    ReviewCount INT DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    IsDeleted BIT DEFAULT 0,
    INDEX IX_SellerID (SellerID),
    INDEX IX_CategoryID (CategoryID),
    INDEX IX_Name (Name),
    INDEX IX_Status (Status),
    FULLTEXT INDEX FTX_Product ON (Name, Description)
);
```

#### 9. ProductImages
```sql
CREATE TABLE ProductImages (
    ImageID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    ImagePath NVARCHAR(500) NOT NULL,
    ThumbnailPath NVARCHAR(500),
    DisplayOrder INT DEFAULT 0,
    IsPrimary BIT DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_ProductID (ProductID),
    INDEX IX_IsPrimary (IsPrimary)
);
```

#### 10. ProductVariants
```sql
CREATE TABLE ProductVariants (
    VariantID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    Name NVARCHAR(100) NOT NULL,
    Value NVARCHAR(100) NOT NULL,
    SKU NVARCHAR(100),
    PriceModifier DECIMAL(18,2) DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_ProductID (ProductID),
    INDEX IX_Name (Name)
);
```

#### 11. Inventory
```sql
CREATE TABLE Inventory (
    InventoryID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    QuantityOnHand INT DEFAULT 0,
    QuantityReserved INT DEFAULT 0,
    QuantityAvailable AS (QuantityOnHand - QuantityReserved),
    ReorderLevel INT DEFAULT 10,
    ReorderQuantity INT DEFAULT 50,
    LastRestockedAt DATETIME,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    UNIQUE (ProductID),
    INDEX IX_ProductID (ProductID),
    INDEX IX_QuantityAvailable (QuantityAvailable)
);
```

#### 12. Wishlists
```sql
CREATE TABLE Wishlists (
    WishlistID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    AddedAt DATETIME DEFAULT GETUTCDATE(),
    UNIQUE (UserID, ProductID),
    INDEX IX_UserID (UserID),
    INDEX IX_ProductID (ProductID)
);
```

#### 13. Carts
```sql
CREATE TABLE Carts (
    CartID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    ExpiresAt DATETIME,
    UNIQUE (UserID),
    INDEX IX_UserID (UserID),
    INDEX IX_ExpiresAt (ExpiresAt)
);
```

#### 14. CartItems
```sql
CREATE TABLE CartItems (
    CartItemID INT PRIMARY KEY IDENTITY(1,1),
    CartID INT NOT NULL FOREIGN KEY REFERENCES Carts(CartID),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    Quantity INT NOT NULL CHECK (Quantity > 0),
    Price DECIMAL(18,2) NOT NULL,
    VariantSelections NVARCHAR(MAX), -- JSON
    AddedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_CartID (CartID),
    INDEX IX_ProductID (ProductID)
);
```

#### 15. Orders
```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY IDENTITY(1,1),
    OrderNumber NVARCHAR(50) NOT NULL UNIQUE,
    BuyerID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    SellerID INT NOT NULL FOREIGN KEY REFERENCES Sellers(SellerID),
    Status NVARCHAR(50) DEFAULT 'PENDING', -- PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED
    SubtotalAmount DECIMAL(18,2) NOT NULL,
    TaxAmount DECIMAL(18,2) DEFAULT 0,
    ShippingAmount DECIMAL(18,2) DEFAULT 0,
    TotalAmount DECIMAL(18,2) NOT NULL,
    ShippingAddress NVARCHAR(MAX),
    ShippingMethod NVARCHAR(100),
    TrackingNumber NVARCHAR(100),
    Notes NVARCHAR(MAX),
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    ShippedAt DATETIME,
    DeliveredAt DATETIME,
    CancelledAt DATETIME,
    CancellationReason NVARCHAR(MAX),
    INDEX IX_BuyerID (BuyerID),
    INDEX IX_SellerID (SellerID),
    INDEX IX_Status (Status),
    INDEX IX_OrderNumber (OrderNumber),
    INDEX IX_CreatedAt (CreatedAt)
);
```

#### 16. OrderItems
```sql
CREATE TABLE OrderItems (
    OrderItemID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT NOT NULL FOREIGN KEY REFERENCES Orders(OrderID),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(18,2) NOT NULL,
    SubtotalPrice DECIMAL(18,2) NOT NULL,
    VariantSelections NVARCHAR(MAX), -- JSON
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_OrderID (OrderID),
    INDEX IX_ProductID (ProductID)
);
```

#### 17. Payments
```sql
CREATE TABLE Payments (
    PaymentID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT NOT NULL FOREIGN KEY REFERENCES Orders(OrderID),
    PaymentMethod NVARCHAR(50), -- STRIPE, PAYPAL, CHAPA, TELEBIRR
    Amount DECIMAL(18,2) NOT NULL,
    Status NVARCHAR(50) DEFAULT 'PENDING', -- PENDING, COMPLETED, FAILED, REFUNDED
    TransactionID NVARCHAR(255),
    GatewayResponse NVARCHAR(MAX), -- JSON
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    RefundedAt DATETIME,
    RefundAmount DECIMAL(18,2),
    INDEX IX_OrderID (OrderID),
    INDEX IX_Status (Status),
    INDEX IX_TransactionID (TransactionID)
);
```

#### 18. Reviews
```sql
CREATE TABLE Reviews (
    ReviewID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    OrderID INT NOT NULL FOREIGN KEY REFERENCES Orders(OrderID),
    BuyerID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    Rating INT NOT NULL CHECK (Rating >= 1 AND Rating <= 5),
    Title NVARCHAR(255),
    Comment NVARCHAR(MAX),
    IsVerifiedPurchase BIT DEFAULT 1,
    HelpfulCount INT DEFAULT 0,
    UnhelpfulCount INT DEFAULT 0,
    SellerResponse NVARCHAR(MAX),
    SellerResponseAt DATETIME,
    Status NVARCHAR(50) DEFAULT 'PENDING', -- PENDING, APPROVED, REJECTED
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    IsDeleted BIT DEFAULT 0,
    INDEX IX_ProductID (ProductID),
    INDEX IX_BuyerID (BuyerID),
    INDEX IX_Status (Status),
    INDEX IX_CreatedAt (CreatedAt)
);
```

#### 19. Auctions
```sql
CREATE TABLE Auctions (
    AuctionID INT PRIMARY KEY IDENTITY(1,1),
    ProductID INT NOT NULL FOREIGN KEY REFERENCES Products(ProductID),
    SellerID INT NOT NULL FOREIGN KEY REFERENCES Sellers(SellerID),
    StartPrice DECIMAL(18,2) NOT NULL,
    ReservePrice DECIMAL(18,2),
    BidIncrement DECIMAL(18,2) DEFAULT 1.00,
    Status NVARCHAR(50) DEFAULT 'DRAFT', -- DRAFT, SCHEDULED, ACTIVE, ENDED, CANCELLED
    StartTime DATETIME NOT NULL,
    EndTime DATETIME NOT NULL,
    ActualEndTime DATETIME,
    WinnerID INT FOREIGN KEY REFERENCES Users(UserID),
    WinningBidAmount DECIMAL(18,2),
    MinParticipants INT DEFAULT 0,
    CurrentHighestBidID INT,
    BidCount INT DEFAULT 0,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_SellerID (SellerID),
    INDEX IX_Status (Status),
    INDEX IX_StartTime (StartTime),
    INDEX IX_EndTime (EndTime),
    INDEX IX_WinnerID (WinnerID)
);
```

#### 20. Bids
```sql
CREATE TABLE Bids (
    BidID INT PRIMARY KEY IDENTITY(1,1),
    AuctionID INT NOT NULL FOREIGN KEY REFERENCES Auctions(AuctionID),
    BidderID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    BidAmount DECIMAL(18,2) NOT NULL,
    BidTime DATETIME DEFAULT GETUTCDATE(),
    IsWinningBid BIT DEFAULT 0,
    IPAddress NVARCHAR(50),
    INDEX IX_AuctionID (AuctionID),
    INDEX IX_BidderID (BidderID),
    INDEX IX_BidTime (BidTime),
    INDEX IX_IsWinningBid (IsWinningBid)
);
```

#### 21. Notifications
```sql
CREATE TABLE Notifications (
    NotificationID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT NOT NULL FOREIGN KEY REFERENCES Users(UserID),
    Type NVARCHAR(100), -- ORDER_CREATED, AUCTION_WON, PAYMENT_RECEIVED, etc.
    Title NVARCHAR(255),
    Message NVARCHAR(MAX),
    Data NVARCHAR(MAX), -- JSON with context data
    IsRead BIT DEFAULT 0,
    ReadAt DATETIME,
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_UserID (UserID),
    INDEX IX_IsRead (IsRead),
    INDEX IX_CreatedAt (CreatedAt)
);
```

#### 22. AuditLogs
```sql
CREATE TABLE AuditLogs (
    AuditLogID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT FOREIGN KEY REFERENCES Users(UserID),
    Action NVARCHAR(100),
    EntityType NVARCHAR(100),
    EntityID INT,
    OldValues NVARCHAR(MAX), -- JSON
    NewValues NVARCHAR(MAX), -- JSON
    IPAddress NVARCHAR(50),
    UserAgent NVARCHAR(500),
    CreatedAt DATETIME DEFAULT GETUTCDATE(),
    INDEX IX_UserID (UserID),
    INDEX IX_EntityType (EntityType),
    INDEX IX_CreatedAt (CreatedAt)
);
```

---

## Normalization Analysis

### First Normal Form (1NF)
✅ **Achieved**: All tables have atomic values (no repeating groups)
- ProductImages: Separate table for images
- CartItems: Separate table for cart items
- OrderItems: Separate table for order line items
- Bids: Separate table for auction bids

### Second Normal Form (2NF)
✅ **Achieved**: All non-key attributes depend on entire primary key
- All composite keys properly handled
- No partial dependencies
- Example: CartItems depends on both CartID and ProductID

### Third Normal Form (3NF)
✅ **Achieved**: No transitive dependencies
- AverageRating in Products is calculated, not stored redundantly
- TotalSales in Sellers is calculated from Orders
- Status fields in Orders and Auctions are normalized

---

## Indexing Strategy

### Primary Indexes (By Table)

| Table | Index Columns | Purpose |
|-------|---------------|----------|
| Users | Email | Fast login lookups |
| Users | CreatedAt | User growth tracking |
| Products | SellerID, CategoryID | Seller/category product lists |
| Products | Name (Full-Text) | Product search |
| Orders | BuyerID, SellerID, Status | Dashboard queries |
| Orders | CreatedAt | Time-based reporting |
| Auctions | Status, StartTime | Active auction queries |
| Bids | AuctionID, BidTime | Bid history |
| CartItems | CartID, ProductID | Cart operations |
| Reviews | ProductID, Status | Product reviews |

### Composite Indexes

```sql
-- Performance optimization indexes
CREATE INDEX IX_Orders_BuyerID_CreatedAt ON Orders(BuyerID, CreatedAt);
CREATE INDEX IX_Orders_SellerID_Status ON Orders(SellerID, Status);
CREATE INDEX IX_Products_SellerID_Status ON Products(SellerID, Status);
CREATE INDEX IX_Auctions_Status_EndTime ON Auctions(Status, EndTime);
```

---

## Relationships

### One-to-One Relationships
- User ↔ RefreshToken (One user can have multiple tokens)
- User ↔ Seller (One user is one seller, one seller is one user)
- Seller ↔ Store (One seller has one store)
- Product ↔ Inventory (One product has one inventory record)
- Cart ↔ User (One user has one active cart)

### One-to-Many Relationships
- Seller → Products (One seller has many products)
- Category → Products (One category has many products)
- Product → ProductImages (One product has many images)
- Product → ProductVariants (One product has many variants)
- Product → Reviews (One product has many reviews)
- Product → Auctions (One product can have many auctions)
- User → Orders (One user buys many orders)
- Order → OrderItems (One order has many items)
- Order → Payments (One order can have multiple payment attempts)
- Auction → Bids (One auction has many bids)
- User → Bids (One user places many bids)
- User → Notifications (One user receives many notifications)

### Many-to-Many Relationships
- User ↔ Role (Users have roles, roles have users) - through UserRoles table
- Product ↔ Tags (Products have tags, tags belong to products) - through ProductTags table
- User ↔ Wishlist (Users save products, products saved by users)

---

## Design Decisions

### 1. Soft Deletes
**Decision**: Use `IsDeleted` flag instead of physical deletion
**Reason**: 
- Maintain audit trail
- Enable data recovery
- Preserve referential integrity
- Support compliance requirements

### 2. Status Fields
**Decision**: Use NVARCHAR status fields instead of separate tables
**Reason**:
- Flexibility for status additions
- Easy querying and filtering
- Indexes on status for performance
- Examples: Order status, Product status, Auction status

### 3. JSON for Complex Data
**Decision**: Use NVARCHAR(MAX) with JSON storage for flexible data
**Reason**:
- Variant selections in CartItems and OrderItems
- Gateway responses in Payments
- Notification context data in Notifications
- Flexible without schema changes

### 4. Denormalization for Performance
**Decision**: Store calculated fields like AverageRating
**Reason**:
- Avoid expensive joins on every query
- Real-time availability requirements
- Update triggers refresh values
- Trade small storage for significant query performance

### 5. Partitioning Ready
**Decision**: Design supports range partitioning on large tables
**Reason**:
- CreatedAt fields on Orders, Auctions, Bids
- Easy to partition by date ranges
- Historical data can be archived
- Query performance on massive datasets

---

## Performance Considerations

### Query Optimization

1. **Product Search**
   ```sql
   -- Uses FULLTEXT index on Name, Description
   SELECT * FROM Products
   WHERE CONTAINS((Name, Description), @searchTerm)
   AND Status = 'ACTIVE'
   ```

2. **User Orders**
   ```sql
   -- Uses composite index on BuyerID, CreatedAt
   SELECT * FROM Orders
   WHERE BuyerID = @userId
   ORDER BY CreatedAt DESC
   ```

3. **Active Auctions**
   ```sql
   -- Uses composite index on Status, EndTime
   SELECT * FROM Auctions
   WHERE Status = 'ACTIVE'
   AND EndTime > GETUTCDATE()
   ```

### Caching Strategy

- **Product Cache** (Redis): Popular products, cached 1 hour
- **Category Cache** (Redis): All categories, cached 24 hours
- **User Session** (Redis): Login sessions, TTL = token expiry
- **Auction Data** (Redis): Active auctions, real-time updates

### Archival Strategy

- Orders/Bids older than 2 years moved to archive tables
- Improves query performance on active data
- Maintains compliance with data retention policies
- Archive tables searchable if needed

---

**Document Version**: 1.0  
**Status**: Approved  
**Last Updated**: 2026-06-12
