# UML Diagrams - Enterprise E-Commerce Marketplace Platform

**Document Version**: 1.0  
**Status**: In Development  
**Last Updated**: 2026-06-12

---

## Table of Contents
1. [Use Case Diagrams](#use-case-diagrams)
2. [Class Diagrams](#class-diagrams)
3. [Sequence Diagrams](#sequence-diagrams)
4. [Activity Diagrams](#activity-diagrams)
5. [State Diagrams](#state-diagrams)
6. [Component Diagrams](#component-diagrams)

---

## Use Case Diagrams

### Buyer Use Cases

```
                           ┌─────────────────────────────────────┐
                           │         Marketplace System          │
                           └─────────────────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
            ┌───────▼────────┐  ┌───────▼────────┐  ┌───────▼────────┐
            │ Browse Products│  │ Search Products│  │ Filter Products│
            └────────────────┘  └────────────────┘  └────────────────┘
                    │                   │                   │
                    └───────────────────┼───────────────────┘
                                        │
                            ┌───────────▼───────────┐
                            │   View Product Detail │
                            └───────────┬───────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
            ┌───────▼────────┐  ┌───────▼────────┐  ┌───────▼────────┐
            │ Add to Wishlist│  │ Add to Cart    │  │ Leave Review   │
            └────────────────┘  └────────────────┘  └────────────────┘
                                        │
                            ┌───────────▼───────────┐
                            │   Proceed to Checkout │
                            └───────────┬───────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
            ┌───────▼────────┐  ┌───────▼────────┐  ┌───────▼────────┐
            │Enter Shipping  │  │Select Shipping │  │Select Payment  │
            │   Address      │  │    Method      │  │    Method      │
            └────────────────┘  └────────────────┘  └────────────────┘
                                        │
                            ┌───────────▼───────────┐
                            │   Place Order         │
                            └───────────┬───────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
            ┌───────▼────────┐  ┌───────▼────────┐  ┌───────▼────────┐
            │Track Order     │  │Receive Order   │  │Manage Reviews  │
            └────────────────┘  └────────────────┘  └────────────────┘

                           ┌─────────────────────┐
                           │      Buyer User     │
                           └─────────────────────┘
                                  │
                                  │ performs
                                  │
                    ┌─────────────▼─────────────┐
                    │   Browse & Search         │
                    │   View Products           │
                    │   Manage Cart             │
                    │   Checkout                │
                    │   Track Orders            │
                    │   Leave Reviews           │
                    │   Place Auction Bids      │
                    └───────────────────────────┘
```

### Seller Use Cases

```
                        ┌──────────────────────────┐
                        │   Seller User            │
                        └──────────────────────────┘
                               │
                               │ performs
                               │
                ┌──────────────▼──────────────┐
                │  Store Management           │
                │  Product Management         │
                │  Auction Management         │
                │  Order Fulfillment          │
                │  View Analytics             │
                │  Respond to Reviews         │
                └──────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
    ┌────────┐    ┌─────────┐    ┌──────────┐
    │Create  │    │Manage   │    │Create    │
    │Product │    │Inventory│    │Auction   │
    └────────┘    └─────────┘    └──────────┘
        │               │               │
        ├─Upload Images─┤               │
        ├─Set Price─────┤               ├─Set Price
        ├─Add Tags──────┤               ├─Set Duration
        └─Set Category──┘               └─Monitor Bids
```

### Admin Use Cases

```
                        ┌──────────────────────────┐
                        │   Admin User             │
                        └──────────────────────────┘
                               │
                               │ performs
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
    ┌────────────┐        ┌─────────────┐       ┌─────────────┐
    │User        │        │Product      │       │Seller       │
    │Management  │        │Moderation   │       │Verification │
    └────────────┘        └─────────────┘       └─────────────┘
        │                      │                      │
        ├─Verify Users─────────┤                      ├─Approve KYC
        ├─Ban/Suspend──────────┤                      ├─Verify Bank
        └─Manage Roles─────────┤                      └─Reject Seller
                                │
                                ├─Review Content
                                ├─Approve/Reject
                                └─Flag Inappropriate

    ┌──────────────┐       ┌──────────────┐      ┌──────────────┐
    │Transaction   │       │Analytics &   │      │System Config │
    │Monitoring    │       │Reporting     │      │              │
    └──────────────┘       └──────────────┘      └──────────────┘
        │                      │                      │
        ├─Detect Fraud──┐      ├─Revenue Report      ├─Settings
        ├─Refund Issue──┤      ├─User Growth─────────┤─Maintenance
        └─View Disputes─┘      └─Category Stats      └─Backup
```

---

## Class Diagrams

### User Domain

```
┌─────────────────────────────────────┐
│            User                     │
├─────────────────────────────────────┤
│ - UserId: int {PK}                  │
│ - Email: string {unique}            │
│ - PasswordHash: string              │
│ - FirstName: string                 │
│ - LastName: string                  │
│ - Phone: string                     │
│ - CreatedAt: DateTime               │
│ - IsActive: bool                    │
├─────────────────────────────────────┤
│ + Register(email, password)         │
│ + Login(email, password)            │
│ + ResetPassword(token, newPass)     │
│ + GetProfile(): UserDto             │
│ + UpdateProfile(data)               │
└──────────────┬──────────────────────┘
               │ has
       ┌───────┴────────┐
       │                │
       ▼                ▼
┌─────────────┐  ┌──────────────┐
│Role         │  │RefreshToken  │
├─────────────┤  ├──────────────┤
│- RoleId     │  │- TokenId     │
│- Name       │  │- Token       │
├─────────────┤  │- ExpiresAt   │
│BUYER        │  │- RevokedAt   │
│SELLER       │  └──────────────┘
│ADMIN        │
└─────────────┘

┌──────────────────────────────────────┐
│         Seller                       │
├──────────────────────────────────────┤
│ - SellerID: int {PK}                 │
│ - UserID: int {FK}                   │
│ - BusinessName: string               │
│ - VerificationStatus: enum           │
│ - KYCDocPath: string                 │
│ - BankAccount: string                │
│ - CommissionRate: decimal            │
├──────────────────────────────────────┤
│ + CreateProduct(data): Product       │
│ + GetAnalytics(period): Analytics    │
│ + UpdateStore(data)                  │
│ + GetOrders(filter): Order[]         │
└──────────────┬───────────────────────┘
               │ has
               ▼
        ┌──────────────┐
        │   Store      │
        ├──────────────┤
        │- StoreID     │
        │- StoreName   │
        │- Description │
        │- Logo        │
        │- Banner      │
        └──────────────┘
```

### Product Domain

```
┌──────────────────────────────────────┐
│         Product                      │
├──────────────────────────────────────┤
│ - ProductID: int {PK}                │
│ - SellerID: int {FK}                 │
│ - CategoryID: int {FK}               │
│ - Name: string                       │
│ - Description: string                │
│ - Price: decimal                     │
│ - Status: enum                       │
│ - CreatedAt: DateTime                │
├──────────────────────────────────────┤
│ + GetDetails(): ProductDto           │
│ + UpdateInventory(qty)               │
│ + GetReviews(): Review[]             │
│ + GetAvgRating(): decimal            │
└──────────┬──────────────┬────────────┘
           │ contains     │ has
           ▼              ▼
   ┌────────────────┐  ┌──────────────┐
   │ProductImage    │  │Inventory     │
   ├────────────────┤  ├──────────────┤
   │- ImageId       │  │- InventoryId │
   │- ImagePath     │  │- OnHand      │
   │- IsPrimary     │  │- Reserved    │
   └────────────────┘  │- Available   │
                       └──────────────┘

   ┌──────────────────┐
   │ProductVariant    │
   ├──────────────────┤
   │- VariantId       │
   │- Name            │
   │- Value           │
   │- PriceModifier   │
   └──────────────────┘
```

### Order Domain

```
┌──────────────────────────────────────┐
│         Order                        │
├──────────────────────────────────────┤
│ - OrderID: int {PK}                  │
│ - OrderNumber: string {unique}       │
│ - BuyerID: int {FK}                  │
│ - SellerID: int {FK}                 │
│ - Status: enum                       │
│ - TotalAmount: decimal               │
│ - CreatedAt: DateTime                │
│ - ShippedAt: DateTime                │
│ - DeliveredAt: DateTime              │
├──────────────────────────────────────┤
│ + UpdateStatus(newStatus)            │
│ + GetItems(): OrderItem[]            │
│ + ProcessRefund(reason)              │
│ + GetTrackingInfo(): Tracking        │
└──────────┬──────────────┬────────────┘
           │ contains     │ has
           ▼              ▼
   ┌────────────────┐  ┌──────────────┐
   │OrderItem       │  │Payment       │
   ├────────────────┤  ├──────────────┤
   │- ItemId        │  │- PaymentId   │
   │- ProductId     │  │- Method      │
   │- Qty           │  │- Amount      │
   │- UnitPrice     │  │- Status      │
   │- SubtotalPrice │  │- TransactionId
   └────────────────┘  └──────────────┘
```

### Auction Domain

```
┌──────────────────────────────────────┐
│         Auction                      │
├──────────────────────────────────────┤
│ - AuctionID: int {PK}                │
│ - ProductID: int {FK}                │
│ - SellerID: int {FK}                 │
│ - StartPrice: decimal                │
│ - ReservePrice: decimal              │
│ - BidIncrement: decimal              │
│ - Status: enum                       │
│ - StartTime: DateTime                │
│ - EndTime: DateTime                  │
│ - WinnerID: int {FK}                 │
│ - WinningBidAmount: decimal          │
├──────────────────────────────────────┤
│ + PlaceBid(amount, bidderId)         │
│ + GetCurrentBid(): decimal           │
│ + ConcludeAuction()                  │
│ + GetBidHistory(): Bid[]             │
└──────────┬────────────────────────────┘
           │ has
           ▼
   ┌────────────────┐
   │Bid             │
   ├────────────────┤
   │- BidId         │
   │- BidderId      │
   │- BidAmount     │
   │- BidTime       │
   │- IsWinningBid  │
   └────────────────┘
```

---

## Sequence Diagrams

### Product Purchase Sequence

```
Buyer    Frontend    API        Service    Repository   Database
  │         │          │           │            │           │
  │─View Product──────│           │            │           │
  │         │──GET /products/1────│            │           │
  │         │          │──Select────────────────│           │
  │         │          │          │───Query────│──────────│
  │         │          │          │            │           │
  │─Add to Cart────────│           │            │           │
  │         │──POST /cart────────│            │           │
  │         │          │──Add Item────────────│           │
  │         │          │          │────INSERT─│──────────│
  │         │          │          │◄──Confirm─│──────────│
  │         │          │◄─Item Added──────────│           │
  │         │◄──OK──────           │            │           │
  │         │          │           │            │           │
  │─Checkout──────────│           │            │           │
  │         │──POST /orders────────│            │           │
  │         │          │──Create──────────────│           │
  │         │          │          │──Validate─│──────────│
  │         │          │          │───Reserve─Stock───────│
  │         │          │          │            │           │
  │─Payment──────────│           │            │           │
  │         │──POST /payments────│            │           │
  │         │          │──Process──────────────│           │
  │         │          │          │──Payment──Success──────│
  │         │          │          │───Update─Order Status──│
  │         │          │◄──Success─────────────           │
  │         │◄──Order Created──────           │           │
  │         │          │           │            │           │
```

### Auction Bidding Sequence (Real-Time)

```
Bidder   Frontend    API        SignalR    Service    Database
  │         │          │          │           │           │
  │─Place Bid────────│           │           │           │
  │         │──POST /bids────────│           │           │
  │         │          │──Validate───────────│           │
  │         │          │          │──Broadcast to viewers─│
  │         │          │          │──Update─Highest Bid───│
  │         │          │◄─Bid Accepted       │           │
  │         │◄──Success──         │           │           │
  │         │          │          │           │           │
  │         │ Other Connected Clients:       │           │
  │         │          │◄─Bid Update────────│           │
  │         │[Update UI Real-Time]          │           │
  │         │          │          │           │           │
```

---

## Activity Diagrams

### Order Processing Activity

```
        (Start)
          │
          ▼
      ┌─────────────┐
      │ Receive     │
      │ Order       │
      └──────┬──────┘
             │
             ▼
      ┌─────────────────┐
      │ Validate Order  │
      │ Items & Stock   │
      └────┬────────┬───┘
           │        │
       Valid?       │
        / \         │
       /   \        │
      Y     N───────┼──►[Reject Order]──►(End)
      │             │
      ▼             │
  ┌────────────┐    │
  │Reserve     │    │
  │Stock       │    │
  └─────┬──────┘    │
        │           │
        ▼           │
  ┌────────────┐    │
  │Process     │    │
  │Payment     │    │
  └─────┬──────┘    │
        │           │
   Payment OK?       │
     / \            │
    /   \           │
   Y     N──────────┼──►[Refund Stock]──►(End)
   │                │
   ▼                │
┌───────────────┐   │
│Create Shipment│   │
└────┬──────────┘   │
     │              │
     ▼              │
┌───────────────┐   │
│Send           │   │
│Notification   │   │
└────┬──────────┘   │
     │              │
     ▼              │
┌───────────────┐   │
│Update Order   │   │
│Status: SHIPPED│   │
└────┬──────────┘   │
     │              │
     └──────────────┘
          │
          ▼
      (End)
```

---

## State Diagrams

### Order State Diagram

```
    ┌─────────────┐
    │   PENDING   │◄──────(Order Created)
    └──────┬──────┘
           │
      [Payment Accepted]
           │
           ▼
    ┌─────────────┐
    │ CONFIRMED   │
    └──────┬──────┘
           │
      [Start Processing]
           │
           ▼
    ┌─────────────┐
    │ PROCESSING  │
    └──────┬──────┘
           │
      [Shipment Created]
           │
           ▼
    ┌─────────────┐
    │  SHIPPED    │
    └──────┬──────┘
           │
      [Delivery Confirmed]
           │
           ▼
    ┌─────────────┐
    │ DELIVERED   │
    └──────┬──────┘
           │
      [30 days passed]
           │
           ▼
    ┌─────────────┐
    │ COMPLETED   │
    └─────────────┘

    ╔═════════════════╗ [Anytime: Refund Request]
    ║   CANCELLED     ║◄───────────────────────────
    ╚═════════════════╝
```

### Auction State Diagram

```
    ┌─────────────┐
    │   DRAFT     │◄──────(Auction Created)
    └──────┬──────┘
           │
      [Publish]
           │
           ▼
    ┌─────────────┐
    │ SCHEDULED   │
    └──────┬──────┘
           │
      [Start Time Reached]
           │
           ▼
    ┌─────────────┐
    │   ACTIVE    │◄─────(Accept Bids)
    └──────┬──────┘
           │
      [End Time Reached]
           │
           ▼
    ┌─────────────┐
    │   ENDED     │
    └──────┬──────┘
           │
      [Select Winner]
           │
           ▼
    ┌─────────────┐
    │ COMPLETED   │
    └─────────────┘

    ╔═════════════════╗ [Anytime]
    ║  CANCELLED      ║◄──────────
    ╚═════════════════╝
```

---

## Component Diagrams

### System Components

```
┌──────────────────────────────────────────────────────────────┐
│                     Marketplace System                       │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Presentation Layer                      │   │
│  │  ┌────────────────┐  ┌────────────────┐             │   │
│  │  │ React Frontend │  │ Admin Dashboard│             │   │
│  │  └────────────────┘  └────────────────┘             │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│                    HTTP/HTTPS │ WebSocket                   │
│                           │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              API Gateway Layer                       │   │
│  │  ┌────────────────────────────────────────────────┐  │   │
│  │  │ ASP.NET Core API (Controllers, Middleware)    │  │   │
│  │  │ • Authentication • Authorization • Validation  │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Application Layer                      │   │
│  │  ┌────────────────┐  ┌────────────────┐             │   │
│  │  │Service Layer   │  │SignalR Hub     │             │   │
│  │  │• Business Logic│  │• Real-time Bid │             │   │
│  │  │• Transactions  │  │• Notifications │             │   │
│  │  └────────────────┘  └────────────────┘             │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Data Access Layer                      │   │
│  │  ┌────────────────────────────────────────────────┐  │   │
│  │  │ EF Core + Repositories + Unit of Work         │  │   │
│  │  └────────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Data Layer                            │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐   │   │
│  │  │SQL Server  │  │Redis Cache │  │AWS S3      │   │   │
│  │  │(Relational)│  │(Session)   │  │(Images)    │   │   │
│  │  └────────────┘  └────────────┘  └────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              External Services                      │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐   │   │
│  │  │Stripe      │  │PayPal      │  │SendGrid    │   │   │
│  │  │Chapa       │  │Telebirr    │  │CloudFront  │   │   │
│  │  └────────────┘  └────────────┘  └────────────┘   │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

**Document Version**: 1.0  
**Status**: In Development  
**Last Updated**: 2026-06-12
