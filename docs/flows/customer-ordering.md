# Customer Ordering Flow

## Purpose

This document describes the MVP customer ordering flow for Foodpilot.

The customer ordering flow is the public-facing process that starts when a customer opens a restaurant's Foodpilot ordering page from an existing restaurant website.

Foodpilot is not a marketplace and not a restaurant discovery platform. Customers usually reach the ordering flow through a restaurant-owned website, social media profile, QR code or direct ordering link.

The goal of the flow is simple:

```text
open ordering page
→ choose products
→ configure cart
→ choose pickup or delivery
→ enter customer details
→ choose payment
→ place order
→ receive confirmation
```

---

## Scope

The MVP customer ordering flow supports:

- public ordering pages for restaurants
- one active location per restaurant
- fixed restaurants and mobile food trucks
- menu categories
- menu items
- product options
- product labels
- product availability
- cart handling
- pickup orders
- delivery orders
- delivery address entry
- delivery radius validation
- guest checkout
- customer account checkout
- PayPal
- cash payment where allowed
- preorders where enabled
- fulfilment time estimates
- order confirmation
- registered-customer reviews after completed orders

The flow should be simple enough for customers to complete an order without needing to understand Foodpilot as a platform.

---

## Non Goals

The MVP customer ordering flow does not include:

- restaurant search
- marketplace browsing
- customer discovery feed
- loyalty program
- coupons and promotions
- live GPS tracking
- native mobile app requirement
- customer-facing real-time driver map
- multi-restaurant cart
- complex account area
- anonymous public reviews
- guest reviews

---

## Entry Points

Customers can enter the ordering flow through:

```text
restaurant website order button
direct ordering URL
social media profile link
QR code later
```

Example button label:

```text
Order food
```

Example German label:

```text
Essen bestellen
```

Example URL:

```text
https://foodpilot.org/pizza-example
```

A self-hosted deployment may use a different base URL.

---

## High-Level Flow

```text
Open ordering page
→ Load restaurant and active location
→ Check public page state
→ Show menu
→ Select product
→ Configure options
→ Add to cart
→ Open cart
→ Choose pickup or delivery
→ Enter customer data
→ Enter delivery address if required
→ Select fulfilment time
→ Select payment method
→ Review order summary
→ Place order
→ Complete payment if required
→ Show confirmation
```

---

## Public Page States

The ordering page can be visible even when checkout is not available.

Supported public states:

```text
setup_incomplete
closed
paused
open
```

### setup_incomplete

The restaurant ordering page exists, but required setup is missing.

Customer behavior:

- menu may be hidden or shown as unavailable
- checkout is disabled
- customer sees a clear message

Example message:

```text
Online ordering is not available yet.
```

### closed

The active location is currently outside opening hours.

Customer behavior:

- menu may remain visible
- checkout is disabled unless valid preorders are enabled
- customer sees opening information

Example message:

```text
We are currently closed.
```

### paused

The restaurant is inside opening hours, but order acceptance is manually paused.

Customer behavior:

- menu may remain visible
- checkout is disabled
- customer sees a temporary pause message

Example message:

```text
Online orders are currently paused.
```

### open

The restaurant can accept orders.

Customer behavior:

- menu is visible
- cart is available
- checkout is available when cart and fulfilment rules are valid

---

## Restaurant and Location Loading

When the ordering page opens, Foodpilot loads:

```text
restaurant by slug
active location
restaurant menu configuration
active menu categories
active or visible menu items
pickup and delivery settings
opening status
order acceptance status
preorder settings
payment availability
```

Checkout requires:

```text
restaurant is active
activeLocationId is set
active location exists
active location is enabled
active location can accept orders
at least one fulfilment type is enabled
at least one available product exists
```

---

## Menu Display

The menu is grouped by active categories.

The menu should show:

- category names
- product names
- product descriptions
- prices
- labels
- availability state
- product options where relevant
- customer-facing unavailable state where configured

### Product Labels

Labels can be used for information such as:

```text
vegan
vegetarian
spicy
gluten_free
lactose_free
halal
custom restaurant labels
```

Labels are flexible and restaurant-defined.

### Product Availability

Products can have these states:

```text
available
temporarily_unavailable
inactive
```

Customer-facing behavior depends on restaurant configuration:

```text
unavailableProductDisplayMode:
  hidden
  shown_as_unavailable
```

Inactive products are not part of the active menu.

Temporarily unavailable products may either be hidden or shown as unavailable.

---

## Product Detail Flow

When a customer selects a product, the product detail view opens.

The product detail should show:

- product name
- description
- price
- labels
- availability
- option groups
- selectable options
- quantity selector
- add-to-cart action

### Product Options

Product options may represent:

- size
- extras
- toppings
- sauces
- removals
- preparation choices

The backend must validate selected options and prices before order creation.

Frontend validation improves the user experience but is not authoritative.

---

