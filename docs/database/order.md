# Orders

## Overview

The `orders` collection stores the immutable commercial snapshot and current fulfilment state of every order.

Foodpilot supports:

- delivery and pickup
- registered customers and guest checkout
- PayPal and cash payment
- restaurant acceptance before preparation
- optional product-level inventory tracking
- reviews linked to completed orders

Historical orders remain unchanged when customer profiles, locations, menu items, prices, tax rates or options are updated later.

The MVP uses:

- one restaurant
- one active location
- one menu
- one customer database
- one order flow

The active location can represent a fixed business address or a mobile operating point, such as the current position of a food truck.

---

## Architectural Context

The order document is the central record for:

- immutable customer, location, address and product snapshots
- current fulfilment state
- payment summary
- totals and tax breakdown
- operational timestamps
- status history

Related data is stored separately:

```text
payments
inventoryReservations
orderCounters
deliveryTours
reviews
```

Restaurant-level configuration includes:

```js
{
  cashOnPickupPolicy:
    "disabled"
    | "returning_customers_only"
    | "all_customers",

  inventoryReservationMinutes: Number,
  orderAcceptanceTimeoutMinutes: Number,

  enabledPaymentMethods: [
    "paypal",
    "cash"
  ]
}
```

The default pickup policy is:

```js
cashOnPickupPolicy: "returning_customers_only"
```

---

## Collections

Primary collection:

```text
orders
```

Related MVP collections:

```text
payments
inventoryReservations
orderCounters
deliveryTours
reviews
```

---

