# Delivery Estimation Algorithm

## Purpose

This document describes how Foodpilot estimates pickup and delivery fulfilment times in the MVP.

The goal is to provide customers and restaurant operators with a realistic, understandable estimate before and after an order is placed. The estimate is a practical operational approximation. It is not exact real-time navigation and it does not promise live driver tracking.

Foodpilot stores fulfilment estimates on the order so later changes remain traceable.

---

## Scope

The MVP estimation logic supports:

- one restaurant business
- one active location
- fixed restaurants and mobile food trucks
- pickup orders
- delivery orders
- guest checkout and customer accounts
- PayPal and cash payments
- preorders
- drivers
- delivery tours with multiple stops
- external navigation handoff, for example to Google Maps

Before customers can order, the restaurant must have an `activeLocationId`. The active location must be enabled and open for order acceptance.

---

## Non Goals

The MVP does not include:

- live GPS driver tracking
- turn-by-turn navigation inside Foodpilot
- automatic traffic-aware route updates
- automatic route optimisation
- automatic stop reordering
- a required native driver app
- guaranteed delivery times
- failed delivery attempts as a normal input for the initial estimate

Foodpilot can prepare orders, drivers, tours and stop lists. Navigation and traffic-aware route guidance are delegated to an external navigation provider when needed.

---

## Inputs

The estimation uses the following inputs where available.

### Restaurant

```js
activeLocationId: ObjectId | null
orderConfiguration: Object
```

### Location

```js
restaurantId: ObjectId
displayAddress: String
geoLocation: {
  type: "Point",
  coordinates: [Number, Number]
}
openingHours: Array
openingHourExceptions: Array
timezone: String
orderAcceptanceStatus: "open" | "paused"
preorderSettings: Object
deliverySettings: {
  deliveryEnabled: Boolean
  pickupEnabled: Boolean
  deliveryRadiusKm: Number | null
  minimumOrderValueInCents: Number
  deliveryFeeInCents: Number
}
isEnabled: Boolean
```

### Orders

```js
restaurantId: ObjectId
locationId: ObjectId
customerId: ObjectId | null
deliveryTourId: ObjectId | null
orderType: "delivery" | "pickup"
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
  | "expired"
locationSnapshot: Object
deliveryAddressSnapshot: Object | null
items: Array
deliveryAssignmentHistory: Array
fulfilmentEstimateHistory: Array
```

Pickup orders always use:

```js
deliveryTourId: null
```

Delivery orders can reference a delivery tour.

### Drivers

```js
restaurantId: ObjectId
name: String
phone: String | null
vehicleType: "bike" | "car" | "scooter" | "walking" | "other"
dutyStatus: "off_duty" | "on_duty" | "paused"
isActive: Boolean
currentTourId: ObjectId | null
```

Only active drivers with `dutyStatus: "on_duty"` are considered for delivery planning.

### Delivery Tours

```js
restaurantId: ObjectId
locationId: ObjectId
driverId: ObjectId | null
status:
  "planned"
  | "in_progress"
  | "returning"
  | "completed"
  | "cancelled"
loadingStatus: "open" | "closed"
stops: Array
estimatedDepartureAt: Date | null
estimatedReturnAt: Date | null
startedAt: Date | null
returnStartedAt: Date | null
completedAt: Date | null
cancelledAt: Date | null
```

A delivery tour is one physical delivery run from the active restaurant location to one or more customers and back to the restaurant location.

---

## Pickup Estimation

Pickup estimation depends on kitchen preparation, current kitchen load and a safety buffer.

```text
foodReadyIn =
  preparationTime
  + kitchenLoadBuffer

estimatedPickupTime =
  foodReadyIn
  + safetyBuffer
```

The pickup estimate represents the earliest realistic time at which the order can be ready for the customer.

Pickup orders are not assigned to delivery tours.

---

## Delivery Estimation

Delivery estimation depends on food readiness, driver availability, delivery travel time and a safety buffer.

Kitchen work and driver availability can happen in parallel. The estimate must therefore not simply add kitchen time and driver waiting time together.

```text
foodReadyIn =
  preparationTime
  + kitchenLoadBuffer

dispatchDelay =
  max(foodReadyIn, driverAvailabilityDelay)

estimatedDeliveryTime =
  dispatchDelay
  + deliveryTravelTime
  + safetyBuffer
```