## Cart Flow

The cart stores the customer's selected items before checkout.

The cart should show:

- selected products
- selected options
- item quantity
- item price
- subtotal
- delivery fee where applicable
- total price
- unavailable item warnings
- fulfilment type selection where appropriate

Cart actions:

```text
increase quantity
decrease quantity
remove item
edit item options
clear cart
continue to checkout
```

Before checkout, the backend must revalidate the cart against current menu, availability and pricing rules.

---

## Fulfilment Choice

The customer chooses between:

```text
pickup
delivery
```

Only available fulfilment types are shown.

### Pickup

Pickup can be selected when pickup is enabled for the active location.

Pickup does not require:

- delivery address
- delivery radius validation
- delivery tour
- driver

Pickup orders always use:

```text
deliveryTourId: null
```

### Delivery

Delivery can be selected when delivery is enabled for the active location.

Delivery requires:

- valid delivery address
- delivery radius validation when radius is configured
- delivery fee calculation
- driver and delivery availability strategy

Delivery orders may later be assigned to a delivery tour by the restaurant.

---

## Delivery Address Flow

For delivery orders, the customer enters a delivery address.

Required fields:

```text
street
house number
zip
city
```

Optional fields:

```text
floor
note
```

The backend must validate:

- address completeness
- geocoding or coordinate availability
- delivery radius when configured
- delivery availability

Foodpilot stores the delivery address as an immutable order snapshot after order creation.

---

## Customer Data Flow

Customers may order as:

```text
guest
registered customer
```

### Guest Checkout

Guest checkout is supported in the MVP.

Guest checkout should require only the information needed to complete the order.

For delivery orders:

```text
phone required
```

For pickup orders:

```text
phone optional
```

Restaurant/payment rules may require additional contact information for specific payment and fulfilment combinations.

### Registered Customer Checkout

Registered customers can log in during checkout.

Benefits:

- saved customer information
- saved addresses where supported
- order history later
- verified review eligibility after completed orders

Only registered customers can create reviews in the MVP.

---

## Fulfilment Time

The customer receives a fulfilment estimate before placing the order.

Pickup estimate means:

```text
expected ready-for-pickup time
```

Delivery estimate means:

```text
expected delivery arrival time
```

Estimation is documented separately in:

```text
docs/algorithms/delivery-estimation.md
```

The estimate is not exact real-time navigation.

Foodpilot may use external travel-time providers or a simple fallback estimate.

The final estimate is stored on the order in `fulfilmentEstimateHistory`.

---

## Preorders

Preorders allow customers to choose a later fulfilment time.

Preorders are only available when enabled by the restaurant.

The requested time must be valid for:

- active location
- opening hours
- opening hour exceptions
- timezone
- order acceptance state
- pickup or delivery availability
- delivery radius for delivery orders
- preorder settings
- preparation and delivery feasibility

In the MVP, preorders require online payment.

Cash payment is not offered for preorders.

If preorders are disabled, customers can only choose the next available fulfilment time.

---

## Payment Selection

The MVP supports:

```text
PayPal
cash
```

Only payment methods that are valid for the current order should be shown.

### PayPal

PayPal is used for online payment.

PayPal can be required for:

- online orders
- preorders
- restaurant-specific payment rules

The customer completes the PayPal flow before the order is treated as processable.

### Cash

Cash payment may be offered depending on restaurant configuration and risk rules.

Cash can be available for:

- pickup
- delivery

Cash may be restricted for guest pickup orders or customers without successful order history.

The backend decides whether cash is available for the current checkout.

---

## Order Summary

Before placing the order, the customer should see a clear summary.

The summary should include:

- restaurant name
- active location or pickup location
- order type
- fulfilment time estimate or selected preorder time
- customer information
- delivery address where relevant
- products and options
- subtotal
- delivery fee
- total amount
- payment method
- important notes

The customer confirms the order from this screen.

---

## Order Creation

When the customer places the order, the backend:

```text
validates restaurant and active location
validates opening and order acceptance state
validates fulfilment type
validates delivery address and radius where needed
validates cart items and options
validates prices
validates product availability
validates optional inventory
validates payment method
creates order number
creates order
creates payment record where needed
creates inventory reservation where needed
stores snapshots
stores fulfilment estimate
```

Snapshots include:

- location snapshot
- customer snapshot
- delivery address snapshot where relevant
- ordered item snapshots

This preserves historical correctness even if data changes later.

---

## Online Payment Flow

For online payment, the flow is:

```text
order created as awaiting_payment
→ payment attempt created
→ customer completes PayPal payment
→ backend verifies payment
→ order becomes processable
→ restaurant receives new order
```

If payment fails or expires, the order should not enter the normal restaurant preparation flow.

---

## Cash Payment Flow

For cash payment, the flow is:

```text
checkout validated
→ order created
→ restaurant receives new order
→ customer pays at pickup or delivery
```

