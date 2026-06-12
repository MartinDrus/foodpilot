# Reviews Collection

## Purpose

The `reviews` collection stores verified customer reviews for restaurants and individual ordered products.

A review is always linked to a successfully completed order and can only be created by a registered customer.

Public reviews without a verified order are not supported.

The collection is kept separate from `orders` so that reviews can be queried, moderated and aggregated without loading order data.

---

## Collection

```text
reviews
```

---

## Review Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,
  locationId: ObjectId,
  orderId: ObjectId,
  customerId: ObjectId,

  authorDisplayName: String,

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

  moderationStatus:
    "published"
    | "hidden",

  moderationReason:
    "spam"
    | "abuse"
    | "personal_data"
    | "other"
    | null,

  moderatedAt: Date | null,
  publishedAt: Date | null,
  deletedAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

---

## Field Definitions

### Restaurant, Location and Order References

```js
restaurantId: ObjectId
locationId: ObjectId
orderId: ObjectId
```

Each review belongs to exactly one restaurant, one location and one successfully completed order.

The location is copied from the linked order. This supports later location-specific review statistics without requiring an order lookup for every review.

Only one review record can exist per order.

### Customer Reference

```js
customerId: ObjectId
```

A review can only be created by a registered customer.

The linked order must belong to the authenticated customer.

Guest checkout remains supported, but guest customers cannot create reviews in the MVP.

### Public Author Name

```js
authorDisplayName: String
```

This field contains a public-safe display name.

Example:

```text
Martin D.
```

Full names, email addresses, phone numbers and delivery addresses are not stored in the review document.

### Overall Rating

```js
overallRating: Number
```

The overall rating is required and stored as an integer between `1` and `5`.

### Delivery Rating

```js
deliveryRating: Number | null
```

The delivery rating is optional.

For pickup orders:

```js
deliveryRating: null
```

For delivery orders, customers may provide an additional integer rating between `1` and `5`.

### Comment

```js
comment: String | null
```

A written restaurant-level comment is optional.

The comment is trimmed before storage and limited to `2000` characters.

### Item Ratings

```js
itemRatings: [
  {
    orderItemId: ObjectId,
    menuItemId: ObjectId,
    nameSnapshot: String,

    rating: Number,
    comment: String | null
  }
]
```

Customers can optionally rate individual ordered products such as burgers, pizzas, side dishes or drinks.

Each item rating references the purchased order item and the corresponding menu item.

`nameSnapshot` preserves the product name shown in the historical order.

The item-level comment is optional, trimmed before storage and limited to `1000` characters.

### Moderation

```js
moderationStatus:
  "published"
  | "hidden"
```

New reviews use:

```js
moderationStatus: "published"
```

A hidden review is excluded from public display and rating aggregation.

```js
moderationReason:
  "spam"
  | "abuse"
  | "personal_data"
  | "other"
  | null
```

`moderationReason` and `moderatedAt` are set when a review is hidden.

Restaurants cannot edit or hide customer reviews directly.

Moderation actions are handled by the platform.

### Soft Deletion

```js
deletedAt: Date | null
```

Customers can delete their own reviews.

Deletion is implemented as a soft delete.

Deleted reviews are excluded from public display and rating aggregation.

---

## Review Eligibility

A review can only be created by a registered customer.

The linked order must:

```text
belong to the authenticated customer
use order status delivered or picked_up
not already have a review
```

Guest checkout remains supported, but guest customers cannot create reviews in the MVP.

---

## Review Behaviour

- only one review record can exist per order
- a review can be edited by its author
- editing a hidden review does not publish it again automatically
- a review can be soft-deleted by its author
- restaurants cannot modify customer ratings or comments
- restaurants cannot publish or hide reviews directly
- hidden and deleted reviews remain excluded from public statistics
- overall restaurant ratings are part of the MVP
- optional delivery ratings are part of the MVP
- optional item-level ratings are part of the MVP
- driver ratings are outside the MVP scope

---

## Rating Aggregation

Restaurant ratings are calculated only from reviews that match:

```js
{
  moderationStatus: "published",
  deletedAt: null
}
```

Product ratings are calculated only from published and non-deleted reviews containing matching entries in:

```js
itemRatings
```

Location-specific statistics can later be calculated through:

```js
locationId
```

Denormalised aggregate values are not stored in the MVP.

If performance requirements justify it later, aggregate values such as the following can be added:

```js
averageRating: Number | null,
reviewCount: Number
```

---

## Validation Rules

- `restaurantId` is required
- `locationId` is required
- `orderId` is required
- `customerId` is required
- `orderId` must reference a successfully completed order
- the order must belong to the same restaurant as `restaurantId`
- `locationId` must match the location stored on the order
- `customerId` must match the customer linked to the order
- guest orders cannot receive reviews in the MVP
- only one review record can exist per order
- `authorDisplayName` is required and must be public-safe
- `overallRating` must be an integer between `1` and `5`
- `deliveryRating` is `null` for pickup orders
- `deliveryRating`, when present, must be an integer between `1` and `5`
- item ratings reference order items from the same order
- item ratings reference menu items stored on the same order items
- each order item can be rated at most once within a review
- item-level `rating` values must be integers between `1` and `5`
- restaurant-level comments are optional and limited to `2000` characters
- item-level comments are optional and limited to `1000` characters
- `moderationReason` must be `null` while `moderationStatus` is `"published"`
- `moderatedAt` must be set when `moderationStatus` is `"hidden"`
- hidden and deleted reviews are excluded from public display and rating aggregation

---

## Indexes

Only one review can exist per order:

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

Public restaurant review list:

```js
db.reviews.createIndex({
  restaurantId: 1,
  moderationStatus: 1,
  deletedAt: 1,
  createdAt: -1
})
```

Location-specific review queries:

```js
db.reviews.createIndex({
  restaurantId: 1,
  locationId: 1,
  moderationStatus: 1,
  deletedAt: 1,
  createdAt: -1
})
```

Customer review history:

```js
db.reviews.createIndex({
  customerId: 1,
  createdAt: -1
})
```

Published product rating queries:

```js
db.reviews.createIndex({
  "itemRatings.menuItemId": 1,
  moderationStatus: 1,
  deletedAt: 1
})
```

---

## Relationships

```text
restaurants  1 ─── n reviews
locations    1 ─── n reviews
orders       1 ─── 0..1 reviews
customers    1 ─── n reviews
```

---

## MVP Scope

Included in the MVP:

- restaurant ratings from `1` to `5`
- optional written restaurant-level comments
- optional delivery ratings for delivery orders
- optional item-level ratings and comments
- one verified review per completed order
- reviews only from registered customers
- customer editing and soft deletion
- automatic publication of new reviews
- basic platform moderation
- exclusion of hidden and deleted reviews from public statistics

---

## Later Extensions

Possible later extensions:

- guest reviews through hashed single-use review tokens
- restaurant replies
- report workflow for restaurants and customers
- moderation audit logs
- automated abuse and spam detection
- denormalised restaurant rating aggregates
- denormalised product rating aggregates
- location-specific rating displays
- food quality and service sub-ratings
- driver ratings
- review images
- helpful-vote functionality

---

## Design Decisions Summary

| Topic | Decision |
|---|---|
| Review source | Reviews require a successfully completed order. |
| Registered customers | Only registered customers can create reviews in the MVP. |
| Review uniqueness | Each order can receive at most one review. |
| Restaurant rating | Every review stores a required overall rating from `1` to `5`. |
| Delivery rating | Delivery orders can receive an optional delivery rating. |
| Item ratings | Individual purchased products can receive optional ratings and comments. |
| Location reference | Every review stores the location copied from the linked order. |
| Publication | New reviews are published automatically. |
| Moderation | Restaurants cannot edit or hide reviews directly. Moderation is handled by the platform. |
| Soft deletion | Customers can soft-delete their own reviews. |
| Aggregation | MVP aggregates ratings from published, non-deleted reviews without storing denormalised values. |

---

## Open Questions

None currently.
