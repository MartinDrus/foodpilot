# Admin Dashboard

## Purpose

This document describes the MVP restaurant admin dashboard for Foodpilot.

The dashboard is the operational control center for restaurants, takeaway businesses and food trucks. It helps restaurant operators set up their business, manage daily orders, maintain the menu, control delivery and pickup, manage drivers, review payments, view customers, handle reviews and integrate the public ordering page into an existing website.

This document is intended as a practical foundation for UI design, Figma wireframes and later frontend implementation.

Foodpilot is not a marketplace, not a delivery network and not a restaurant website builder. The dashboard supports the restaurant's own ordering and fulfilment workflow.

---

## Scope

The MVP dashboard supports:

- restaurant setup and onboarding status
- one active location
- fixed restaurant locations and mobile food truck operating points
- opening hours and opening hour exceptions
- order acceptance control
- pickup settings
- delivery settings
- preorder settings
- PayPal and cash payment settings
- menu categories
- menu items
- product options
- product labels
- product availability
- optional inventory tracking
- order management
- delivery tour management
- driver management
- customer overview
- verified review overview
- website integration through an ordering link and button snippets

The dashboard should be understandable for non-technical restaurant operators.

---

## Non Goals

The MVP dashboard does not include:

- marketplace management
- restaurant discovery profile management
- full website builder tools
- CMS features for restaurant websites
- native driver app management
- live GPS tracking
- complex internal route optimisation
- automatic stop reordering
- advanced analytics
- full accounting software features
- multi-restaurant marketplace administration

---

## Dashboard Principles

The dashboard should not feel like a database administration tool.

It should clearly answer the operator's most important daily questions:

```text
Can I accept orders right now?
Is my ordering page ready?
What still needs to be set up?
Are there new orders?
Which orders need action?
Are products unavailable or sold out?
Are drivers available?
Are delivery tours planned or active?
Are payments working?
How do I add the Foodpilot button to my website?
```

The dashboard should be:

- calm
- practical
- action-oriented
- understandable without technical knowledge
- clear about setup problems
- clear about operational problems
- focused on daily restaurant work

Every warning should link directly to the place where the operator can fix the issue.

---

## Main Navigation

The MVP dashboard uses the following main navigation:

```text
Dashboard
Orders
Menu
Locations
Delivery
Drivers
Customers
Reviews
Payments
Settings
Integration
```

### Navigation Purpose

| Section | Purpose |
|---|---|
| Dashboard | Operational overview and setup status |
| Orders | Incoming and historical orders |
| Menu | Categories, products, options, labels and availability |
| Locations | Active location, saved locations, opening hours and order acceptance |
| Delivery | Delivery settings and delivery tours |
| Drivers | Driver records and duty status |
| Customers | Customer overview and order statistics |
| Reviews | Verified customer reviews |
| Payments | PayPal, cash settings and payment records |
| Settings | Restaurant profile, branding, legal and configuration |
| Integration | Public ordering URL and website button snippets |

---

## Dashboard Home

The dashboard home is the first screen after login.

It should combine setup guidance and daily operations.

### Main Areas

The dashboard home should show:

- setup checklist
- ordering page readiness
- order acceptance status
- active location
- opening status
- today's orders
- open orders
- delivery status
- driver status
- payment readiness
- integration status
- warnings
- quick actions

### Suggested Layout

```text
Top bar
→ restaurant name
→ active location
→ current open/closed/paused state
→ quick preview link

Main content
→ setup checklist or readiness card
→ today's order overview
→ open orders requiring action
→ delivery and driver status
→ warnings

Side panel
→ quick actions
→ ordering page link
→ integration status
```

### Quick Actions

The dashboard should provide direct actions for common tasks:

```text
Pause orders
Accept orders
Add menu item
Create delivery tour
Copy ordering link
Open ordering page preview
Configure opening hours
Configure payment
```

Actions should be shown only when they make sense.

For example, `Create delivery tour` should only be prominent when delivery is enabled and there are delivery orders that can be planned.

---

## Setup Checklist

The setup checklist helps a new operator move from account creation to a working ordering page.

### Required Before Checkout

The ordering page can accept checkout only when these requirements are met:

```text
Restaurant profile exists
Active location configured
Opening hours configured
Pickup or delivery enabled
Payment method configured
At least one active category exists
At least one available menu item exists
Ordering page slug exists
Website integration available
```

