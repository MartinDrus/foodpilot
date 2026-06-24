# Restaurant Onboarding Flow

## Purpose

This document describes the MVP onboarding flow for restaurant operators in Foodpilot.

The goal of onboarding is to guide a restaurant, takeaway business or food truck from registration to a working ordering page that can be linked from an existing website.

Foodpilot is not a website builder. The onboarding flow prepares the restaurant account, menu, order settings, payment settings and website integration so customers can place orders through Foodpilot.

---

## Scope

The MVP onboarding flow supports:

- restaurant account registration
- restaurant profile setup
- one active location
- fixed restaurants and mobile food trucks
- pickup configuration
- delivery configuration
- preorder configuration
- PayPal and cash payment configuration
- first menu setup
- driver setup for self-delivery
- ordering page activation
- website button integration
- onboarding checklist inside the restaurant dashboard

The onboarding flow should be practical, guided and understandable for non-technical restaurant operators.

---

## Non Goals

The MVP onboarding flow does not include:

- full restaurant website creation
- marketplace listing setup
- restaurant discovery profiles
- native driver app setup
- live GPS setup
- complex multi-location operations
- automated legal verification
- automatic menu import from PDFs
- full design editor for the restaurant website

---

## Main Idea

A restaurant operator should be able to understand Foodpilot in one simple flow:

```text
create account
→ create restaurant profile
→ configure location
→ configure opening hours
→ configure pickup and delivery
→ configure payment
→ create menu
→ add drivers if delivery is offered
→ activate ordering page
→ add Foodpilot button to existing website
```

The admin dashboard should always show what is already completed and what is still required before orders can be accepted.

---

## Onboarding States

A restaurant account can move through the following onboarding states:

```text
account_created
restaurant_profile_created
location_configured
ordering_configured
payment_configured
menu_created
drivers_configured
ordering_page_ready
ordering_page_active
```

These states do not have to be stored as a separate enum in the MVP. They can be derived from existing restaurant, location, menu, payment and driver data.

---

## Required Setup Checklist

The dashboard should show a setup checklist after registration.

Required before customer checkout can be enabled:

```text
Restaurant profile
Active location
Opening hours
Pickup or delivery enabled
Payment method configured
At least one active menu category
At least one available menu item
Ordering page slug
```

Required only when delivery is enabled:

```text
Delivery radius
Delivery fee
Delivery address validation
At least one active driver or valid delivery availability strategy
```

Recommended but not strictly required:

```text
Restaurant logo
Brand color
Legal information
Product labels
Product options
Preorder settings
Additional saved locations
Website integration button
```

---

## Registration Flow

The restaurant operator starts by creating a restaurant user account.

Required fields:

```text
name
email
password
```

The first restaurant user becomes the main admin.

After registration, the user is redirected into the onboarding flow.

---

## Step 1: Restaurant Profile

The operator creates the restaurant profile.

Fields:

```text
restaurant name
slug
contact email
contact phone
business type
currency
```

Supported business types:

```text
restaurant
takeaway
food_truck
other
```

The slug is used for the public ordering page.

Example:

```text
https://foodpilot.org/pizza-example
```

A self-hosted installation may use a different base URL.

---

## Step 2: Location Setup

The operator creates the first location.

The location can represent:

- a fixed restaurant address
- a takeaway location
- a food truck operating point

Required for enabled locations:

```text
location name
display address
coordinates
timezone
```

The first enabled location becomes the active location.

Foodpilot should make the concept simple in the UI:

```text
This is the place where orders start.
For a restaurant, this is usually your address.
For a food truck, this is your current or selected operating location.
```

A restaurant may save multiple locations, but the MVP uses one active location for customer ordering.

---

## Step 3: Opening Hours

The operator configures opening hours for the active location.

Opening hours are used to determine when customers can place orders.

The UI should support:

- weekday-based opening hours
- multiple intervals per day
- closed days
- overnight intervals
- exceptions for holidays or special days

The dashboard should explain:

```text
Customers can only order when your active location is open and order acceptance is enabled.
```

---

## Step 4: Order Acceptance

The operator controls whether the active location currently accepts orders.

Supported state:

```text
orderAcceptanceStatus: "open" | "paused"
```

This is different from opening hours.

