# Menu and Product Data Model

## Overview

Foodpilot stores a restaurant menu as structured categories and menu items.

The menu model covers:

- categories
- menu items
- prices and tax rates
- ingredients, allergens and additives
- optional product images
- manual product availability
- optional product-level inventory tracking
- selectable options and variants

Historical orders remain unchanged when the menu is edited. Orders store snapshots of all product and pricing values relevant to the purchase.

---

## Architectural Context

Foodpilot is designed primarily for small restaurants, takeaways and food trucks operating through one active location.

The default deployment model is:

- one restaurant
- one active location
- one menu
- one customer database
- one order flow

Foodpilot is not a restaurant marketplace or discovery platform.

Every category and menu item belongs to exactly one restaurant through `restaurantId`.

Location data is managed separately in the `locations` collection.

The active location can represent a fixed business address or a mobile operating point and can be updated without changing the menu structure.

---

## Collections

The menu is stored in two collections:

```text
menuCategories
menuItems
```

The MVP does not use a separate `menus` collection. Each restaurant has one menu represented by its categories and menu items.

---

## Schema

### `menuCategories`

```js
{
  _id: ObjectId,
  restaurantId: ObjectId,

  name: String,
  slug: String,
  description: String | null,

  sortOrder: Number,
  isActive: Boolean,

  createdAt: Date,
  updatedAt: Date
}
```

### `menuItems`

```js
{
  _id: ObjectId,
  restaurantId: ObjectId,
  categoryId: ObjectId,

  internalNumber: String | null,
  name: String,
  slug: String,
  description: String | null,

  priceInCents: Number,
  taxRate: Number,

  ingredients: [String],
  allergenCodes: [String],
  additiveCodes: [String],
  labels: [String],

  imageUrl: String | null,

  preparationTimeMinutes: Number,

  availabilityStatus:
    "available"
    | "temporarily_unavailable"
    | "inactive",

  unavailableReason:
    "sold_out"
    | "manual_pause"
    | null,

  unavailableSince: Date | null,

  inventoryTrackingEnabled: Boolean,
  stockQuantity: Number | null,
  lowStockThreshold: Number | null,

  options: [
    {
      _id: ObjectId,

      name: String,
      type: "single" | "multiple",

      required: Boolean,
      minSelections: Number,
      maxSelections: Number,

      choices: [
        {
          _id: ObjectId,

          name: String,
          priceModifierInCents: Number,

          allergenCodes: [String],
          additiveCodes: [String],

          isDefault: Boolean,
          isActive: Boolean
        }
      ]
    }
  ],

  sortOrder: Number,

  createdAt: Date,
  updatedAt: Date
}
```

---

## Shared Fields

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `_id` | `ObjectId` | Yes | Generated | Internal document identifier. |
| `restaurantId` | `ObjectId` | Yes | — | Restaurant that owns the category or menu item. |
| `createdAt` | `Date` | Yes | Generated | Creation timestamp. |
| `updatedAt` | `Date` | Yes | Generated | Last update timestamp. |

---

## Categories

Categories structure the menu and define the display order of product groups.

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `name` | `String` | Yes | — | Visible category name. |
| `slug` | `String` | Yes | — | URL-safe identifier, unique within the restaurant. |
| `description` | `String \| null` | No | `null` | Optional category description. |
| `sortOrder` | `Number` | Yes | `0` | Manual display order. Lower values appear first. |
| `isActive` | `Boolean` | Yes | `true` | Controls whether the category is visible and orderable. |

Inactive categories are hidden from customers. Their products remain stored and can be reactivated later.

---

## Menu Items

A menu item represents one orderable product, such as a pizza, burger, drink or side dish.

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `categoryId` | `ObjectId` | Yes | — | Category that contains the menu item. |
| `internalNumber` | `String \| null` | No | `null` | Optional internal article number such as `P-101`. |
| `name` | `String` | Yes | — | Visible product name. |
| `slug` | `String` | Yes | — | URL-safe identifier, unique within the restaurant. |
| `description` | `String \| null` | No | `null` | Optional product description. |
| `sortOrder` | `Number` | Yes | `0` | Manual display order within the category. |