### Required When Delivery Is Enabled

```text
Delivery radius configured
Delivery fee configured
Delivery address validation available
At least one active driver or valid delivery strategy available
```

### Required When Preorders Are Enabled

```text
Online payment configured
Preorder settings valid
```

### Checklist Behavior

Each checklist item should have:

- status
- short explanation
- direct action button
- link to the relevant dashboard section

Example:

```text
Opening hours are missing.
Add opening hours so Foodpilot knows when customers can order.
[Configure opening hours]
```

---

## Readiness States

The dashboard and public ordering page use clear readiness states.

```text
setup_incomplete
closed
paused
open
```

### setup_incomplete

The restaurant exists, but required setup is missing.

Checkout is disabled.

The dashboard should show which setup steps are missing.

### closed

The active location is outside opening hours.

Checkout is disabled unless preorder rules allow a valid future fulfilment time.

### paused

The location is inside opening hours, but order acceptance has been manually paused.

Checkout is disabled.

This is used when the restaurant is overloaded, temporarily unavailable or unable to accept new orders.

### open

The active location is ready and customers can place orders.

---

## Orders

The Orders section is the main operational workflow for incoming and historical orders.

### Order List

The order list should show:

- order number
- order type
- status
- payment status
- customer name
- pickup or delivery time estimate
- total amount
- created time
- assigned delivery tour where relevant
- warning indicators

### Filters

Useful filters:

```text
All
New
Accepted
Preparing
Ready
Delivery
Pickup
Out for delivery
Delivery failed
Completed
Cancelled
Payment pending
```

### Order Statuses

The dashboard supports these order statuses:

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

### Order Detail

The order detail should show:

- order number
- customer snapshot
- delivery address snapshot where relevant
- ordered items and options
- item prices and total
- payment status
- payment method
- fulfilment estimate
- status history
- delivery assignment history where relevant
- internal notes where supported
- available actions

### Common Order Actions

Depending on status and order type, actions include:

```text
Accept order
Reject order
Start preparing
Mark as ready
Mark as picked up
Assign to delivery tour
Mark out for delivery
Mark as delivered
Mark delivery failed
Cancel order
Update estimate manually
```

### Delivery Order Actions

Delivery orders may be assigned to a delivery tour.

Orders do not store `driverId` directly.

Driver assignment is resolved through:

```text
order → deliveryTour → driver
```

If delivery fails, the order can move to:

```text
delivery_failed
```

From there, the restaurant can either plan a retry through another tour or cancel the order if no retry is planned.

### Pickup Order Actions

Pickup orders never use a delivery tour.

```text
deliveryTourId: null
```

The main pickup actions are:

```text
Accept
Preparing
Ready
Picked up
Cancel
```

---

## Menu

The Menu section manages the public menu shown to customers.

It should be easy to update during daily operation.

### Menu Areas

The Menu section contains:

- categories
- menu items
- product detail editor
- options
- labels
- availability
- optional inventory tracking

### Categories

Category fields:

```text
name
slug
sort order
active state
```

Category actions:

```text
Create category
Edit category
Reorder categories
Deactivate category
```

### Menu Items

Menu item fields:

```text
name
description
price
category
labels
options
availability status
unavailable reason
optional inventory tracking
```

### Product Availability

Supported availability states:

```text
available
temporarily_unavailable
inactive
```

Meaning:

| State | Meaning |
|---|---|
| available | Product can be ordered |
| temporarily_unavailable | Product is currently unavailable but remains part of the menu |
| inactive | Product is not currently part of the active menu |

Unavailable products remain unavailable until the operator marks them available again.

### Unavailable Product Display

The restaurant-level setting controls customer-facing display:

```text
unavailableProductDisplayMode:
  hidden
  shown_as_unavailable
```

The UI should explain:

```text
Choose whether unavailable products disappear from the menu or remain visible as unavailable.
```

### Inventory Tracking

Inventory tracking is optional per menu item.

When enabled, the dashboard can show:

- current stock
- low stock warning
- sold out state
- reserved stock where relevant

The MVP should not force inventory tracking for all products.

---

## Locations

The Locations section manages the active operational origin for orders.

### Location Concepts

A location can represent:

- a fixed restaurant address
- a takeaway counter
- a food truck operating point