## Order Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  locationId: ObjectId,

  locationSnapshot: {
    name: String,
    displayAddress: String,

    address: {
      street: String | null,
      houseNumber: String | null,
      zip: String | null,
      city: String | null,
      countryCode: String | null
    },

    geoLocation: {
      type: "Point",
      coordinates: [Number, Number]
    }
  },


  orderNumber: String | null,

  customerId: ObjectId | null,

  customerSnapshot: {
    firstName: String,
    lastName: String,
    email: String,
    phone: String | null
  },

  deliveryType:
    "delivery"
    | "pickup",

  deliveryAddressSnapshot: {
    street: String,
    houseNumber: String,
    zip: String,
    city: String,
    floor: String | null,
    note: String | null,

    geoLocation: {
      type: "Point",
      coordinates: [Number, Number]
    }
  } | null,

  deliveryTourId: ObjectId | null,

  deliveryAssignmentHistory: [
    {
      deliveryTourId: ObjectId,
      assignedAt: Date,
      unassignedAt: Date | null,

      reason:
        "initial_assignment"
        | "manual_reassignment"
        | "delivery_retry"
        | "tour_cancelled"
    }
  ],

  items: [
    {
      _id: ObjectId,
      menuItemId: ObjectId,

      internalNumber: String | null,
      name: String,

      quantity: Number,

      unitPriceInCents: Number,
      taxRate: Number,

      allergenCodes: [String],
      additiveCodes: [String],
      labels: [String],

      selectedOptions: [
        {
          optionGroupId: ObjectId,
          optionGroupName: String,

          choiceId: ObjectId,
          choiceName: String,

          priceModifierInCents: Number,

          allergenCodes: [String],
          additiveCodes: [String]
        }
      ],

      lineTotalInCents: Number,
      note: String | null
    }
  ],

  customerNote: String | null,

  currency: "EUR",

  subtotalInCents: Number,
  deliveryFeeInCents: Number,
  discountTotalInCents: Number,
  tipInCents: Number,
  totalInCents: Number,

  taxSummary: [
    {
      taxRate: Number,
      netInCents: Number,
      taxInCents: Number,
      grossInCents: Number
    }
  ],

  paymentMethod:
    "paypal"
    | "cash",

  paymentStatus:
    "pending"
    | "paid"
    | "failed"
    | "refund_pending"
    | "refunded"
    | "partially_refunded"
    | "cancelled",

  paidAt: Date | null,
  refundedAt: Date | null,
  refundedTotalInCents: Number,

  orderStatus:
    "awaiting_payment"
    | "new"
    | "accepted"
    | "preparing"
    | "ready"
    | "out_for_delivery"
    | "delivery_failed"
    | "delivered"
    | "picked_up"
    | "rejected"
    | "cancelled"
    | "expired",

  statusHistory: [
    {
      previousStatus: String | null,
      status: String,

      changedAt: Date,

      changedByType:
        "customer"
        | "restaurant"
        | "driver"
        | "system",

      changedById: ObjectId | null,
      note: String | null
    }
  ],

  termination: {
    type:
      "rejected"
      | "cancelled"
      | "expired",

    reasonCode: String,
    note: String | null,

    changedByType:
      "customer"
      | "restaurant"
      | "system",

    changedById: ObjectId | null,
    occurredAt: Date
  } | null,

  fulfilmentEstimate: {
    estimatedPreparationMinutes: Number,
    estimatedReadyAt: Date,

    estimatedDeliveryMinutes: Number | null,
    estimatedDeliveryAt: Date | null,

    updatedAt: Date,

    updatedByType:
      "system"
      | "restaurant"
  } | null,

  fulfilmentEstimateHistory: [
    {
      estimatedPreparationMinutes: Number,
      estimatedReadyAt: Date,

      estimatedDeliveryMinutes: Number | null,
      estimatedDeliveryAt: Date | null,

      changedAt: Date,

      changedByType:
        "system"
        | "restaurant",

      changedById: ObjectId | null,
      note: String | null
    }
  ],

  paymentReservationExpiresAt: Date | null,
  acceptanceExpiresAt: Date | null,

  unpaidPickupGuardKey: String | null,

  acceptedAt: Date | null,
  preparingAt: Date | null,
  readyAt: Date | null,
  outForDeliveryAt: Date | null,
  completedAt: Date | null,
  terminatedAt: Date | null,

  customerStatisticsAppliedAt: Date | null,

  isArchived: Boolean,
  archivedAt: Date | null,
  archivedById: ObjectId | null,

  createdAt: Date,
  updatedAt: Date
}
```

---

## Location Snapshot

Every order stores:

```js
locationId: ObjectId
```

and an immutable:

```js
locationSnapshot
```

The snapshot preserves the active location used when the order was created.

This is required because the active location can change later, especially for mobile food trucks.

Updating the active restaurant location never modifies historical orders.

---

## Customer and Address Snapshots

Every order stores an immutable:

```js
customerSnapshot
```

For registered customers:

```js
customerId: ObjectId
```

For guest checkout:

```js
customerId: null
```

For delivery orders:

- `deliveryAddressSnapshot` is required
- `customerSnapshot.phone` is required

For pickup orders:

- `deliveryAddressSnapshot` is `null`
- a phone number is normally optional
- a phone number is required for guest cash pickup orders

Customer profile updates and account deletion never modify historical order snapshots.

---

## Order Item Snapshots

Each order item stores the purchased product and selected options as immutable snapshots.

The server recalculates all values from current menu data during checkout.

Client-provided prices, totals, tax rates, allergen codes and additive codes are never trusted.

```text
lineTotalInCents =
  (
    unitPriceInCents
    + selectedOptionModifiersInCents
  )
  * quantity
```

Product labels are stored as snapshots because they may change later.

---

## Delivery and Pickup

Pickup order:

```js
{
  deliveryType: "pickup",
  deliveryAddressSnapshot: null
}
```

Delivery order:

```js
{
  deliveryType: "delivery",
  deliveryAddressSnapshot: Object
}
```

Only delivery orders can reference a delivery tour.

Pickup orders always use:

```js
deliveryTourId: null
```
The assigned driver is resolved through:

```text
order
→ deliveryTour
→ driver
```

`fulfilmentEstimate` stores the current customer-facing estimate.

For pickup:

```js
{
  estimatedReadyAt: Date,
  estimatedDeliveryMinutes: null,
  estimatedDeliveryAt: null
}
```

For delivery:

```js
{
  estimatedReadyAt: Date,
  estimatedDeliveryAt: Date
}
```

Every estimate change is appended to:

```js
fulfilmentEstimateHistory
```

The calculation rules are documented separately in:

```text
docs/algorithms/delivery-estimation.md
```

---

## Prices and Totals

All monetary values are stored as integer cent amounts.

```js
subtotalInCents: Number
deliveryFeeInCents: Number
discountTotalInCents: Number
tipInCents: Number
totalInCents: Number
```

Calculation:

```text
totalInCents =
  subtotalInCents
  + deliveryFeeInCents
  - discountTotalInCents
  + tipInCents
