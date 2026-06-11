# Customer Data Model

## Overview

The `customers` collection stores the account data of registered customers.

Foodpilot is designed for small restaurants, takeaways, and food trucks. Each deployment represents a restaurant instance with its own customer database.

The customer document contains:

- identity and contact details
- authentication data
- saved delivery addresses
- basic order statistics
- account status
- marketing consent
- customer self-deletion through soft delete

Orders, payments, restaurant details, and location details are stored separately.

Foodpilot supports guest checkout. Guest orders do not automatically create customer accounts.

---

## Architectural Context

The default deployment model is:

- one restaurant
- one location
- one customer database

Customers interact with a single restaurant instance.

Restaurant locations are managed separately in the `locations` collection. This keeps the customer model simple while preserving the option to support multiple locations later.

---

## Collection

```js
customers
```

Each registered customer is stored as a separate document.

---

## Schema

```js
{
  _id: ObjectId,

  firstName: String,
  lastName: String,

  email: String,
  emailNormalized: String,
  emailVerified: Boolean,

  phone: String | null,

  passwordHash: String | null,

  addresses: [
    {
      _id: ObjectId,

      label: String,

      street: String,
      houseNumber: String,

      zip: String,
      city: String,

      floor: String | null,
      note: String | null,

      isDefault: Boolean,

      createdAt: Date,
      updatedAt: Date
    }
  ],

  orderCount: Number,
  totalSpentCents: Number,
  lastOrderAt: Date | null,

  accountStatus: "active" | "blocked" | "deleted",
  deletedAt: Date | null,

  marketingConsent: Boolean,
  marketingConsentUpdatedAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

---

## Field Reference

### Identity

| Field | Type | Required | Description |
|---|---|---:|---|
| `_id` | `ObjectId` | Yes | Unique customer identifier. |
| `firstName` | `String` | Yes | Customer's first name. |
| `lastName` | `String` | Yes | Customer's last name. |

---

### Contact Details

| Field | Type | Required | Description |
|---|---|---:|---|
| `email` | `String` | Yes | Customer-facing email address. |
| `emailNormalized` | `String` | Yes | Lowercase and trimmed email address used for uniqueness checks. |
| `emailVerified` | `Boolean` | Yes | Indicates whether the email address has been verified. |
| `phone` | `String \| null` | No | Optional phone number stored in the customer account. |

The phone number remains optional in the customer account.

During checkout:

- a phone number is required for delivery orders
- a phone number is optional for pickup orders

Email verification is not required before placing the first order.

Foodpilot supports guest checkout, and mandatory verification before checkout would add unnecessary friction. Email verification remains available for account-related features.

Example:

```js
{
  email: "Max.Mueller@example.de",
  emailNormalized: "max.mueller@example.de"
}
```

---

### Authentication

| Field | Type | Required | Description |
|---|---|---:|---|
| `passwordHash` | `String \| null` | Conditional | Secure password hash. Required for active customer accounts and removed when an account is deleted. |

Passwords must always be hashed before they are written to the database.

Plain-text passwords must never be stored:

```js
{
  password: "plain-text-password"
}
```

### Password Reset Tokens

Password reset functionality uses a separate collection:

```js
passwordResetTokens
```

```js
{
  _id: ObjectId,

  customerId: ObjectId,

  tokenHash: String,

  expiresAt: Date,
  usedAt: Date | null,

  createdAt: Date
}
```

Plain-text reset tokens must never be stored.

Reset tokens are short-lived and become invalid after their expiration date or after they have been used.

Expired tokens are removed automatically through a TTL index.

---

## Delivery Addresses

A customer can store multiple delivery addresses.

```js
addresses: [
  {
    _id: ObjectId,

    label: "Home",

    street: "Musterstraße",
    houseNumber: "12",

    zip: "04103",
    city: "Leipzig",

    floor: "2nd floor",
    note: "Please ring the bell",

    isDefault: true,

    createdAt: Date,
    updatedAt: Date
  }
]
```

### Address Fields

| Field | Type | Required | Description |
|---|---|---:|---|
| `_id` | `ObjectId` | Yes | Unique identifier of the saved address. |
| `label` | `String` | Yes | Customer-defined label such as `Home` or `Work`. |
| `street` | `String` | Yes | Street name. |
| `houseNumber` | `String` | Yes | House number including optional suffixes. |
| `zip` | `String` | Yes | Postal code. |
| `city` | `String` | Yes | City name. |
| `floor` | `String \| null` | No | Optional floor information. |
| `note` | `String \| null` | No | Optional delivery instruction. |
| `isDefault` | `Boolean` | Yes | Marks the preferred delivery address. |
| `createdAt` | `Date` | Yes | Creation timestamp. |
| `updatedAt` | `Date` | Yes | Last modification timestamp. |

### Default Address Rules

A customer can have no more than one default address.

- The first saved address becomes the default address automatically.
- When a new address is marked as default, the previous default address is updated to `false`.
- When the default address is deleted, another available address becomes the default address.
- If no saved address exists, the `addresses` array remains empty.

---

## Customer Statistics

```js
{
  orderCount: 12,
  totalSpentCents: 35640,
  lastOrderAt: Date
}
```

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `orderCount` | `Number` | Yes | `0` | Number of successfully completed orders. |
| `totalSpentCents` | `Number` | Yes | `0` | Total value of successfully completed orders in cents. |
| `lastOrderAt` | `Date \| null` | No | `null` | Timestamp of the most recent completed order. |

The complete order history remains in the `orders` collection.

The initial restaurant admin interface exposes only:

```js
orderCount
totalSpentCents
lastOrderAt
```

No additional customer profiling is included in the MVP.

Customer statistics are updated only after an order reaches the relevant final state:

```js
orderStatus: "completed"
```

Cancelled or failed orders do not increase `orderCount` or `totalSpentCents`.

All monetary values are stored in the smallest currency unit.

```js
{
  totalSpentCents: 35640
}
```

represents:

```text
356.40 EUR
```

---

## Account Status

```js
{
  accountStatus: "active",
  deletedAt: null
}
```

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `accountStatus` | `"active" \| "blocked" \| "deleted"` | Yes | `"active"` | Current state of the customer account. |
| `deletedAt` | `Date \| null` | No | `null` | Timestamp of a soft deletion. |

### Status Values

| Value | Description |
|---|---|
| `active` | Normal customer account. |
| `blocked` | Login or ordering is restricted. |
| `deleted` | Soft-deleted account. Personal account data is anonymised where legally possible. |

Customers can delete their own accounts.

Deletion is implemented as a soft delete. Legally relevant order records remain available according to applicable retention requirements.

When an account is deleted, personal account data is anonymised or removed where legally possible.

The customer document is updated as follows:

```js
{
  firstName: "Deleted",
  lastName: "Customer",

  email: "deleted-<customerId>@anonymized.invalid",
  emailNormalized: "deleted-<customerId>@anonymized.invalid",
  emailVerified: false,

  phone: null,
  passwordHash: null,

  addresses: [],

  accountStatus: "deleted",
  deletedAt: Date,

  marketingConsent: false,
  marketingConsentUpdatedAt: Date
}
```

The anonymised email placeholder contains the customer ID to ensure that it remains unique.

This releases the original email address so that it can be used for a new account later.

The following authentication data is invalidated when an account is deleted:

- password hash
- active sessions
- unused password reset tokens

Historical orders remain available according to applicable retention requirements.

Order snapshots are stored separately and remain unchanged when a customer account is edited or deleted.

---

## Marketing Consent

```js
{
  marketingConsent: false,
  marketingConsentUpdatedAt: null
}
```

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `marketingConsent` | `Boolean` | Yes | `false` | Indicates whether the customer has opted in to promotional communication. |
| `marketingConsentUpdatedAt` | `Date \| null` | No | `null` | Timestamp of the most recent consent change. |

Transactional messages such as order confirmations are handled separately from promotional communication.

Marketing consent remains part of the schema in the MVP. Active marketing features are outside the MVP scope.

---

## Guest Checkout

Foodpilot supports guest checkout.

A customer account is not required to place an order.

Guest orders store a checkout snapshot instead of creating a customer document automatically:

```js
{
  customerId: null,

  customerSnapshot: {
    firstName: String,
    lastName: String,
    email: String,
    phone: String | null
  },

  deliveryAddressSnapshot: {
    street: String,
    houseNumber: String,
    zip: String,
    city: String,
    floor: String | null,
    note: String | null
  }
}
```

After placing an order, guests can create an account.

---

## Relations to Other Collections

### Orders

Orders reference the customer account when the order was placed by a registered customer:

```js
{
  customerId: ObjectId
}
```

Every order stores a snapshot of the customer and delivery details used during checkout:

```js
{
  customerSnapshot: {
    firstName: String,
    lastName: String,
    email: String,
    phone: String | null
  },

  deliveryAddressSnapshot: {
    street: String,
    houseNumber: String,
    zip: String,
    city: String,
    floor: String | null,
    note: String | null
  }
}
```

Historical orders remain unchanged when a customer edits or deletes a saved address later.

---

### Locations

The customer document does not store location data.

Coordinates, opening hours, delivery zones, and location-specific settings belong in the `locations` collection.

---

## Payment Data

We do not store raw payment details.

Foodpilot stores only the minimum information required to associate an order with a payment transaction.

Example:

```js
{
  paymentProvider: "stripe",
  paymentReference: String,
  paymentStatus: "pending" | "paid" | "failed" | "refunded"
}
```

Payment-related fields belong in the order or payment collection, not in the customer document.

Sensitive information such as card numbers, security codes, or payment-account passwords is handled exclusively by the payment provider.

---

## Validation Rules

The API validates customer data before writing it to the database.

### Identity

- `firstName` is required
- `lastName` is required
- names are trimmed
- maximum lengths are defined

### Email

- email is required
- email is trimmed and normalized
- email format is validated
- `emailNormalized` is unique

### Phone Number

The phone number remains optional in the customer account.

During checkout:

- a phone number is required for delivery orders
- a phone number is optional for pickup orders
- phone numbers are trimmed
- international formatting is supported

### Address

- street is required
- house number is required
- postal code is required
- city is required
- delivery notes have a maximum length
- only one address can be the default address

### Statistics

- `orderCount` cannot be negative
- `totalSpentCents` cannot be negative
- statistics are never accepted from untrusted frontend input

---

## Indexes

### Customer Indexes

```js
db.customers.createIndex(
  { emailNormalized: 1 },
  { unique: true }
)
```

### Password Reset Token Indexes

```js
db.passwordResetTokens.createIndex(
  { tokenHash: 1 },
  { unique: true }
)