Opening hours describe the regular schedule.

Order acceptance describes the current operational state.

Example UI text:

```text
Pause order acceptance when your kitchen is overloaded, you are temporarily closed, or you cannot handle new orders.
```

When paused, customers should not be able to place new orders.

---

## Step 5: Pickup and Delivery Settings

The operator decides whether to offer:

```text
pickup
delivery
both
```

At least one fulfilment type must be enabled before checkout can be activated.

### Pickup

Pickup settings should be simple.

The operator enables pickup and defines whether pickup orders are currently accepted during valid opening hours.

Pickup orders do not require drivers or delivery tours.

### Delivery

Delivery settings include:

```text
delivery enabled
delivery radius
minimum order value
delivery fee
```

Delivery orders require a valid delivery address.

When a delivery radius is configured, Foodpilot validates whether the customer address is inside the delivery area.

The MVP should keep the delivery setup clear:

```text
Set the area where you deliver.
Set the minimum order value.
Set the delivery fee.
Add drivers if you deliver with your own staff.
```

---

## Step 6: Preorder Settings

The operator may allow customers to place orders for a later time.

Preorders are optional.

Settings:

```text
enabled
mode
minimum lead time
maximum advance days
slot interval
maximum orders per slot
```

Supported modes:

```text
next_available_time
time_slots
```

In the MVP, preorders require online payment.

Cash payment is not offered for preorders.

The UI should explain:

```text
Preorders allow customers to order now and choose a later pickup or delivery time.
```

---

## Step 7: Payment Setup

The MVP supports:

```text
PayPal
cash
```

### PayPal

The operator connects or enters PayPal payment details.

PayPal is used for online payment.

Foodpilot should make clear that payment setup is required for online orders and preorders.

### Cash

Cash payment may be enabled depending on the restaurant configuration.

Cash can be offered for:

- pickup
- delivery

Risk control rules may restrict cash pickup for guests or customers without successful order history.

### Payment Readiness

The dashboard should show payment readiness clearly:

```text
Online payment ready
Cash payment enabled
Preorders available
```

If PayPal is missing, preorders should remain unavailable.

---

## Step 8: Menu Setup

The operator creates the first menu.

Minimum required:

```text
one active category
one available menu item
```

Menu categories define structure.

Menu items define products.

Menu item setup includes:

```text
name
description
price
category
availability status
labels
options
optional inventory tracking
```

The dashboard should guide the operator to create a simple first menu quickly.

Example sequence:

```text
Create category: Pizza
Create item: Margherita
Set price
Mark as available
Save
```

The operator can edit products at any time through the dashboard.

---

## Step 9: Product Availability

The operator controls whether products are currently available.

Supported menu item states:

```text
available
temporarily_unavailable
inactive
```

Unavailable product display is configured at restaurant level:

```text
hidden
shown_as_unavailable
```

The UI should explain this in plain language:

```text
You decide whether unavailable products disappear from the menu or remain visible as unavailable.
```

---

## Step 10: Driver Setup

Driver setup is only required when the restaurant offers self-delivery.

The operator can create drivers.

Driver fields:

```text
name
phone
vehicle type
duty status
active state
```

Vehicle types:

```text
bike
car
scooter
walking
other
```

Duty statuses:

```text
off_duty
on_duty
paused
```

Drivers do not need their own login in the MVP.

The dashboard should explain:

```text
Drivers are used for planning delivery tours. In the MVP, drivers are managed by the restaurant team inside the admin dashboard.
```

---

## Step 11: Ordering Page Preview

Before activation, the operator should be able to preview the ordering page.

Preview should show:

- restaurant name
- active location information
- opening status
- menu categories
- menu items
- product options
- pickup and delivery choice
- estimated fulfilment information where available

The preview should make it clear whether the page is ready for customers.

---

## Step 12: Activate Ordering Page

The ordering page can be activated when all required setup checks pass.

Required readiness checks:

```text
restaurant profile exists
active location exists
active location is enabled
opening hours are configured
order acceptance is open or can be opened
pickup or delivery is enabled
payment rules are valid
at least one active category exists
at least one available menu item exists
ordering slug exists
```

Additional delivery readiness checks when delivery is enabled:

```text
delivery radius is configured or delivery is intentionally unrestricted
delivery fee is configured
delivery address validation is available
driver setup is completed or restaurant uses another valid delivery handling strategy
```

When the ordering page is active, customers can access it through the public ordering URL.

---

## Step 13: Website Integration

Foodpilot is designed to be linked from existing restaurant websites.

The operator receives:

```text
public ordering URL
button preview
copyable HTML snippet
copyable plain link
optional QR code later
```

The simplest integration is a normal link or button.

Example:

```html
<a href="https://foodpilot.org/pizza-example">Order food</a>
```

The dashboard should not assume that the restaurant uses a specific website system.

It should provide simple instructions for:

- static HTML websites
- WordPress
- website builders
- social media profiles
- QR code usage later

---

## Website Button Variants

Foodpilot should provide several visual button variants, similar to payment provider buttons.

The operator can choose the variant that fits the existing website.

### Variant 1: Primary Button

Use case:

Main call-to-action on the restaurant website.

Label examples:

```text
Order food
Essen bestellen
Order online
Jetzt bestellen
```

Visual direction:

```text
solid background
high contrast
rounded corners
Foodpilot branding optional
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-primary" href="https://foodpilot.org/pizza-example">
  Order food
</a>
```

---

### Variant 2: Outline Button

Use case:

Websites that already have a strong visual design and need a lighter integration.

Visual direction:

```text
transparent background
border
text color
rounded corners
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-outline" href="https://foodpilot.org/pizza-example">
  Order food
</a>
```

---

### Variant 3: Compact Button

Use case:

Header navigation, mobile menus or small spaces.

Visual direction:

```text
smaller height
short label
minimal padding
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-compact" href="https://foodpilot.org/pizza-example">
  Order
</a>
```

---

### Variant 4: Full Width Button

Use case:

Mobile websites, landing pages or prominent ordering sections.

Visual direction:

```text
full container width
large touch target
strong call-to-action
```

Example HTML:

```html
<a class="foodpilot-button foodpilot-button-full" href="https://foodpilot.org/pizza-example">
  Order food
</a>
```

---

### Variant 5: Branded Foodpilot Button

Use case:

Restaurants that want customers to recognise the ordering provider.

Visual direction:

```text
Foodpilot logo or wordmark
label text
consistent brand color
```

Label examples:

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

### Variant 6: Plain Text Link

Use case:

Footer links, menus, social media profiles or simple websites.

Example:

```html
<a href="https://foodpilot.org/pizza-example">Order food</a>
```

---

## Button Configuration Options

The dashboard should allow the operator to configure:

```text
button label
language
button variant
button size
open in same tab or new tab
optional tracking parameter
```

The MVP should avoid a complex visual editor.

Instead, it should offer a small set of reliable, accessible button variants.

---

## Button Snippet Requirements

Generated button snippets should be:

- simple
- accessible
- copyable
- independent of a JavaScript framework
- usable on most websites
- safe to paste into common website builders
- easy to understand

The default snippet should use a normal link.

Foodpilot may later provide embeddable widgets, but the MVP should rely on a simple link or button.

---

## Admin Dashboard Structure

The restaurant dashboard should contain the following main sections:

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

### Dashboard

The dashboard is the operational home screen.

It should show:

- onboarding checklist
- order acceptance status
- active location
- today's orders
- open orders
- delivery status
- setup warnings
- quick actions

Quick actions:

```text
Pause orders
Open orders
Add product
Create delivery tour
Copy order link
```

### Orders

The orders section manages incoming and historical orders.

It should support:

- order list
- status filters
- order detail
- accept or reject order
- update preparation state
- mark ready
- assign delivery order to tour
- mark picked up
- mark delivered
- handle failed delivery

### Menu

The menu section manages:

- categories
- menu items
- product options
- labels
- availability
- optional inventory

### Locations

The locations section manages:

- active location
- saved locations
- address
- coordinates
- opening hours
- exceptions
- order acceptance state

### Delivery

The delivery section manages:

- delivery enabled state
- delivery radius
- delivery fee
- minimum order value
- delivery tours
- planned tours
- active tours
- completed tours

### Drivers

The drivers section manages:

- driver list
- driver details
- duty status
- vehicle type
- active state

### Customers

The customers section shows customer records and useful customer statistics.

