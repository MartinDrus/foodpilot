# Restaurant and Location Data Model

## Overview

Foodpilot is an open-source ordering and delivery platform for small restaurants, snack bars and food trucks.

The MVP is optimized for one gastronomic business with one active sales, pickup and delivery origin at a time.

```text
one restaurant
one active location
one menu
one customer database
one order flow
```

A restaurant can operate from a permanent address or use a mobile operating model. A mobile business can store several recurring locations and select exactly one active location for the current operation.

Foodpilot is not a marketplace and does not provide restaurant discovery.

## Architectural Context

Restaurant and location are separate domain areas.

```text
Restaurant
→ the gastronomic business

Location
→ the currently selectable sales, pickup and delivery origin
```

Examples:

```text
Restaurant:
Burger Bros

Active location:
Wilhelm-Leuschner-Platz, Leipzig
```

```text
Restaurant:
Dönerhaus Leipzig

Active location:
Tauchaer Straße 10, Leipzig
```

The menu belongs to the restaurant:

```js
restaurantId: ObjectId
```

Menu items do not contain a `locationId` in the MVP.

Orders store both references:

```js
restaurantId: ObjectId,
locationId: ObjectId
```

Orders additionally store an immutable location snapshot. Historical orders therefore remain traceable when a mobile business selects a different active location later.

## Collections

The restaurant domain uses three collections:

```text
restaurants
locations
restaurantUsers
```

Customers remain stored separately in the `customers` collection.

## Restaurant Schema

```js
{
  _id: ObjectId,

  name: String,
  slug: String,

  description: String | null,

  locationMode:
    "fixed"
    | "mobile",

  activeLocationId: ObjectId | null,

  contact: {
    email: String | null,
    phone: String | null
  },

  legalInformation: {
    businessName: String,
    ownerName: String | null,
    legalForm: String | null,
    representedBy: String | null,

    address: {
      street: String,
      houseNumber: String,
      zip: String,
      city: String,
      countryCode: String
    },

    email: String,
    phone: String | null,

    vatId: String | null,
    registerCourt: String | null,
    registerNumber: String | null
  },

  branding: {
    logoUrl: String | null,
    coverImageUrl: String | null
  },

  currency: "EUR",

  orderConfiguration: {
    enabledPaymentMethods: [
      "paypal",
      "cash"
    ],

    cashOnPickupPolicy:
      "disabled"
      | "returning_customers_only"
      | "all_customers",

    inventoryReservationMinutes: Number,
    orderAcceptanceTimeoutMinutes: Number
  },

  menuConfiguration: {
    unavailableProductDisplayMode:
      "hidden"
      | "shown_as_unavailable"
  },

  isActive: Boolean,

  createdAt: Date,
  updatedAt: Date
}
```

## Location Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,

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
    coordinates: [
      Number,
      Number
    ]
  },

  openingHours: [
    {
      weekday: Number,

      intervals: [
        {
          opensAt: String,
          closesAt: String,
          closesNextDay: Boolean
        }
      ],

      isClosed: Boolean
    }
  ],

  openingHourExceptions: [
    {
      date: String,

      intervals: [
        {
          opensAt: String,
          closesAt: String,
          closesNextDay: Boolean
        }
      ],

      isClosed: Boolean
    }
  ],

  timezone: String,

  orderAcceptanceStatus:
    "open"
    | "paused",

  preorderSettings: {
    enabled: Boolean,

    mode:
      "next_available_time"
      | "time_slots",

    minimumLeadTimeMinutes: Number,
    maximumAdvanceDays: Number,

    slotIntervalMinutes: Number | null,
    maximumOrdersPerSlot: Number | null
  },

  deliverySettings: {
    deliveryEnabled: Boolean,
    pickupEnabled: Boolean,

    deliveryRadiusKm: Number | null,
    minimumOrderValueInCents: Number,
    deliveryFeeInCents: Number
  },

  isEnabled: Boolean,

  createdAt: Date,
  updatedAt: Date
}
```

## Restaurant User Schema

Restaurant login accounts are stored in a separate collection. They are fully documented here because they belong to the restaurant domain.

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,

  name: String,
  email: String,
  emailNormalized: String,
  passwordHash: String,

  role:
    "main_admin"
    | "operator",

  isActive: Boolean,

  lastLoginAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

## Restaurant Fields

### `name`

Public restaurant name shown in the ordering flow.

### `slug`

Stable URL-safe identifier for the restaurant ordering page.

### `description`

Optional public description.

### `locationMode`

Defines the operating model:

```text
fixed
→ the business normally operates from one permanent location