db.passwordResetTokens.createIndex(
  { expiresAt: 1 },
  { expireAfterSeconds: 0 }
)

db.passwordResetTokens.createIndex(
  { customerId: 1 }
)
```

The TTL index removes expired password reset tokens automatically.

---

## Security Considerations

Customer data is sensitive and must be protected accordingly.

Minimum requirements:

- passwords are never stored as plain text
- passwords are hashed using a modern password hashing algorithm
- customer data is only returned to authorised users
- frontend input is validated on the backend
- login and password-reset endpoints are rate-limited
- `passwordHash` is never returned to the frontend
- logs do not contain unnecessary personal data

Example API response:

```js
{
  _id: ObjectId,

  firstName: "Max",
  lastName: "Müller",

  email: "max@example.de",
  emailVerified: true,

  phone: "017612345678",

  addresses: [...],

  accountStatus: "active"
}
```

The following field is never included in API responses:

```js
{
  passwordHash: "..."
}
```

---

## Privacy and Data Minimisation

Foodpilot stores only data required for:

- account management
- ordering
- delivery
- customer support
- legal obligations

Optional data remains optional.

Delivery notes may contain personal information entered by the customer. They are shared only where required for order fulfilment.

---

## MVP Scope

The schema above defines the customer document for the first functional version.

The first version includes:

- registered customer accounts
- guest checkout
- multiple saved delivery addresses
- one default address
- optional email verification for account-related features
- basic customer statistics
- account blocking
- soft deletion
- marketing consent
- customer self-deletion through soft delete

---

## Later Extensions

The following features are outside the MVP scope:

- saved order templates
- favourite products
- notification preferences
- preferred language
- loyalty points
- restaurant-specific customer notes
- customer data export
- address validation
- geocoding
- automated delivery-zone validation
- admin audit logs

---

## Design Decisions Summary

| Decision | Reason |
|---|---|
| Customer accounts belong to one restaurant instance | Each deployment represents one restaurant instance |
| Restaurant locations remain separate from customers | Location management belongs in the restaurant domain |
| Names are stored separately | Supports account management and order documents |
| Normalized emails are stored separately | Reliable uniqueness checks |
| Addresses are stored as subdocuments | Simple handling of multiple delivery addresses |
| Every saved address has an `_id` | Clean editing and deletion |
| Orders contain customer and address snapshots | Historical orders remain unchanged |
| Complete orders remain outside customers | Clear separation of responsibilities |
| Monetary values are stored in cents | Avoids floating-point rounding errors |
| Raw payment details are not stored | Payment providers handle sensitive data |
| Guest checkout is supported | Reduces friction during checkout |
| Deleted accounts are soft-deleted | Preserves legally relevant records |
| Personal account data is anonymised after deletion | Reduces retained personal data where legally possible |
| Phone numbers are required for delivery orders | Restaurants and drivers need a way to resolve delivery issues |
| Email verification is not required before the first order | Guest checkout remains frictionless |
| Password reset tokens use a separate collection | Reset tokens remain short-lived and independently manageable |
| Phone numbers remain optional in customer accounts | The requirement depends on the checkout type |
| Deleted email addresses are replaced with unique placeholder values | Original email addresses can be reused for new accounts |
| Authentication data is invalidated after account deletion | Deleted accounts must no longer allow login or password recovery |
| Expired password reset tokens are removed automatically | Short-lived tokens should not remain in the database |