Cash orders must still pass all restaurant, fulfilment, cart and availability validations.

---

## Order Confirmation

After successful order creation and payment handling, the customer sees an order confirmation.

The confirmation should show:

- order number
- order status
- payment status
- order type
- estimated pickup or delivery time
- restaurant name
- pickup address or delivery address
- ordered items
- total amount

For delivery orders, the confirmation should not promise live driver tracking in the MVP.

Example delivery message:

```text
Your order has been received. The restaurant will prepare your food and deliver it as soon as possible.
```

Example pickup message:

```text
Your order has been received. Please come to the restaurant at the estimated pickup time.
```

---

## Order Status Visibility

Customers may see simplified order status labels.

Internal status:

```text
awaiting_payment
new
accepted
preparing
ready
out_for_delivery
delivery_failed
delivered
picked_up
rejected
cancelled
expired
```

Customer-facing labels may be simpler:

| Internal Status | Customer Label |
|---|---|
| awaiting_payment | Waiting for payment |
| new | Order received |
| accepted | Accepted |
| preparing | Preparing |
| ready | Ready for pickup |
| out_for_delivery | Out for delivery |
| delivery_failed | Delivery issue |
| delivered | Delivered |
| picked_up | Picked up |
| rejected | Rejected |
| cancelled | Cancelled |
| expired | Expired |

For pickup orders, `ready` means ready for pickup.

For delivery orders, `ready` means ready for dispatch.

---

## Failed Delivery

Failed delivery attempts are not part of the normal checkout flow.

If a delivery fails after dispatch, the customer-facing state may become:

```text
Delivery issue
```

The restaurant can decide whether to retry delivery or cancel the order.

Foodpilot should keep the message clear and avoid exposing unnecessary internal operational details.

---

## Review Eligibility

Reviews are available only after successful order completion.

Eligible order statuses:

```text
delivered
picked_up
```

Review requirements:

```text
registered customer
order belongs to customer
order is completed successfully
order has no existing review
```

Guest orders cannot receive reviews in the MVP.

Reviews are stored separately in the `reviews` collection.

---

## Error and Validation Messages

The customer flow should show practical validation messages.

Examples:

```text
This restaurant is currently closed.
Online orders are currently paused.
This product is no longer available.
Your delivery address is outside the delivery area.
Please choose pickup or delivery.
Please enter a valid delivery address.
Please choose a payment method.
PayPal payment could not be completed.
Cash payment is not available for this order.
The selected preorder time is no longer available.
```

Messages should be clear and actionable.

---

## Empty States

### Empty Menu

```text
No products are available right now.
```

### Empty Category

```text
No products in this category are currently available.
```

### Empty Cart

```text
Your cart is empty.
```

### Closed Restaurant

```text
We are currently closed.
```

### Paused Orders

```text
Online orders are currently paused.
```

### Delivery Unavailable

```text
Delivery is currently not available.
```

### Pickup Unavailable

```text
Pickup is currently not available.
```

---

## MVP Screen List

The customer ordering flow should support the following screens.

### Public Ordering

```text
Restaurant ordering page
Menu category view
Product detail modal or page
Cart drawer or page
Fulfilment selection
Delivery address form
Customer details form
Preorder time selection
Payment selection
Order summary
PayPal redirect or payment step
Order confirmation
Order status view
Review form for eligible registered customers
```

### Authentication During Checkout

```text
Continue as guest
Login
Create customer account
Forgot password
```

Account creation must not block guest checkout unless a specific account-only feature is required.

---

## Accessibility and Usability

The customer ordering flow should be usable on mobile first.

Important UI requirements:

- clear product cards
- readable prices
- large touch targets
- simple cart access
- clear checkout steps
- visible total price
- clear error messages
- keyboard-accessible forms
- semantic form labels
- accessible buttons and links
- no hidden mandatory steps

The ordering flow should work well on common smartphones because many food orders happen on mobile devices.

---

## Later Extensions

Later versions may add:

- customer account dashboard
- order history page
- reorder feature
- coupons and discounts
- loyalty features
- saved favorite products
- guest review tokens
- live delivery tracking
- push notifications
- SMS notifications
- native app wrapper
- Apple Pay and Google Pay through payment provider
- additional payment providers
- delivery time slot capacity management
- QR code ordering entry points
- embeddable menu widget

---

## Summary

The Foodpilot customer ordering flow is a focused ordering process connected to a restaurant's existing website.

It should let customers:

```text
open the ordering page
choose products
configure options
choose pickup or delivery
enter required details
choose payment
place the order
receive confirmation
```

The MVP supports guest checkout, customer accounts, pickup, delivery, PayPal, cash payment where allowed, preorders and verified reviews for registered customers.

Foodpilot keeps the customer flow simple while the backend enforces all important rules for restaurant availability, product availability, delivery radius, payment, inventory and review eligibility.