mobile
→ the business can store recurring locations and select one active location
```

### `activeLocationId`

References the currently active location used as the sales, pickup and delivery origin.

### `contact`

Public operational contact details for customers.

### `legalInformation`

Stores the business details required for the public legal information of the ordering page.

### `branding`

Stores optional local upload paths for the restaurant logo and cover image.

### `currency`

The MVP uses:

```js
currency: "EUR"
```

All monetary values are stored in cents.

### `isActive`

Controls whether the restaurant is generally enabled in Foodpilot.

```text
true
→ restaurant ordering page is active

false
→ menu can remain available for administration, but customer checkout is disabled
```

## Location Fields

### `name`

Internal and public label for the saved location.

Examples:

```text
Main restaurant
Wilhelm-Leuschner-Platz
Marktplatz Taucha
Cospudener See, entrance north
```

### `displayAddress`

Human-readable location description shown to customers. It is required for every enabled location.

### `address`

Structured postal address. Individual fields remain optional because mobile operating locations do not always have a street or house number.

### `geoLocation`

Required GeoJSON point used for radius checks, distance calculation and later route estimation.

The coordinate order is:

```text
[lng, lat]
```

Example:

```js
geoLocation: {
  type: "Point",
  coordinates: [
    12.3731,
    51.3397
  ]
}
```

### `timezone`

IANA timezone used for opening hours and preorder calculations.

Example:

```js
timezone: "Europe/Berlin"
```

### `isEnabled`

Controls whether a saved location may be selected as the active location.

## Active Location

Every restaurant has exactly one active location during customer operation.

```js
activeLocationId: ObjectId | null
```

The referenced location must:

```text
belong to the same restaurant
use isEnabled: true
contain a displayAddress
contain valid GeoJSON coordinates
```

Orders store both `locationId` and an immutable snapshot:

```js
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
    coordinates: [
      Number,
      Number
    ]
  }
}
```

## Fixed and Mobile Locations

### Fixed business

A restaurant with:

```js
locationMode: "fixed"
```

normally uses one permanent location. The active location is configured during setup and does not require a recurring selection in daily operation.

### Mobile business

A restaurant with:

```js
locationMode: "mobile"
```

can store several recurring locations. The gastronomic operator selects one active location before accepting orders.

Examples:

```text
Monday → Wilhelm-Leuschner-Platz
Wednesday → Marktplatz Taucha
Weekend → event location
```

Only one location can be active at the same time.

## Address and Coordinates

Coordinates are required for every enabled location.

Structured address fields can remain empty for mobile locations. `displayAddress` provides the customer-facing location description in these cases.

Coordinates are initially entered through a map selection or manual positioning in the admin interface. Automatic address geocoding can be added later without changing the schema.

## Opening Hours

Locations store regular weekly opening hours and explicit date-based exceptions.

Weekdays use:

```text
1 = Monday
2 = Tuesday
3 = Wednesday
4 = Thursday
5 = Friday
6 = Saturday
7 = Sunday
```

Multiple intervals per day support breaks:

```js
{
  weekday: 1,
  intervals: [
    {
      opensAt: "11:00",
      closesAt: "14:00",
      closesNextDay: false
    },
    {
      opensAt: "17:00",
      closesAt: "21:00",
      closesNextDay: false
    }
  ],
  isClosed: false
}
```

Opening hours over midnight use:

```js
closesNextDay: true
```

Date-based exceptions support holidays, vacation days and special opening hours:

```js
{
  date: "2026-12-24",
  intervals: [],
  isClosed: true
}
```

## Restaurant Availability

Three independent controls are used.

### Restaurant activation

```js
restaurants.isActive
```

Controls whether the restaurant is generally enabled.

### Location selection

```js
restaurants.activeLocationId
locations.isEnabled
```

Controls the current operating location.

### Temporary order pause

```js
locations.orderAcceptanceStatus:
  "open"
  | "paused"