A restaurant may save multiple locations, but the MVP uses one active location for customer ordering.

### Location List

The location list should show:

- location name
- display address
- enabled state
- active state
- order acceptance status
- opening status

### Location Detail

The location detail should manage:

- location name
- display address
- structured address
- coordinates
- timezone
- enabled state
- active location selection
- opening hours
- opening hour exceptions
- order acceptance status
- preorder settings
- delivery and pickup settings

### Active Location

The active location determines where orders start.

Checkout requires:

```text
active location exists
active location belongs to the restaurant
active location is enabled
active location has valid coordinates
```

### Order Acceptance

Order acceptance is controlled separately from opening hours.

```text
orderAcceptanceStatus: open | paused
```

Opening hours define the regular schedule.

Order acceptance controls the current operational state.

---

## Delivery

The Delivery section manages delivery configuration and delivery tours.

### Delivery Settings

Delivery settings include:

```text
delivery enabled
pickup enabled
delivery radius
minimum order value
delivery fee
```

Delivery can be disabled while pickup remains enabled.

Pickup can be disabled while delivery remains enabled.

At least one fulfilment type must be enabled before checkout can be active.

### Delivery Radius

The delivery radius defines where customers can request delivery.

When a radius is configured, the customer delivery address must be inside the delivery area.

### Delivery Tours

Delivery tours are used for self-delivery operations.

A tour:

- starts from the active location
- contains one or more customer stops
- may be assigned to a driver
- may be planned before it starts
- can be open or closed for adding orders
- can be completed or cancelled

### Tour List

The tour list should be grouped by:

```text
Planned
In progress
Returning
Completed
Cancelled
```

### Tour Detail

The tour detail should show:

- tour status
- loading status
- assigned driver
- active location snapshot
- stops
- stop sequence
- order references
- delivery addresses
- estimated arrival times
- estimated return time
- route handoff action
- status actions

### Tour Actions

Tour actions include:

```text
Create tour
Assign driver
Add order to tour
Remove order from tour
Close loading
Start tour
Open route in map provider
Mark stop delivered
Mark stop failed
Start return
Complete tour
Cancel tour
```

### Route Handoff

Foodpilot does not perform complex internal route optimisation in the MVP.

The dashboard can prepare the list of stops and pass them to an external map or navigation provider.

Possible providers include:

```text
Google Maps
Mapbox
HERE
```

The MVP does not include live GPS tracking.

---

## Drivers

The Drivers section manages restaurant-owned delivery drivers.

Drivers do not need their own login in the MVP.

### Driver List

The driver list should show:

- name
- phone
- vehicle type
- duty status
- active state
- current or next tour where relevant

### Driver Fields

```text
name
phone
vehicleType
dutyStatus
isActive
```

### Vehicle Types

```text
bike
car
scooter
walking
other
```

### Duty Status

```text
off_duty
on_duty
paused
```

### Driver Actions

```text
Create driver
Edit driver
Set on duty
Set off duty
Pause driver
Activate driver
Deactivate driver
Assign to tour
```

Driver availability is derived from:

```text
isActive
dutyStatus
active tours
planned tours
```

---

## Customers

The Customers section gives the restaurant an overview of customer records and ordering history.

### Customer List

The customer list should show:

- display name
- email
- phone where available
- order count
- total spent
- last order date
- account status

### Customer Detail

The customer detail may show:

- customer information
- saved addresses where available
- order history
- successful order count
- review history where relevant

The MVP should avoid exposing unnecessary personal data.

Customer deletion and anonymisation rules are handled according to the customer data model and legal retention requirements.

---

## Reviews

The Reviews section shows verified customer reviews.

### Review Rules

Reviews in the MVP follow these rules:

- only registered customers can create reviews
- guest orders cannot receive reviews
- only completed orders can receive reviews
- eligible statuses are `delivered` and `picked_up`
- each order can receive at most one review
- reviews are published by default
- hidden and soft-deleted reviews are excluded from public rating statistics

### Review List

The review list should show:

- customer display name
- overall rating
- delivery rating where relevant
- comment
- item ratings where available
- related order
- published date
- moderation status

### Restaurant Permissions

Restaurants can view reviews.

Restaurants cannot hide negative reviews only because they are negative.

Platform moderation can hide reviews that violate rules, such as spam, abuse or personal data exposure.

