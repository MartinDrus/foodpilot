# Drivers and Delivery Tours

## Overview

Foodpilot stores drivers in the `drivers` collection and concrete delivery runs in the `deliveryTours` collection.

A delivery tour represents one physical trip from the restaurant location to one or more customers and back to the same restaurant location. A delivery order does not automatically create a tour. It is assigned during dispatch planning to an existing open planned tour or to a newly created planned tour.

The model supports small restaurants, snack bars and food trucks without introducing shift planning, live GPS tracking or a custom route-planning algorithm into the MVP.

## Architectural Context

Foodpilot uses the following standard model:

```text
one restaurant
one active location
one menu
one customer database
one order flow
```

A restaurant can operate at a fixed location or use recurring mobile locations. Every delivery tour belongs to the active restaurant location from which the delivery run starts.

A tour starts and ends at the same restaurant location in the MVP:

```text
restaurant location
→ one or more customer stops
→ restaurant location
```

Route planning and delivery-time calculation are handled outside this document. The tour model stores the operational data required by those modules.

## Collections

```text
drivers
deliveryTours
```

## Driver Schema

```js
{
  _id: ObjectId,

  restaurantId: ObjectId,

  name: String,
  phone: String | null,

  vehicleType:
    "bike"
    | "car"
    | "scooter"
    | "walking"
    | "other",

  dutyStatus:
    "off_duty"
    | "on_duty"
    | "paused",

  internalNote: String | null,

  isActive: Boolean,

  createdAt: Date,
  updatedAt: Date
}
```

## Delivery Tour Schema

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

  driverId: ObjectId | null,
  assignedAt: Date | null,

  status:
    "planned"
    | "in_progress"
    | "returning"
    | "completed"
    | "cancelled",

  loadingStatus:
    "open"
    | "closed",

  stops: [
    {
      orderId: ObjectId,

      sequence: Number,

      status:
        "pending"
        | "out_for_delivery"
        | "delivered"
        | "failed"
        | "cancelled",

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
      },

      plannedHandoverDurationMinutes: Number,
      estimatedArrivalAt: Date | null,

      deliveredAt: Date | null,
      failedAt: Date | null,
      cancelledAt: Date | null,

      failureReason:
        "customer_unavailable"
        | "wrong_address"
        | "customer_refused"
        | "payment_issue"
        | "other"
        | null,

      note: String | null
    }
  ],

  estimatedReturnAt: Date | null,

  startedAt: Date | null,
  returnStartedAt: Date | null,
  completedAt: Date | null,
  cancelledAt: Date | null,

  createdAt: Date,
  updatedAt: Date
}
```

## Driver Fields

### `restaurantId`

A driver belongs to exactly one restaurant in the MVP.

### `dutyStatus`

`dutyStatus` describes whether the driver is currently available for dispatch planning.

```text
off_duty
= the driver is not working

on_duty
= the driver is working and can be included in dispatch planning

paused
= the driver is temporarily unavailable for a new tour
```

The driver's current operational position is not stored separately. It is derived from the driver's active tour.

### `isActive`

`isActive` describes whether the driver profile may still be used. Former drivers are retained for historical references but cannot receive new tour assignments.

### `vehicleType`

`vehicleType` stores the driver's usual delivery vehicle. It can be used by later route-planning and delivery-estimation modules.

## Driver Status

The application derives the operational driver state from `dutyStatus` and the active tour:

```text
on_duty + no active tour
→ at restaurant location

on_duty + tour status "in_progress"
→ delivering

on_duty + tour status "returning"
→ returning to restaurant location
```

`assigned`, `delivering` and `returning` are not stored as separate driver status values.

## Tour Status

```text
planned
→ in_progress
→ returning
→ completed
```

A tour can also end as:

```text
cancelled
```

### `planned`

The tour exists but has not started. Stops can be added, removed or reordered while `loadingStatus` is `"open"`.

### `in_progress`

The driver has left the restaurant location. The tour is closed for additional orders. The included delivery orders use:

```text
out_for_delivery
```

### `returning`

All non-cancelled stops have ended as `"delivered"` or `"failed"`. The driver is returning to the restaurant location.

### `completed`

The driver has returned to the restaurant location and the tour is finished.

## Loading Status

```text
open
= additional orders may be added to the planned tour

closed
= the driver takes no additional orders on this tour
```

The restaurant operator can close a planned tour manually. Starting a tour always sets:

```js
loadingStatus: "closed"
```

A running tour cannot receive additional orders.

## Active and Planned Tours

A driver can have at most:

```text
one active tour
plus one planned next tour
```

The active tour uses:

```text
in_progress
returning
```

Additional future tours are not created in advance. If delivery demand exceeds the capacity of the active and next tour, the delivery-estimation module increases the estimated delivery time or blocks further delivery orders according to the restaurant configuration.

The `drivers` collection does not store:

```js
currentTourId
nextTourId
```

Tours are queried through `deliveryTours.driverId` and `deliveryTours.status` to avoid duplicated state.

## Stops

Each stop belongs to exactly one delivery order and stores an immutable delivery-address snapshot including coordinates.

`sequence` stores the order in which stops are served. The value is required, unique within the tour and relevant for delivery-time calculation.

`plannedHandoverDurationMinutes` stores the handover duration used by the delivery-estimation module for this stop.

`estimatedArrivalAt` is a calculated value maintained by the delivery-estimation module. The calculation method is outside the scope of this document.

`arrivedAt` is not stored in the MVP because it cannot be captured reliably without a driver app or GPS tracking.

## Assignment Workflow

A new delivery order is initially not assigned to a tour:

```js
deliveryTourId: null
```

During dispatch planning:

```text
new delivery order
→ add to an existing open planned tour
→ or create a new planned tour
```

A tour can be prepared while the assigned driver is still completing an active tour.

If the restaurant operator decides that the driver must not take additional orders, the planned tour is closed:

```js
loadingStatus: "closed"
```

When the tour starts:

```text
tour status
planned → in_progress