```

A pause blocks new checkouts without changing opening hours or disabling the restaurant.

The menu remains visible outside opening hours and during a temporary pause.

## Preorders

Preorders are configurable per location.

```js
preorderSettings: {
  enabled: Boolean,

  mode:
    "next_available_time"
    | "time_slots",

  minimumLeadTimeMinutes: Number,
  maximumAdvanceDays: Number,

  slotIntervalMinutes: Number | null,
  maximumOrdersPerSlot: Number | null
}
```

The gastronomic operator decides whether preorders are disabled or enabled.

When preorders are enabled, the operator chooses the mode.

### `next_available_time`

The checkout shows the next available delivery or pickup time calculated from opening hours and the delivery-estimation logic.

In this mode:

```js
slotIntervalMinutes: null,
maximumOrdersPerSlot: null
```

### `time_slots`

The customer chooses from available delivery or pickup time slots.

The following field is required in this mode:

```js
slotIntervalMinutes: Number
```

The restaurant can optionally limit the number of orders per slot:

```js
maximumOrdersPerSlot: Number | null
```

Example:

```js
{
  enabled: true,
  mode: "time_slots",

  minimumLeadTimeMinutes: 30,
  maximumAdvanceDays: 7,

  slotIntervalMinutes: 30,
  maximumOrdersPerSlot: 5
}
```

Preorders are accepted only when an online payment method is enabled. Cash payment is not available for preorders.

The requested fulfilment time must:

```text
fall within the configured opening hours
respect minimumLeadTimeMinutes
not exceed maximumAdvanceDays
use an available time slot when mode is time_slots
```

When the restaurant is currently closed, the customer interface clearly shows:

```text
the restaurant is currently closed
the order is a preorder
the expected delivery or pickup time
```

When:

```js
orderAcceptanceStatus: "paused"
```

new regular orders and new preorders are blocked.

## Delivery and Pickup Settings

Delivery and pickup settings belong to the location because the active sales origin determines the delivery radius and pickup point.

```js
deliverySettings: {
  deliveryEnabled: Boolean,
  pickupEnabled: Boolean,

  deliveryRadiusKm: Number | null,
  minimumOrderValueInCents: Number,
  deliveryFeeInCents: Number
}
```

A fixed restaurant and a food truck can both offer delivery.

The MVP uses:

```text
one delivery radius
one minimum order value
one delivery fee
```

## Delivery Estimation Inputs

The restaurant and location schemas do not store a general `defaultPreparationMinutes` value.

Preparation duration belongs to each menu item:

```js
preparationTimeMinutes: Number
```

The later delivery estimation algorithm considers:

```text
preparation duration of the ordered products
open orders
route distance
planned stops on the route
handover duration per stop
driver availability
remaining duration of an active driver tour
an additional delivery buffer
```

The detailed calculation is documented separately in:

```text
docs/algorithms/delivery-estimation.md
```

## Order Configuration

Restaurant-wide order settings are grouped under:

```js
orderConfiguration
```

### Payment methods

```js
enabledPaymentMethods: [
  "paypal",
  "cash"
]
```

Only enabled methods are offered during checkout.

### Cash payment for pickup

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

Cash payment for a regular delivery is allowed when:

```js
enabledPaymentMethods.includes("cash")
```

Cash payment is never offered for preorders outside opening hours.

### Inventory reservation

```js
inventoryReservationMinutes: Number
```

Default:

```js
inventoryReservationMinutes: 15
```

### Restaurant acceptance timeout

```js
orderAcceptanceTimeoutMinutes: Number
```

Default:

```js
orderAcceptanceTimeoutMinutes: 5
```

## Menu Configuration

The restaurant-wide menu setting is:

```js
menuConfiguration: {
  unavailableProductDisplayMode:
    "hidden"
    | "shown_as_unavailable"
}
```

Default:

```js
unavailableProductDisplayMode: "shown_as_unavailable"
```

Customer-facing behavior:

```text
available
→ shown and orderable

temporarily_unavailable
→ hidden or shown as unavailable according to restaurant configuration

inactive
→ always hidden
```

## Branding

Branding is stored in:

```js
branding: {
  logoUrl: String | null,
  coverImageUrl: String | null
}
```
Images are optional and stored locally in the MVP.

## Ordering Page Integration

Foodpilot provides a standalone ordering interface for each restaurant.

The ordering page is identified through the restaurant slug:

```text
https://order.example.com/<restaurant-slug>
```

A restaurant can link this page from an existing website through a simple call-to-action button:

```html
<a href="https://order.example.com/burger-bros">
  Order online
</a>
```

The ordering flow includes:

```text
menu
→ cart
→ delivery or pickup
→ customer details
→ order summary
→ payment
→ order confirmation
```

The MVP does not include a full restaurant website builder or content management system.

## Restaurant Users and Access Roles

Restaurant users are separate from customers.

```text
customers
→ customer accounts and guest checkout data

restaurantUsers
→ gastronomic operator accounts
```

Authentication tokens must identify the account type:

```js
accountType:
  "customer"
  | "restaurant_user"
```

Restaurant user tokens additionally contain:

```js
role:
  "main_admin"
  | "operator"