---

## Prices

All monetary values are stored as integer cent values.

```js
priceInCents: 990
```

Option surcharges follow the same rule:

```js
priceModifierInCents: 200
```

Floating-point prices are not stored.

---

## Tax Rate

Each menu item stores its applicable tax rate:

```js
taxRate: 7
```

The value is stored as a percentage number and copied into the order snapshot when an order is created.

---

## Ingredients

Ingredients are stored as customer-facing strings:

```js
ingredients: [
  "Tomatensauce",
  "Käse",
  "Salami"
]
```

Ingredients provide product information only. They are not used for inventory tracking in the MVP.

---

## Allergens and Additives

Allergen and additive definitions remain application constants in the MVP.

The restaurant operator independently selects the relevant stable codes while creating or editing a menu item.

```js
allergenCodes: [
  "gluten",
  "milk"
],

additiveCodes: [
  "preservatives"
]
```

The default values are empty arrays:

```js
allergenCodes: [],
additiveCodes: []
```

Option choices can introduce additional allergens or additives:

```js
choices: [
  {
    _id: ObjectId,

    name: String,
    priceModifierInCents: Number,

    allergenCodes: [String],
    additiveCodes: [String],

    isDefault: Boolean,
    isActive: Boolean
  }
]
```

The frontend combines the codes from the main product and the selected choices.

---

## Product Labels

Product labels provide additional customer-facing information.

```js
labels: [
  "vegan",
  "vegetarian",
  "spicy",
  "gluten_free",
  "lactose_free",
  "halal"
]
```
---

## Images

Product images are optional.

```js
imageUrl: String | null
```

The self-hosted MVP stores uploaded images locally. Image files are not embedded in MongoDB documents.

---

## Preparation Time

Each menu item stores an estimated preparation time:

```js
preparationTimeMinutes: 15
```

The value is used as input for pickup and delivery-time estimates.

---

## Availability

Product availability is controlled through an explicit status:

```js
availabilityStatus:
  "available"
  | "temporarily_unavailable"
  | "inactive"
```

| Status | Description |
|---|---|
| `available` | Product is visible and orderable. |
| `temporarily_unavailable` | Product is paused or sold out until the restaurant operator actively reactivates it. |
| `inactive` | Product is not part of the current menu but remains stored. |

Additional fields provide context:

```js
unavailableReason:
  "sold_out"
  | "manual_pause"
  | null,

unavailableSince: Date | null
```

A temporarily unavailable product remains unavailable until the restaurant operator explicitly marks it as available again.

There is no automatic daily reset.

### Customer-Facing Display

The customer-facing display mode for unavailable products is configured at restaurant level:

```js
unavailableProductDisplayMode:
  "hidden"
  | "shown_as_unavailable"
```

This field belongs to the restaurant configuration and is not stored in `menuItems`.

A product-specific override can be introduced later if required.

---

## Optional Inventory Tracking

Inventory tracking is optional per menu item and refers to orderable units or portions, not ingredient quantities.

```js
{
  inventoryTrackingEnabled: true,
  stockQuantity: 18,
  lowStockThreshold: 5
}
```

A menu item without inventory tracking uses:

```js
{
  inventoryTrackingEnabled: false,
  stockQuantity: null,
  lowStockThreshold: null
}
```

### Inventory Fields

| Field | Type | Required | Default | Description |
|---|---|---:|---:|---|
| `inventoryTrackingEnabled` | `Boolean` | Yes | `false` | Enables product-level inventory tracking. |
| `stockQuantity` | `Number \| null` | Conditional | `null` | Number of remaining orderable units or portions. |
| `lowStockThreshold` | `Number \| null` | No | `null` | Threshold for an admin warning. |

### Inventory Behaviour

When inventory tracking is enabled:

- stock is reserved atomically when a binding order is created
- products are not reserved merely because they were added to a shopping cart
- reservations expire automatically if checkout or payment is not completed within the configured reservation period
- releasing a reservation is idempotent and can restore the reserved quantity only once
- reserved quantities are released when an order fails, is rejected, expires or is cancelled before fulfilment
- `stockQuantity` never becomes negative
- when `stockQuantity` reaches `0`, the product uses:

```js
availabilityStatus: "temporarily_unavailable",
unavailableReason: "sold_out"
```

- a product with enabled inventory tracking cannot be marked as `available` while `stockQuantity` is `0`
- increasing the stock does not automatically reactivate the product
- after restocking, the restaurant operator must explicitly mark the product as available again
- the admin interface shows a warning when `stockQuantity` is less than or equal to `lowStockThreshold`

The exact implementation of inventory reservations belongs in the order and payment documentation. This document defines the business rules only.

---

## Options and Variants

Options represent selectable product variations and extras.

Examples:

- size
- extras
- side dishes
- spice level

Each option group defines whether one or multiple choices can be selected.

### Single Selection

```js
options: [
  {
    name: "Größe",
    type: "single",

    required: true,
    minSelections: 1,
    maxSelections: 1,

    choices: [
      {
        name: "Normal",
        priceModifierInCents: 0,
        allergenCodes: [],
        additiveCodes: [],
        isDefault: true,
        isActive: true
      },
      {
        name: "Groß",
        priceModifierInCents: 200,
        allergenCodes: [],
        additiveCodes: [],
        isDefault: false,
        isActive: true
      }
    ]
  }
]
```

### Multiple Selection

```js
{
  name: "Extras",
  type: "multiple",

  required: false,
  minSelections: 0,
  maxSelections: 5,

  choices: [
    {
      name: "Extra Käse",
      priceModifierInCents: 150,
      allergenCodes: ["milk"],
      additiveCodes: [],
      isDefault: false,
      isActive: true
    }
  ]
}
```

Inactive choices remain stored but cannot be selected in new orders.

Inventory tracking applies to the menu item as a whole in the MVP.

---

## Relation to Orders

Orders store snapshots of purchased menu items.

A stored order item contains at least:

```js
{
  menuItemId: ObjectId,

  internalNumber: "P-101",
  name: "Pizza Salami",

  quantity: 2,

  unitPriceInCents: 990,
  taxRate: 7,

  allergenCodes: ["gluten", "milk"],
  additiveCodes: ["preservatives"],

  selectedOptions: [
    {
      optionGroupId: ObjectId,
      optionGroupName: "Größe",

      choiceId: ObjectId,
      choiceName: "Groß",

      priceModifierInCents: 200,
      allergenCodes: [],
      additiveCodes: []
    }
  ],

  lineTotalInCents: 2380
}
```

The snapshot stores:

- product name
- internal product number, when present
- item price in cents
- tax rate
- allergen and additive codes of the product
- selected option group names
- selected choice names
- selected option surcharges in cents
- allergen and additive codes of selected choices
- calculated line total in cents

References remain stored for traceability. Historical display and calculations always use snapshot values.

---

## Validation Rules

### Categories

- `restaurantId` is required
- `name` is required, trimmed and non-empty
- `slug` is required and unique within the restaurant
- `sortOrder` is an integer greater than or equal to `0`
- `isActive` is a Boolean value

### Menu Items

- `restaurantId` is required
- `categoryId` is required
- the referenced category belongs to the same restaurant
- `name` is required, trimmed and non-empty
- `slug` is required and unique within the restaurant
- `internalNumber`, when present, is unique within the restaurant
- `priceInCents` is an integer greater than or equal to `0`
- `taxRate` is a numeric percentage value between `0` and `100`
- `preparationTimeMinutes` is an integer greater than or equal to `0`
- `sortOrder` is an integer greater than or equal to `0`
- `imageUrl`, when present, is a string path or URL
- `allergenCodes` and `additiveCodes` default to empty arrays

### Availability

- `availabilityStatus` is one of the documented status values
- `unavailableReason` is `null` while the item is available
- `unavailableSince` is `null` while the item is available
- temporarily unavailable items contain an `unavailableReason`
- temporarily unavailable items contain an `unavailableSince` timestamp

### Inventory