```

Free delivery uses:

```js
deliveryFeeInCents: 0
```

The MVP stores:

```js
discountTotalInCents: 0
tipInCents: 0
```

Discount campaigns, coupon codes and online tipping remain outside the MVP.

Customer-facing prices are stored as gross amounts.

`taxSummary` stores the tax breakdown snapshot.

---

## Order Status

### Online Payment Flow

```text
awaiting_payment
  → new
```

A PayPal order becomes visible to the restaurant only after successful payment.

### Delivery Flow


```text
new
  → accepted
  → preparing
  → ready
  → out_for_delivery
  → delivered
```

### Failed Delivery Attempt

```text
out_for_delivery
  → delivery_failed
```

A failed delivery attempt is not treated as a successfully completed order.

A later retry uses:

```text
delivery_failed
  → out_for_delivery
```

when the order is assigned to a new delivery tour.

If no retry is planned:

```text
delivery_failed
  → cancelled
```

### Pickup Flow

```text
new
  → accepted
  → preparing
  → ready
  → picked_up
```

`preparing` can be skipped:

```text
accepted
  → ready
```

Negative terminal states:

```text
rejected
cancelled
expired
```

There is no additional `completed` status.

Successful completion is represented by:

```text
delivered
picked_up
```

Only these statuses count as successful orders for customer statistics and completed-order revenue statistics.

---

## Status History

Every status change is appended to:

```js
statusHistory
```

`orderStatus` stores the current state for efficient queries.

`statusHistory` remains the immutable audit trail for operational review and support cases.

---

## Payments

The MVP supports:

```text
paypal
cash
```

PayPal is the MVP online payment method.

Cash payment is collected at pickup or delivery according to restaurant configuration.

The order stores the current payment summary:

```js
paymentMethod
paymentStatus
paidAt
refundedAt
refundedTotalInCents
```

Payment attempts and refunds are stored separately in:

```text
payments
```

### Payment Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  orderId: ObjectId,

  type:
    "payment"
    | "refund",

  method:
    "paypal"
    | "cash",

  provider:
    "paypal"
    | null,

  amountInCents: Number,

  status:
    "pending"
    | "succeeded"
    | "failed"
    | "cancelled",

  providerReference: String | null,
  idempotencyKey: String,

  failureCode: String | null,

  completedAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

Raw card data, provider credentials and payment secrets are never stored.

### PayPal Flow

```text
create awaiting_payment order
→ reserve inventory atomically
→ create payment attempt
→ start PayPal payment
→ receive successful provider confirmation
→ set paymentStatus to paid
→ allocate order number
→ set orderStatus to new
```

When a paid order is rejected or cancelled:

```text
create refund record
→ set paymentStatus to refund_pending
→ process refund
→ set paymentStatus to refunded or partially_refunded
```

Provider webhook processing is idempotent.

---

## Cash Payment for Pickup Orders

Cash payment on pickup is protected through:

```js
cashOnPickupPolicy:
  "disabled"
  | "returning_customers_only"
  | "all_customers"