It should show:

- customer name
- email
- phone where available
- order count
- total spent
- last order date

### Reviews

The reviews section shows verified reviews.

Restaurants can view reviews but cannot directly hide negative reviews only because they are negative.

Platform moderation can hide reviews that violate rules.

### Payments

The payments section shows:

- payment configuration
- payment status overview
- PayPal configuration
- cash payment settings
- payment history

### Settings

Settings include:

- restaurant profile
- branding
- legal information
- order configuration
- menu configuration
- user management

### Integration

The integration section provides:

- public ordering URL
- button preview
- button variant selector
- generated HTML snippet
- plain link
- setup instructions

This section is important because Foodpilot is meant to be connected to existing restaurant websites.

---

## Dashboard Readiness Warnings

The dashboard should show clear warnings when the ordering page cannot accept orders.

Examples:

```text
No active location selected.
Opening hours are missing.
Pickup and delivery are both disabled.
PayPal is required before preorders can be enabled.
No available menu items exist.
Delivery is enabled but no delivery radius is configured.
Delivery is enabled but no active driver is available.
Order acceptance is currently paused.
```

Warnings should be practical and link directly to the section where the problem can be fixed.

---

## Customer-Facing Activation Rules

The ordering page can be visible before it is fully active.

Possible public states:

```text
setup_incomplete
closed
paused
open
```

### Setup Incomplete

The page exists, but checkout is disabled.

Use when required setup is missing.

### Closed

The active location is currently outside opening hours.

The menu may still be visible depending on configuration.

### Paused

The restaurant is open by schedule but has manually paused order acceptance.

### Open

Customers can place orders.

---

## Practical MVP Defaults

The MVP should use sensible defaults to reduce setup friction.

Suggested defaults:

```text
currency: EUR
timezone: derived from location or deployment
pickupEnabled: true
deliveryEnabled: false
orderAcceptanceStatus: open
preordersEnabled: false
unavailableProductDisplayMode: shown_as_unavailable
cashPaymentEnabled: true
paypalPaymentEnabled: false until configured
```

Delivery should require explicit setup before it becomes available.

---

## Data Relationships Used During Onboarding

The onboarding flow creates and updates the following collections:

```text
restaurantUsers
restaurants
locations
menuCategories
menuItems
drivers
```

Later operational use creates:

```text
orders
payments
inventoryReservations
orderCounters
deliveryTours
reviews
```

The onboarding flow must not create fake operational data.

---

## Validation Rules

The backend must validate:

- restaurant user authentication
- restaurant ownership
- unique restaurant slug
- valid active location
- enabled location before checkout
- valid coordinates for enabled locations
- at least one fulfilment type before checkout
- valid payment configuration
- PayPal before preorder activation
- at least one active category before activation
- at least one available menu item before activation
- valid delivery settings before delivery activation
- valid driver setup when delivery depends on restaurant drivers
- safe website integration snippet generation

Frontend validation improves the setup experience, but backend validation is authoritative.

---

## Recommended First-Time UX

The first-time experience should be guided and calm.

The operator should not see the full complexity at once.

Recommended structure:

```text
Welcome
Business profile
Location
Opening hours
Pickup and delivery
Payment
First menu item
Website integration
Done
```

After onboarding, the operator lands in the full dashboard.

The full dashboard should still show the setup checklist until all required steps are complete.

---

## Later Extensions

Later versions may add:

- native driver app onboarding
- live GPS setup
- advanced route optimisation setup
- automatic menu import
- QR code generator
- embeddable menu widget
- deeper WordPress plugin integration
- custom button styling
- hosted custom ordering domains
- multi-location workflows
- staff invitations during onboarding
- guided test order
- onboarding progress analytics

---

## Summary

Restaurant onboarding turns a new account into a usable Foodpilot ordering setup.

The MVP onboarding flow should be simple:

```text
account
→ restaurant
→ location
→ opening hours
→ pickup or delivery
→ payment
→ menu
→ drivers if needed
→ ordering page
→ website button
```

The restaurant dashboard must make the setup status visible at all times.

The website integration step is essential. Foodpilot should provide a public ordering link and several ready-to-use button variants so restaurants can add a clear "Order food" call-to-action to their existing website without needing a new website or marketplace listing.