- when inventory tracking is enabled and `stockQuantity` reaches `0`, the product uses `availabilityStatus: "temporarily_unavailable"` and `unavailableReason: "sold_out"`
- `stockQuantity` is required when inventory tracking is enabled
- `stockQuantity` and `lowStockThreshold`, when present, are integers greater than or equal to `0`
- disabled inventory tracking uses `stockQuantity: null` and `lowStockThreshold: null`
- a product with enabled inventory tracking cannot be marked as `available` while `stockQuantity` is `0`
- customer input never determines stored stock quantities

### Options

- every option group has a non-empty `name`
- `type` is either `single` or `multiple`
- `minSelections` and `maxSelections` are integers greater than or equal to `0`
- `maxSelections` is greater than or equal to `minSelections`
- a required group has `minSelections` greater than or equal to `1`
- a `single` group has `maxSelections` equal to `1`
- every choice has a non-empty `name`
- `priceModifierInCents` is an integer greater than or equal to `0`
- choice-level `allergenCodes` and `additiveCodes` default to empty arrays
- a `single` group has at most one active default choice

---

## Indexes

### `menuCategories`

```js
{ restaurantId: 1, slug: 1 }
```

Unique index for category slugs within a restaurant.

```js
{ restaurantId: 1, isActive: 1, sortOrder: 1 }
```

Index for rendering active categories in display order.

### `menuItems`

```js
{ restaurantId: 1, slug: 1 }
```

Unique index for product slugs within a restaurant.

```js
db.menuItems.createIndex(
  {
    restaurantId: 1,
    internalNumber: 1
  },
  {
    unique: true,
    partialFilterExpression: {
      internalNumber: { $type: "string" }
    }
  }
)
```

Partial unique index for internal product numbers when `internalNumber` is present.

```js
{ restaurantId: 1, categoryId: 1, availabilityStatus: 1, sortOrder: 1 }
```

Index for rendering products inside a category.

```js
{ restaurantId: 1, availabilityStatus: 1 }
```

Index for filtering available, paused and inactive products.

```js
{ restaurantId: 1, inventoryTrackingEnabled: 1, stockQuantity: 1 }
```

Index for filtering products with low or depleted stock.

---

## MVP Scope

The MVP includes:

- categories and menu items
- prices stored as integer cents
- tax rate per menu item
- ingredients
- allergens and additives selected by the restaurant operator
- one optional product image
- preparation time per menu item
- manual product availability
- optional product-level inventory tracking
- one low-stock warning threshold
- option groups and selectable choices
- product and pricing snapshots in orders
- optional customer-facing product labels

---

## Later Extensions

The following features remain outside the MVP scope:

- location-specific prices or availability overrides
- managed installations containing multiple independent restaurants
- multiple menus per restaurant
- scheduled product availability
- product-specific overrides for unavailable-product display
- customer-facing low-stock display
- exact public remaining quantities
- a second critical stock threshold
- ingredient-level inventory management
- recipe-based stock calculation
- variant-specific inventory tracking
- centralized allergen and additive catalogs
- multiple images per product
- product import and export
- category and product translations
- external object storage for product images

---

## Design Decisions Summary

| Topic | Decision |
|---|---|
| Default architecture | One restaurant operates through one active location in the MVP. |
| Restaurant ownership | Every category and menu item stores `restaurantId`. |
| Money storage | All monetary values are stored as integer cents. |
| Historical orders | Orders store snapshots of product, price, tax, allergen, additive and selected-option data. |
| Availability | Products use `available`, `temporarily_unavailable` and `inactive`. |
| Availability reactivation | Unavailable products remain unavailable until explicitly reactivated by the restaurant operator. |
| Unavailable-product display | Configured at restaurant level. |
| Inventory tracking | Optional product-level inventory tracks orderable units or portions, not ingredients. |
| Inventory depletion | Products become temporarily unavailable when stock reaches `0`. |
| Options | Embedded option groups support sizes, extras, side dishes and spice levels. |
| Allergens and additives | Stable application-defined codes are stored for products and option choices. |
| Images | One optional local image path or URL is stored per product. |
| Product labels | Restaurant operators can add optional customer-facing labels to menu items. |

---

## Open Questions

None currently.