```

Default:

```js
cashOnPickupPolicy: "returning_customers_only"
```

| Value | Behavior |
|---|---|
| `disabled` | Cash payment on pickup is not offered. |
| `returning_customers_only` | Available only to registered customers with at least one successfully completed previous order. |
| `all_customers` | Available to registered customers and guests. |

A successful previous order is an order with:

```text
delivered
picked_up
```

The existing customer statistic is used:

```js
orderCount >= 1
```

Creating an account alone is not sufficient.

Guests and first-time customers can place pickup orders using PayPal.

When guest cash pickup is allowed:

- a phone number is required
- only one simultaneous unpaid cash pickup order is allowed per identity

`unpaidPickupGuardKey` is stored only while an unpaid cash pickup order remains active.

For registered customers it is derived from:

```js
customerId
```

For guests it is derived from normalized email and phone data using a stable server-side hash.

The restaurant must actively accept every order before preparation begins.

---

## Inventory Reservations

Inventory tracking is optional per menu item and refers to sellable units or portions, not ingredients.

Inventory is not reserved when products are merely added to a shopping cart.

Inventory is reserved atomically when a binding order or required online payment process starts.

Reservations are stored in:

```text
inventoryReservations
```

### Inventory Reservation Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  orderId: ObjectId,

  status:
    "active"
    | "consumed"
    | "released"
    | "expired",

  items: [
    {
      menuItemId: ObjectId,
      quantity: Number
    }
  ],

  expiresAt: Date | null,

  releaseReason:
    "payment_cancelled"
    | "payment_expired"
    | "order_rejected"
    | "order_cancelled"
    | "acceptance_expired"
    | null,

  stockRestoredAt: Date | null,
  consumedAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

Only inventory-tracked items are included.

Reservations transition from `active` to exactly one terminal state.

A conditional state transition and database transaction ensure that released quantities are restored only once.

Stock quantities never become negative.

When stock reaches `0`, the menu item becomes:

```js
{
  availabilityStatus: "temporarily_unavailable",
  unavailableReason: "sold_out"
}
```

Restocking does not automatically reactivate the product.

### Reservation Lifecycle

Online payment:

```text
create awaiting_payment order
→ create active reservation with paymentReservationExpiresAt
→ payment succeeds
→ update order to new
→ update reservation expiry to acceptanceExpiresAt
```

Cash payment:

```text
create new order
→ create active reservation with acceptanceExpiresAt
```

Restaurant acceptance:

```text
accepted
→ reservation expiresAt becomes null
```

Inventory consumption:

```text
preparing
→ reservation becomes consumed
```

If preparation is skipped:

```text
ready
→ reservation becomes consumed
```

Before consumption, quantities are restored when the order is rejected, cancelled or expired.

After consumption, cancellation does not automatically restore inventory. The restaurant can make a manual stock correction when appropriate.

Expired reservations are processed by an application worker.

TTL deletion is not used because reservation history remains auditable.

---

## Cancellation and Rejection

`rejected`, `cancelled` and `expired` have distinct meanings.

| Status | Meaning |
|---|---|
| `rejected` | Restaurant declines a new order before accepting it. |
| `cancelled` | Customer, restaurant or system actively cancels an order. |
| `expired` | System closes an incomplete or unanswered order after a timeout. |

Every rejected, cancelled or expired order stores:

```js
termination
```

### Customer Cancellation

A customer can cancel:

```text
awaiting_payment
new
```

After:

```text
accepted
```

the customer must contact the restaurant.

### Restaurant Cancellation

The restaurant uses:

```text
new
  → rejected
```

After acceptance, the restaurant can cancel:

```text
accepted
preparing
ready
out_for_delivery
delivery_failed
```

A reason is required.

### System Expiration

The system uses:

```text
awaiting_payment
  → expired

new
  → expired
```

`awaiting_payment` expires when online payment is not completed within:

```js
inventoryReservationMinutes
```

`new` expires when the restaurant does not accept the order within:

```js
orderAcceptanceTimeoutMinutes
```

If a terminated order was already paid, a refund is required.

---

## Order Numbers

Customer-facing order numbers are unique per restaurant and reset annually.

Example:

```text
2026-000124
```

`orderNumber` is allocated when an order enters:

```text
new
```

An abandoned `awaiting_payment` checkout does not receive a customer-facing order number.

Counters are stored in:

```text
orderCounters
```

### Order Counter Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  year: Number,
  nextSequence: Number,

  createdAt: Date,
  updatedAt: Date
}
```

The counter is incremented atomically.

Gaps are allowed when a transaction fails after sequence allocation.

Previously issued numbers are never reused.

---

## Reviews

Reviews are included in the MVP and stored separately from orders and menu items.

A review can be created only for an order with:

```text
delivered
picked_up
```

The restaurant controls publication.

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  orderId: ObjectId,
  customerId: ObjectId | null,

  overallRating: Number,
  deliveryRating: Number | null,
  comment: String | null,

  itemRatings: [
    {
      orderItemId: ObjectId,
      menuItemId: ObjectId,
      nameSnapshot: String,

      rating: Number,
      comment: String | null
    }
  ],

  isPublished: Boolean,
  publishedAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

A unique `orderId` index prevents multiple reviews for the same order.

Individual products such as burgers, pizzas or side dishes can receive their own rating through:

```js
itemRatings
```

Aggregated product statistics can be calculated from published reviews.

The detailed review model is documented separately in:

```text
docs/database/reviews.md
```

---

## Archiving and Retention

`isArchived` affects only the restaurant admin view.

It has no effect on:

- payments
- inventory
- statistics
- legal retention