The order can be dispatched when both conditions are true:

- the food is ready
- a suitable driver or delivery tour is available

This avoids overly pessimistic delivery estimates when the kitchen can prepare the order while a driver is still returning.

---

## Preparation Time

`preparationTime` is the expected time required by the kitchen to prepare the order.

The MVP may derive this value from:

- restaurant-level default preparation time
- menu item preparation hints
- order size
- manually configured preparation buffers

If no detailed item-level calculation exists, the restaurant-level default preparation time is used.

---

## Current Order Load

`kitchenLoadBuffer` accounts only for orders that still create kitchen work.

Kitchen load orders use these statuses:

```text
kitchenLoadOrders:
  new
  accepted
  preparing
```

These statuses no longer count as kitchen load:

```text
ready
out_for_delivery
```

Reason:

- `ready` may still affect handoff or driver planning, but it no longer blocks kitchen preparation.
- `out_for_delivery` no longer affects the kitchen.

For the MVP, Foodpilot may use a simple load rule:

```text
kitchenLoadBuffer = numberOfKitchenLoadOrders * minutesPerKitchenOrder
```

Example:

```text
minutesPerKitchenOrder = 5
numberOfKitchenLoadOrders = 4
kitchenLoadBuffer = 20 minutes
```

Foodpilot can also track delivery-side load separately.

```text
deliveryLoad:
  ready
  assigned_to_tour
  out_for_delivery
```

`assigned_to_tour` is a conceptual delivery planning state. It can be derived from `deliveryTourId` and the related delivery tour status. It is not required to be a separate order status in the MVP.

---

## Driver Availability

`driverAvailabilityDelay` describes when a suitable driver can realistically dispatch the order.

Driver availability is derived primarily from the driver's current or next assigned delivery tour.

A driver is usable for delivery planning when:

```js
isActive: true
dutyStatus: "on_duty"
```

### No Active On-Duty Driver

If delivery is enabled but no active on-duty driver exists, Foodpilot should not promise a normal delivery estimate.

The order flow should treat delivery as currently unavailable or require manual restaurant handling, depending on restaurant configuration.

### Driver Without Current or Planned Tour

If a driver is active, on duty and has no current or planned delivery tour, the driver can be used immediately.

```text
driverAvailabilityDelay = 0
```

### Driver With Current Tour

If a driver is assigned to a current tour, the tour's `estimatedReturnAt` is the central input for driver availability.

```text
driverAvailabilityDelay = estimatedReturnAt - now
```

Relevant current tour statuses are:

```text
in_progress
returning
```

If `estimatedReturnAt` is in the past, the delay is treated as `0`.

If `estimatedReturnAt` is missing, Foodpilot uses a conservative fallback buffer.

### Driver With Planned Next Tour

A driver can have one current tour and one planned next tour.

If a new delivery can still be added to an open planned tour, the planned tour can be used for the estimate.

A planned tour is usable when:

```js
status: "planned"
loadingStatus: "open"
```

If the planned tour is already closed, the new order waits for the next available driver or tour slot.

### Multiple Drivers

When multiple drivers are active and on duty, Foodpilot uses the earliest realistic availability.

If at least one driver has no current or planned tour:

```text
driverAvailabilityDelay = 0
```

If all drivers are currently on active or returning tours:

```text
driverAvailabilityDelay =
  earliest estimatedReturnAt among active or returning tours
  - now
```

If one or more required `estimatedReturnAt` values are missing, Foodpilot uses a conservative fallback buffer for those drivers.

---

## Delivery Tours

Foodpilot supports delivery tours in the MVP.

A delivery tour means:

- a driver starts at the active restaurant location
- the driver takes one or more delivery orders
- the driver follows a planned stop list
- the driver returns to the restaurant location afterwards

Foodpilot can group delivery orders into a tour and preserve a planned stop list.

The MVP does not automatically optimise stop order.

The restaurant operator or driver may decide the stop order manually. The driver may open the tour stops in an external navigation app such as Google Maps.

Tour planning rules:

- a tour belongs to one restaurant and one location
- a tour starts at the active restaurant location
- a tour contains one or more customer stops
- a tour returns to the restaurant location
- orders can be added while `loadingStatus: "open"`
- orders should not be added when `loadingStatus: "closed"`
- completed and cancelled tours are ignored for new estimates

