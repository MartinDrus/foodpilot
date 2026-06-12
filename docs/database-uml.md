# Foodpilot Collections UML

This diagram reflects the current documentation state. Models that are not fully specified yet are marked as planned.

```mermaid
erDiagram
    RESTAURANTS ||--o{ LOCATIONS : owns
    RESTAURANTS ||--o{ RESTAURANT_USERS : has
    RESTAURANTS ||--o{ MENU_CATEGORIES : owns
    RESTAURANTS ||--o{ MENU_ITEMS : owns
    RESTAURANTS ||--o{ ORDERS : receives
    RESTAURANTS ||--o{ PAYMENTS : records
    RESTAURANTS ||--o{ INVENTORY_RESERVATIONS : owns
    RESTAURANTS ||--o{ ORDER_COUNTERS : owns
    RESTAURANTS ||--o{ REVIEWS : receives
    RESTAURANTS ||--o{ DRIVERS : employs

    MENU_CATEGORIES ||--o{ MENU_ITEMS : contains

    CUSTOMERS ||--o{ PASSWORD_RESET_TOKENS : requests
    CUSTOMERS o|--o{ ORDERS : places
    CUSTOMERS o|--o{ REVIEWS : writes

    LOCATIONS ||--o{ ORDERS : serves

    ORDERS ||--o{ PAYMENTS : has
    ORDERS ||--o| INVENTORY_RESERVATIONS : reserves
    ORDERS ||--o| REVIEWS : receives

    MENU_ITEMS ||--o{ INVENTORY_RESERVATIONS : referenced_by
    MENU_ITEMS ||--o{ REVIEWS : rated_in

    DRIVERS o|--o{ ORDERS : assigned_to

    RESTAURANTS {
        ObjectId _id
        string name
        string slug
        string locationMode
        ObjectId activeLocationId
        object contact
        object legalInformation
        object branding
        string currency
        object orderConfiguration
        object menuConfiguration
        boolean isActive
    }

    LOCATIONS {
        ObjectId _id
        ObjectId restaurantId
        string name
        string displayAddress
        object address
        object geoLocation
        array openingHours
        array openingHourExceptions
        string timezone
        string orderAcceptanceStatus
        object preorderSettings
        object deliverySettings
        boolean isEnabled
    }

    RESTAURANT_USERS {
        ObjectId _id
        ObjectId restaurantId
        string name
        string email
        string emailNormalized
        string passwordHash
        string role
        boolean isActive
        date lastLoginAt
    }

    CUSTOMERS {
        ObjectId _id
        string emailNormalized
        array addresses
        number orderCount
        number totalSpentCents
        date lastOrderAt
        string accountStatus
    }

    PASSWORD_RESET_TOKENS {
        ObjectId _id
        ObjectId customerId
        string tokenHash
        date expiresAt
        date usedAt
    }

    MENU_CATEGORIES {
        ObjectId _id
        ObjectId restaurantId
        string name
        string slug
        number sortOrder
        boolean isActive
    }

    MENU_ITEMS {
        ObjectId _id
        ObjectId restaurantId
        ObjectId categoryId
        string name
        number priceInCents
        array labels
        string availabilityStatus
        number stockQuantity
        array options
    }

    ORDERS {
        ObjectId _id
        ObjectId restaurantId
        ObjectId locationId
        ObjectId customerId
        string orderNumber
        object locationSnapshot
        object customerSnapshot
        object deliveryAddressSnapshot
        array items
        string orderStatus
        string paymentStatus
        array statusHistory
    }

    PAYMENTS {
        ObjectId _id
        ObjectId restaurantId
        ObjectId orderId
        string type
        string method
        number amountInCents
        string status
        string providerReference
    }

    INVENTORY_RESERVATIONS {
        ObjectId _id
        ObjectId restaurantId
        ObjectId orderId
        string status
        array items
        date expiresAt
    }

    ORDER_COUNTERS {
        ObjectId _id
        ObjectId restaurantId
        number year
        number nextSequence
    }

    REVIEWS {
        ObjectId _id
        ObjectId restaurantId
        ObjectId orderId
        ObjectId customerId
        number overallRating
        number deliveryRating
        array itemRatings
        boolean isPublished
    }

    DRIVERS {
        ObjectId _id
        ObjectId restaurantId
        string status
        ObjectId currentOrderId
    }
```

## Notes

- `restaurants`, `locations`, and `restaurantUsers` are documented in `docs/database/restaurant.md`.
- `orders.locationSnapshot`, `orders.customerSnapshot`, and `orders.items[]` preserve immutable historical data.
- `customers` currently follows the single-restaurant deployment model and therefore does not yet require a `restaurantId`.
- `reviews` are included in the MVP, but their dedicated documentation in `docs/database/reviews.md` is still pending.
- `drivers`, driver assignment, and delivery tours remain provisional until `docs/database/driver.md` is finalized.