Archiving is a manual restaurant action in the MVP.

Historical orders remain stored according to applicable legal retention requirements.

When a customer deletes an account, the customer profile is anonymized where legally possible.

Historical order snapshots remain unchanged while retention is required.

---

## Relations to Other Collections

| Collection | Relation |
|---|---|
| `restaurants` | `orders.restaurantId` references the restaurant. Restaurant configuration stores cash policy and timeout settings. |
| `locations` | `orders.locationId` references the active location used when the order was created. |
| `customers` | `orders.customerId` references a registered customer or is `null` for guest checkout. |
| `menuItems` | `orders.items[].menuItemId` references the current product while snapshot values remain immutable. |
| `payments` | Payment attempts and refunds reference `orderId`. |
| `inventoryReservations` | One inventory reservation references `orderId`. |
| `orderCounters` | Atomic counters allocate annual restaurant-specific order numbers. |
| `reviews` | At most one review references a completed order. |
| `deliveryTours` | Delivery orders reference the currently assigned tour through `deliveryTourId`. The driver is resolved through the tour. |

---

## Customer Statistics

Customer statistics are updated only when an order first reaches:

```text
delivered
picked_up
```

Fields:

```js
orderCount: Number
totalSpentCents: Number
lastOrderAt: Date | null
```

Updates are idempotent.

`customerStatisticsAppliedAt` records whether the successful completion was already applied.

Rules:

- `orderCount` increments exactly once per successfully completed order
- `totalSpentCents` increases by `totalInCents`
- a successful refund reduces `totalSpentCents` by the refunded amount
- `lastOrderAt` stores the most recent successful completion timestamp
- rejected, cancelled, expired and delivery-failed orders never increase successful-order statistics
  
---

## Validation Rules

### Orders

- `restaurantId` is required
- `locationId` is required
- `locationSnapshot` is required
- `items` contains at least one item
- every item quantity is a positive integer
- all cent amounts are integers and never negative
- `totalInCents` is calculated server-side
- `deliveryAddressSnapshot` is required for delivery and must be `null` for pickup
- a phone number is required for delivery
- a phone number is required for guest cash pickup orders
- `out_for_delivery` and `delivered` are valid only for delivery orders
- `picked_up` is valid only for pickup orders
- `orderNumber` is `null` only before an order enters `new` or when checkout terminates before submission
- a PayPal order can enter `new` only after successful payment confirmation
- the server validates menu availability, current prices and selected options during checkout
- inventory reservation and decrement operations are atomic
- inventory quantities never become negative
- an inventory reservation restores stock at most once
- a restaurant must accept a `new` order before preparation begins
- a customer cannot automatically cancel an order after `accepted`
- cash pickup orders follow `cashOnPickupPolicy`
- under `returning_customers_only`, cash pickup requires `customerId` and `customers.orderCount >= 1`
- only one active unpaid cash pickup order is allowed per `unpaidPickupGuardKey`
- every status change appends one `statusHistory` entry
- every fulfilment estimate change appends one `fulfilmentEstimateHistory` entry
- provider webhook processing is idempotent
- customer statistic updates are idempotent
- delivery address snapshots include valid GeoJSON coordinates
- `delivery_failed` is valid only for delivery orders
- `delivery_failed` does not count as a successfully completed order
- only delivery orders can reference `deliveryTourId`
- pickup orders always use `deliveryTourId: null`
- a referenced delivery tour belongs to the same restaurant and location as the order

### Reviews

- reviews can be created only for `delivered` or `picked_up` orders
- one review is allowed per order
- rating values are integers within the configured rating scale
- `deliveryRating` is `null` for pickup orders
- item ratings reference order items from the same order
- publication is controlled by the restaurant

---

## Indexes

### Orders

```js
db.orders.createIndex(
  {
    restaurantId: 1,
    orderNumber: 1
  },
  {
    unique: true,
    partialFilterExpression: {
      orderNumber: { $type: "string" }
    }
  }
)
```

```js
db.orders.createIndex({
  restaurantId: 1,
  orderStatus: 1,
  createdAt: -1
})
```

```js
db.orders.createIndex({
  restaurantId: 1,
  customerId: 1,
  createdAt: -1
})
```

```js
db.orders.createIndex(
  {
    restaurantId: 1,
    unpaidPickupGuardKey: 1
  },
  {
    unique: true,
    partialFilterExpression: {
      unpaidPickupGuardKey: { $type: "string" }
    }
  }
)
```