---

## Travel Time

`deliveryTravelTime` is the expected travel time for the delivery.

For a single-order delivery, it represents the expected travel time from the active restaurant location to the customer address.

For a tour delivery, it represents the expected customer-facing travel impact of adding the order to the planned stop list.

The MVP can use:

- an external travel-time estimate, if available
- a rough fallback based on distance and vehicle type

The fallback is intentionally simple.

```text
deliveryTravelTime =
  distanceEstimate
  adjusted by vehicleType
  + fixed handoff buffer
```

The fallback does not account for live traffic, road closures, parking, exact route quality or route changes during the drive.

---

## External Navigation Handoff

Foodpilot does not provide turn-by-turn navigation.

For driver navigation, Foodpilot may generate external navigation links for Google Maps or another provider.

Foodpilot is responsible for:

- order handling
- preparation estimates
- driver assignment
- delivery tours
- planned stop lists
- estimate history
- status updates

The external navigation provider is responsible for:

- turn-by-turn navigation
- traffic-aware route guidance
- route adjustments during the drive

Using Google Maps or another navigation app does not automatically give Foodpilot live driver location data or updated remaining travel times.

Live GPS tracking is not part of the MVP. It can be added later as an optional extension.

---

## Buffer Time

`safetyBuffer` protects the estimate from being too optimistic.

It can account for:

- packing time
- handoff time
- payment handling
- small kitchen delays
- short routing inaccuracies
- mobile food truck location handling

The MVP should use a simple fixed buffer, for example 5 to 10 minutes, configured at restaurant or system level.

---

## Preorders

Preorders use the same estimation logic, but the result is anchored to the requested fulfilment time.

For pickup preorders, Foodpilot estimates when preparation must start so the order is ready at the requested pickup time.

For delivery preorders, Foodpilot estimates when preparation and dispatch planning must start so the order can arrive around the requested delivery time.

The requested preorder time must be checked against:

- active location
- opening hours
- opening hour exceptions
- timezone
- order acceptance status
- preorder settings
- pickup or delivery availability
- delivery radius for delivery orders
- driver availability for delivery orders

A preorder should not be accepted when the requested fulfilment time cannot realistically be met.

---

## Estimate History

Foodpilot stores fulfilment estimates on the order.

```js
fulfilmentEstimateHistory: Array
```

A new history entry should be added when the estimate is created or meaningfully changed.

Example reasons:

```text
initial_estimate
restaurant_accepted
kitchen_delay
driver_delay
tour_updated
manual_update
```

Each entry should include enough data to understand how the estimate was created.

Recommended structure:

```js
{
  estimatedReadyAt: Date | null,
  estimatedPickupAt: Date | null,
  estimatedDeliveryAt: Date | null,

  preparationTimeMinutes: Number,
  kitchenLoadBufferMinutes: Number,
  foodReadyInMinutes: Number,

  driverAvailabilityDelayMinutes: Number | null,
  dispatchDelayMinutes: Number | null,
  deliveryTravelTimeMinutes: Number | null,
  safetyBufferMinutes: Number,

  source: "system" | "manual" | "external_api",
  reason: String,
  createdAt: Date
}
```

Pickup orders use `estimatedPickupAt` and usually keep delivery-specific fields as `null`.

Delivery orders use `estimatedDeliveryAt` and should also store `estimatedReadyAt` when available.

---

## MVP Algorithm

### Pickup

```text
1. Load restaurant and active location.
2. Validate that the active location can accept pickup orders.
3. Count kitchen load orders for the same restaurant and location.
4. Calculate preparation time.
5. Calculate kitchen load buffer.
6. Calculate foodReadyIn.
7. Add safety buffer.
8. Store the estimate in fulfilmentEstimateHistory.
9. Return estimated pickup time.
```

Formula:

```text
foodReadyIn =
  preparationTime
  + kitchenLoadBuffer

estimatedPickupTime =
  now
  + foodReadyIn
  + safetyBuffer
```

### Delivery