---

## Payments

The Payments section manages payment readiness and payment records.

### Payment Methods

The MVP supports:

```text
PayPal
cash
```

### Payment Readiness

The dashboard should show:

```text
PayPal configured
Cash enabled
Online payment ready
Preorders available
```

Preorders require online payment.

If PayPal is not configured, preorder activation should be blocked.

### Payment History

Payment records should show:

- order number
- payment method
- payment type
- amount
- status
- provider reference where available
- created date

### Cash Restrictions

Cash payment may be restricted for risk control.

For example, pickup with cash can be restricted for guests or customers without successful order history.

These rules should be enforced by the backend and shown clearly in settings.

---

## Settings

The Settings section contains configuration that does not belong to daily operations.

### Settings Areas

```text
Restaurant profile
Branding
Legal information
Order configuration
Menu configuration
User management
Payment rules
Notifications later
```

### Restaurant Profile

Fields:

```text
restaurant name
slug
contact email
contact phone
business type
currency
```

### Branding

Branding may include:

```text
logo
brand color
ordering page appearance
```

The MVP should keep branding simple.

### Legal Information

Legal information may include:

```text
business name
legal address
tax information where needed
imprint information
privacy information
```

### User Management

Restaurant users are separate from customers.

Supported roles:

```text
main_admin
operator
```

---

## Integration

The Integration section is essential for Foodpilot.

Foodpilot is meant to be connected to existing restaurant websites through a simple ordering link or button.

The Integration section should provide:

- public ordering URL
- ordering page preview
- copy plain link
- copy HTML button snippet
- button preview
- button variant selector
- button label selector
- language selector
- same-tab or new-tab option
- basic setup instructions
- integration readiness status

### Public Ordering URL

Example:

```text
https://foodpilot.org/pizza-example
```

A self-hosted deployment may use a different base URL.

### Plain Link

The simplest integration is a normal link.

```html
<a href="https://foodpilot.org/pizza-example">Order food</a>
```

### Instructions

The dashboard should provide short instructions for:

- static HTML websites
- WordPress
- common website builders
- social media profile links

The MVP should not require JavaScript snippets for basic integration.

---

## Website Button Variants

Foodpilot should provide ready-to-use button variants, similar to payment provider buttons.

All MVP snippets should use simple `<a>` links and should not depend on React, Vue or another frontend framework.

The operator can choose:

```text
Primary Button
Outline Button
Compact Button
Full Width Button
Branded Foodpilot Button
Plain Text Link
```

### Primary Button

Use case:

Main call-to-action on the restaurant website.

Visual direction:

```text
solid background
high contrast
rounded corners
large enough for touch interaction
```

Example labels:

```text
Order food
Essen bestellen
Order online
Jetzt bestellen
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-primary" href="https://foodpilot.org/pizza-example">
  Order food
</a>
```

---

### Outline Button

Use case:

Websites that already have a strong visual style and need a lighter integration.

Visual direction:

```text
transparent background
visible border
clear text
rounded corners
```

Example label:

```text
Order online
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-outline" href="https://foodpilot.org/pizza-example">
  Order online
</a>
```

---

### Compact Button

Use case:

Header navigation, mobile menus or small spaces.

Visual direction:

```text
compact height
short label
reduced padding
clear tap target
```

Example label:

```text
Order
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-compact" href="https://foodpilot.org/pizza-example">
  Order
</a>
```

---

### Full Width Button

Use case:

Mobile websites, landing pages or prominent ordering sections.

Visual direction:

```text
full container width
large touch target
strong call-to-action
```

Example label:

```text
Order food now
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-full" href="https://foodpilot.org/pizza-example">
  Order food now
</a>
```

---

### Branded Foodpilot Button

Use case:

Restaurants that want customers to recognise Foodpilot as the ordering provider.

Visual direction:

```text
Foodpilot wordmark or label
consistent brand color
clear ordering text
```

Example labels:

```text
Order with Foodpilot
Bestellen mit Foodpilot
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-branded" href="https://foodpilot.org/pizza-example">
  Order with Foodpilot
</a>
```

---

### Plain Text Link

Use case:

Footer links, simple websites, social media profiles or text-based sections.

Example labels:

```text
Order food
Essen bestellen
```

Example HTML:

```html
<a href="https://foodpilot.org/pizza-example">Order food</a>
```

---

## Button Configuration

The Integration section should allow the operator to configure:

```text
button label
language
button variant
button size
open in same tab or new tab
optional tracking parameter
```

The MVP should avoid a complex visual editor.

A small set of reliable variants is easier to understand and safer to embed.

---

## Empty States

Every dashboard area should have useful empty states.

### Dashboard

```text
Your setup is not complete yet.
Finish the checklist to activate your ordering page.
```

### Orders

```text
No orders yet.
Once customers place orders, they will appear here.
```

### Menu

```text
No menu items yet.
Create your first category and product to start accepting orders.
```

### Locations

```text
No active location configured.
Add your restaurant address or food truck operating point.
```

### Delivery

```text
Delivery is not enabled.
Enable delivery and configure your delivery area if you want to offer delivery orders.
```

### Drivers

```text
No drivers yet.
Add drivers if you deliver with your own team.
```

### Customers

```text
No customers yet.
Customer records will appear after orders are placed.
```

### Reviews

```text
No reviews yet.
Registered customers can leave reviews after completed orders.
```

### Payments

```text
Online payment is not configured yet.
Connect PayPal to accept online payments and enable preorders.
```

### Integration

```text
Your ordering link is ready.
Copy a button snippet and add it to your existing website.
```

---

## Warnings

Warnings should be practical and directly actionable.

Typical warnings:

```text
No active location selected.
Opening hours are missing.
Pickup and delivery are both disabled.
No available menu items exist.
PayPal is required before preorders can be enabled.
Delivery is enabled but no delivery radius is configured.
Delivery is enabled but no active driver is available.
Order acceptance is currently paused.
Website integration has not been copied yet.
```

Each warning should include a direct action.

Example:

```text
Delivery is enabled but no active driver is available.
[Add driver] [Open driver settings]
```

---

## MVP Screen List

The MVP dashboard should support the following screens.

### Authentication

```text
Restaurant signup
Restaurant login
Password reset
```

### Onboarding

```text
Welcome
Create restaurant profile
Create first location
Configure opening hours
Configure pickup and delivery
Configure payment
Create first menu category
Create first menu item
Website integration
Onboarding complete
```

### Dashboard

```text
Dashboard home
Setup checklist
Readiness warnings
Ordering page preview
```

### Orders

```text
Order list
Order detail
Payment status view
Delivery assignment view
Cancellation flow
Failed delivery flow
```

### Menu

```text
Category list
Category editor
Menu item list
Menu item editor
Product options editor
Availability controls
Inventory controls
```

### Locations

```text
Location list
Location editor
Opening hours editor
Opening hour exceptions editor
Active location selector
Order acceptance control
```

### Delivery

```text
Delivery settings
Delivery tour list
Delivery tour detail
Create delivery tour
Assign orders to tour
Route handoff view
```

### Drivers

```text
Driver list
Driver editor
Duty status controls
```

### Customers

```text
Customer list
Customer detail
Customer order history
```

### Reviews

```text
Review list
Review detail
Moderation status display
```

### Payments

```text
Payment settings
PayPal setup
Cash payment settings
Payment history
```

### Settings

```text
Restaurant profile settings
Branding settings
Legal information settings
Order configuration
Menu configuration
Restaurant user management
```

### Integration

```text
Ordering URL
Button variant selector
Button preview
HTML snippet
Plain link
Integration instructions
```

---

## Later Extensions

Later versions may add:

- native driver app screens
- live GPS tracking dashboard
- push notification settings
- advanced route optimisation
- automatic stop ordering
- customer-facing live tracking
- analytics and reporting
- QR code generator
- WordPress plugin integration
- embeddable menu widget
- custom button styling
- multi-location workflows
- staff invitation flow during onboarding
- review replies
- moderation audit log
- stronger role and permission management

---

## Summary

The Foodpilot admin dashboard is the restaurant's operational control center.

It should help the operator:

```text
set up the business
activate the ordering page
manage the menu
accept and process orders
control pickup and delivery
manage drivers and delivery tours
review customers and reviews
configure payments
integrate the ordering button into an existing website
```

The MVP dashboard should be simple, practical and action-oriented.

It should always make clear whether the restaurant can currently accept orders, what still needs to be configured and which operational actions need attention.