stop status
pending → out_for_delivery

order status
ready → out_for_delivery
```

After the last stop:

```text
tour status
in_progress → returning
```

After the driver returns to the restaurant location:

```text
tour status
returning → completed
```

## Relation to Orders

Only delivery orders can be assigned to a tour.

Pickup orders never receive a tour reference.

Delivery orders store:

```js
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
]
```

Orders do not store:

```js
driverId
```

The assigned driver is resolved through:

```text
order
→ deliveryTour
→ driver
```

This prevents duplicated assignment state when a planned tour is reassigned to another driver.

## Delivery Failure Handling

A failed delivery attempt is not treated as a successful order and is not silently converted into a cancellation.

A failed stop does not prevent the tour from continuing to later stops or from entering `returning` after the final stop.

The stop stores:

```js
status: "failed",
failedAt: Date,
failureReason: String
```

The related order uses:

```text
delivery_failed
```

A later delivery retry creates a new stop in a new tour. The failed stop remains in the original tour for traceability.

The restaurant operator can cancel the order if no further delivery attempt will be made:

```text
delivery_failed
→ cancelled
```

## Driver Access

Drivers do not require their own login in the MVP.

Drivers are created and managed by the restaurant operator. The operator manages tour assignment and delivery status updates.

A dedicated driver login, a mobile driver interface and a `restaurantUsers` role named `"driver"` remain later extensions.

## GPS and Navigation

Live GPS tracking is not part of the MVP.

The driver schema does not store `lastKnownLocation` in the MVP.

Delivery-address and restaurant-location snapshots include coordinates so that route-planning, navigation and delivery-estimation modules can be added or improved without redesigning the driver and tour collections.

Navigation integration and route calculation are documented outside this file.

## Relations to Other Collections

```text
restaurants ||--o{ drivers
restaurants ||--o{ deliveryTours
locations ||--o{ deliveryTours
drivers ||--o{ deliveryTours
deliveryTours ||--o{ orders
orders ||--o{ deliveryAssignmentHistory
```

## Validation Rules

- a driver belongs to exactly one restaurant
- a tour belongs to exactly one restaurant and one restaurant location
- `locationId` must belong to the same restaurant as the tour
- `locationSnapshot` and delivery-address snapshots always include coordinates
- only active drivers with `dutyStatus: "on_duty"` can receive new tour assignments
- a driver can have at most one tour with status `"in_progress"` or `"returning"`
- a driver can have at most one tour with status `"planned"`
- a tour must contain at least one stop before it can start
- a tour must have a driver before it can start
- a tour starts only when all non-cancelled stop orders use order status `"ready"`
- starting a tour sets `loadingStatus: "closed"`
- stops cannot be added, removed or reordered after the tour has started
- `sequence` is unique within a tour
- an order cannot belong to multiple non-terminal tour stops at the same time
- pickup orders cannot be assigned to delivery tours
- a failed stop requires `failedAt` and `failureReason`
- a delivered stop requires `deliveredAt`
- a cancelled stop requires `cancelledAt`
- a completed tour requires `completedAt`
- a cancelled tour requires `cancelledAt`

## Indexes

```js
db.drivers.createIndex({ restaurantId: 1, isActive: 1 })

db.drivers.createIndex({
  restaurantId: 1,
  dutyStatus: 1,
  isActive: 1
})

db.deliveryTours.createIndex({
  restaurantId: 1,
  locationId: 1,
  status: 1
})

db.deliveryTours.createIndex({
  driverId: 1,
  status: 1
})

db.deliveryTours.createIndex({
  "stops.orderId": 1
})
```

The application additionally enforces:

```text
maximum one active tour per driver
maximum one planned tour per driver
```

These constraints must be protected against concurrent writes at database level during implementation.

## MVP Scope

The MVP includes:

```text
driver profiles per restaurant
manual driver duty status
manual tour creation and assignment
one active and one planned next tour per driver
multiple stops per tour
manual tour loading closure
immutable location and delivery-address snapshots
stop sequence storage
handover-duration storage
estimated arrival and return timestamps as externally calculated values
delivery failure handling
navigation integration outside this module
```

The MVP excludes:

```text
live GPS tracking
shift planning
driver accounts
driver mobile app
automatic driver assignment
route-planning algorithms inside Foodpilot
```

## Later Extensions

```text
live GPS tracking
automatic route optimization
automatic driver assignment
driver shifts
restaurantUsers role "driver"
driver mobile app
driver performance statistics
proof of delivery
delivery photos
customer signatures
push notifications
restaurant-independent driver pools
```

## Design Decisions Summary

- drivers and delivery tours use separate collections
- a tour represents a physical delivery run with one or more stops
- orders are assigned to tours during dispatch planning, not automatically when created
- a driver can have one active tour and one planned next tour
- the driver profile stores duty status but no duplicated active-tour reference
- the driver's operational position is derived from the active tour
- a planned tour can be closed manually when the driver must not take additional orders
- a running tour cannot receive additional orders
- each stop stores an immutable delivery-address snapshot including coordinates
- only delivery orders can be assigned to tours
- orders store `deliveryTourId` and assignment history but no duplicated `driverId`
- failed delivery attempts use the order status `delivery_failed`
- GPS, navigation and route-planning logic remain outside the MVP driver module

## Open Questions

None currently.