```text
1. Load restaurant and active location.
2. Validate that the active location can accept delivery orders.
3. Validate delivery address and delivery radius.
4. Count kitchen load orders for the same restaurant and location.
5. Calculate preparation time.
6. Calculate kitchen load buffer.
7. Calculate foodReadyIn.
8. Find active on-duty drivers.
9. Check current and planned delivery tours.
10. Calculate driverAvailabilityDelay from tour availability.
11. Calculate dispatchDelay as max(foodReadyIn, driverAvailabilityDelay).
12. Calculate deliveryTravelTime.
13. Add safetyBuffer.
14. Store the estimate in fulfilmentEstimateHistory.
15. Return estimated delivery time.
```

Formula:

```text
foodReadyIn =
  preparationTime
  + kitchenLoadBuffer

dispatchDelay =
  max(foodReadyIn, driverAvailabilityDelay)

estimatedDeliveryTime =
  now
  + dispatchDelay
  + deliveryTravelTime
  + safetyBuffer
```

---

## Example

Assume the following values:

```text
preparationTime = 20 minutes
numberOfKitchenLoadOrders = 2
minutesPerKitchenOrder = 5 minutes
kitchenLoadBuffer = 10 minutes
foodReadyIn = 30 minutes

driverAvailabilityDelay = 20 minutes
deliveryTravelTime = 12 minutes
safetyBuffer = 5 minutes
```

Delivery estimate:

```text
dispatchDelay = max(30, 20) = 30 minutes

estimatedDeliveryTime =
  30 + 12 + 5
  = 47 minutes
```

If the order is placed at 18:00, the estimated delivery time is 18:47.

The estimate is not:

```text
30 + 20 + 12 + 5 = 67 minutes
```

The kitchen can prepare the food while the driver is returning. The order can leave once both the food and the driver are ready.

Pickup estimate with the same kitchen values:

```text
estimatedPickupTime =
  30 + 5
  = 35 minutes
```

If the order is placed at 18:00, the estimated pickup time is 18:35.

---

## Validation Rules

Foodpilot must validate the following before returning or storing an estimate:

- `restaurants.activeLocationId` is set
- the active location exists
- the active location belongs to the restaurant
- the active location has `isEnabled: true`
- the active location has `orderAcceptanceStatus: "open"`
- pickup is enabled for pickup orders
- delivery is enabled for delivery orders
- delivery orders include a valid delivery address snapshot
- delivery orders are within the configured delivery radius when a radius is configured
- delivery orders require at least one active on-duty driver or a valid manual delivery handling strategy
- pickup orders always use `deliveryTourId: null`
- delivery orders may only reference tours belonging to the same restaurant and location
- delivery tours used for new assignments must not be completed or cancelled
- orders may only be added to tours with `loadingStatus: "open"`
- preorder fulfilment times must be inside valid ordering and fulfilment windows

---

## Failed Delivery Attempts

Failed delivery attempts are not part of the normal initial estimate.

They are handled later as an operational consequence of an already running delivery process.

A failed attempt may affect:

- order status
- customer communication
- delivery assignment history
- later manual estimate updates
- restaurant support workflows

Failed delivery attempts should not make the initial MVP estimation algorithm more complex.

---

## Later Extensions

The MVP algorithm is intentionally simple. Later versions may add:

- live GPS driver tracking
- real-time traffic-aware estimates
- deeper external map provider integration
- automatic route optimisation
- automatic stop ordering
- customer-facing live delivery tracking
- driver app integration
- preparation time prediction based on historical data
- separate kitchen capacity models
- weather-aware delivery buffers
- vehicle-specific routing rules
- driver workload balancing

These extensions should build on the same estimate history model instead of replacing it.

---

## Summary

Foodpilot estimates pickup and delivery times with a simple, transparent MVP algorithm.

Pickup estimates are based on preparation time, current kitchen load and a safety buffer.

Delivery estimates are based on food readiness, driver availability, delivery travel time and a safety buffer. Kitchen work and driver availability are treated as parallel processes, so the delivery estimate uses `max(foodReadyIn, driverAvailabilityDelay)` instead of adding both values together.

Foodpilot supports delivery tours and planned stop lists in the MVP, but it does not provide live GPS tracking, turn-by-turn navigation or automatic route optimisation.

External navigation tools such as Google Maps can be used by drivers for route guidance. This does not automatically provide Foodpilot with live location data or updated remaining travel times.

Each estimate is stored on the order through `fulfilmentEstimateHistory`, making later changes traceable and understandable.