```js
db.orders.createIndex({
  restaurantId: 1,
  isArchived: 1,
  createdAt: -1
})
```

```js
db.orders.createIndex({
  restaurantId: 1,
  deliveryTourId: 1
})
```


### Payments

```js
db.payments.createIndex({
  orderId: 1,
  createdAt: -1
})
```

```js
db.payments.createIndex(
  {
    provider: 1,
    providerReference: 1
  },
  {
    unique: true,
    partialFilterExpression: {
      providerReference: { $type: "string" }
    }
  }
)
```

```js
db.payments.createIndex(
  {
    idempotencyKey: 1
  },
  {
    unique: true
  }
)
```

### Inventory Reservations

```js
db.inventoryReservations.createIndex(
  {
    orderId: 1
  },
  {
    unique: true
  }
)
```

```js
db.inventoryReservations.createIndex({
  status: 1,
  expiresAt: 1
})
```

### Order Counters

```js
db.orderCounters.createIndex(
  {
    restaurantId: 1,
    year: 1
  },
  {
    unique: true
  }
)
```

### Reviews

```js
db.reviews.createIndex(
  {
    orderId: 1
  },
  {
    unique: true
  }
)
```

```js
db.reviews.createIndex({
  restaurantId: 1,
  isPublished: 1,
  createdAt: -1
})
```

```js
db.reviews.createIndex({
  "itemRatings.menuItemId": 1,
  isPublished: 1
})
```

---

## MVP Scope

The MVP includes:

- delivery and pickup orders
- one active location per restaurant
- immutable location snapshots
- registered customers and guest checkout
- immutable customer, address, item, option, price and tax snapshots
- PayPal and cash payment
- configurable cash payment policy for pickup
- one active unpaid cash pickup order per customer identity
- checkout rate limiting
- active restaurant acceptance before preparation
- embedded order status history
- embedded fulfilment estimate history
- optional product-level inventory tracking
- auditable inventory reservations
- expiration of incomplete payment and unanswered new orders
- idempotent stock restoration
- separate payment attempt and refund records
- annual restaurant-specific order numbers
- manual order archiving
- idempotent customer statistics updates
- reviews for completed orders
- optional item-level ratings
- restaurant-controlled review publication
- delivery-tour assignment for delivery orders
- embedded delivery-assignment history
- failed-delivery handling and delivery retries

---

## Later Extensions

- card payment on pickup
- card payment on delivery
- additional online payment providers
- discount campaigns and coupon codes
- online tipping
- driver accounts and dedicated driver interface
- GPS-based delivery tracking
- advanced delivery estimation
- manual cash pickup exceptions for selected customers
- internal no-show markers
- customer blocking after repeated no-shows
- SMS phone verification
- automated archiving rules
- post-retention anonymization jobs
- multi-location order flows
- review moderation history
- restaurant replies to reviews

---

## Design Decisions Summary

| Topic | Decision |
|---|---|
| Order document | Stores the immutable commercial snapshot and current fulfilment state. |
| Location | Every order references one active location and stores an immutable location snapshot. |
| Payments | Attempts and refunds are stored separately from orders. |
| Inventory | Reservations are stored separately from orders and remain auditable. |
| Online payment visibility | PayPal orders become visible to the restaurant only after successful payment. |
| Cash pickup | Defaults to returning customers with at least one successful previous order. |
| Guest checkout | Guests remain supported and can use online payment. |
| Restaurant acceptance | Required before preparation starts. |
| Fulfilment statuses | `delivered` and `picked_up` represent successful completion. |
| Inventory consumption | Occurs when preparation starts or when the order becomes ready if preparation is skipped. |
| Stock restoration | Happens automatically only before inventory consumption. |
| Order numbers | Allocated atomically per restaurant and year when an order enters `new`. |
| Reviews | Stored separately and linked to completed orders. |
| Item ratings | Individual ordered products can receive their own rating. |
| Delivery tours | Delivery orders reference a tour. The assigned driver is resolved through the tour. |
| Delivery assignment history | Tour assignments and reassignments remain traceable inside the order document. |
| Failed delivery attempts | Use `delivery_failed` and can lead to a retry or cancellation. |

---

## Open Questions

None currently.