```

### `main_admin`

The main administrator has full access, including:

```text
restaurant basis data
legal information
branding
payment settings
restaurant users
locations
opening hours
preorder settings
menu configuration
order processing
```

Every restaurant must have at least one active `main_admin` account.

### `operator`

The operator account is intended for daily operational tasks, including:

```text
process orders
pause and resume order acceptance
mark products as sold out or available
update inventory quantities
update opening hours and opening-hour exceptions
select the active location for a mobile business
```

The operator cannot change legal information, payment settings or restaurant user accounts.

## Relations to Other Collections

### Menu items

```js
menuItems.restaurantId: ObjectId
```

Menu items belong to the restaurant and do not contain `locationId` in the MVP.

### Orders

```js
orders.restaurantId: ObjectId,
orders.locationId: ObjectId,
orders.locationSnapshot: Object
```

### Customers

```js
orders.customerId: ObjectId | null
```

Guest checkout remains supported.

### Restaurant users

```js
restaurantUsers.restaurantId: ObjectId
```

### Drivers

Drivers and active delivery tours are documented separately. They are used by the delivery estimation algorithm.

## Validation Rules

### Restaurant

```text
name is required
slug is required and unique
locationMode is required
currency is always EUR in the MVP
activeLocationId is required before customer checkout is enabled
activeLocationId must reference an enabled location of the same restaurant
at least one active main_admin account is required
inventoryReservationMinutes must be greater than 0
orderAcceptanceTimeoutMinutes must be greater than 0
```

### Location

```text
restaurantId is required
name is required
displayAddress is required when isEnabled is true
geoLocation is required when isEnabled is true
geoLocation.type must equal Point
geoLocation.coordinates must contain [lng, lat]
longitude must be between -180 and 180
latitude must be between -90 and 90
timezone is required
at least deliveryEnabled or pickupEnabled must be true before the location is selected as active
deliveryRadiusKm is required when deliveryEnabled is true
deliveryRadiusKm is null when deliveryEnabled is false
minimumOrderValueInCents must be at least 0
deliveryFeeInCents must be at least 0
```

### Opening hours

```text
weekday must be between 1 and 7
opensAt and closesAt use HH:mm format
openingHourExceptions.date uses YYYY-MM-DD format
intervals must be empty when isClosed is true
```

### Preorders

```text
preorderSettings.enabled is required
minimumLeadTimeMinutes must be at least 0
maximumAdvanceDays must be greater than 0 when preorderSettings.enabled is true
mode must be next_available_time or time_slots when preorderSettings.enabled is true
slotIntervalMinutes is required when mode is time_slots
slotIntervalMinutes is null when mode is next_available_time
maximumOrdersPerSlot is null or greater than 0
preorders require an enabled online payment method
```

### Restaurant users

```text
restaurantId is required
name is required
email is required
emailNormalized is required
emailNormalized is lowercase and trimmed
emailNormalized is unique
passwordHash is required
role is required
```

## Indexes

```js
db.restaurants.createIndex(
  {
    slug: 1
  },
  {
    unique: true
  }
)
```

```js
db.locations.createIndex({
  restaurantId: 1,
  isEnabled: 1
})
```

```js
db.locations.createIndex({
  geoLocation: "2dsphere"
})
```

```js
db.restaurantUsers.createIndex(
  {
    emailNormalized: 1
  },
  {
    unique: true
  }
)
```

```js
db.restaurantUsers.createIndex({
  restaurantId: 1,
  role: 1,
  isActive: 1
})
```

## MVP Scope

The MVP includes:

```text
standalone restaurant ordering page
simple integration into existing restaurant websites through a link or button
one active location at a time
fixed or mobile operating mode
stored recurring locations for mobile businesses
manual active-location selection
GeoJSON coordinates for all enabled locations
regular opening hours
opening-hour exceptions
manual temporary order pause
optional preorders outside opening hours
operator-selected preorder mode
online-payment-only rule for preorders
pickup and radius-based delivery
one minimum order value
one delivery fee
restaurant-wide order and menu configuration
local logo and cover-image uploads
separate main-admin and operator accounts
```

## Later Extensions

The following features are outside the MVP:

```text
custom ordering domains
embeddable ordering widgets
WordPress integration
headless API integrations
automatic address geocoding
complex delivery zones
postal-code delivery zones
polygon-based delivery areas
distance-based delivery-fee tiers
additional restaurant-user roles
fine-grained custom permissions
additional online payment providers
online card payments
Apple Pay and Google Pay through a payment provider
card payment on pickup
card payment on delivery
```

## Design Decisions Summary

```text
Restaurant and location are separate domain areas.

Every restaurant has exactly one active location at a time.

A fixed restaurant normally stores one permanent location.

A mobile business can store recurring locations and select one active location.

All enabled locations require coordinates.

Locations use GeoJSON points with [lng, lat] order.

Menu items belong to the restaurant and do not contain locationId in the MVP.

Orders store restaurantId, locationId and an immutable location snapshot.

Opening-hour exceptions are part of the MVP.

The menu remains visible outside opening hours.

Preorders are configurable per location.

The gastronomic operator decides whether preorders are disabled, use the next available time or use selectable time slots.

Preorders require online payment.

A fixed restaurant and a food truck can both offer delivery.

The MVP uses a delivery radius, one minimum order value and one delivery fee.

Preparation duration belongs to menu items, not to restaurants or locations.

Restaurant users are stored separately from customer accounts.

Restaurant access uses main_admin and operator roles.
```

## Open Questions

None currently.
