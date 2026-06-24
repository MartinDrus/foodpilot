# MVP Screens

## Purpose

This document lists the MVP screens required for Foodpilot.

The goal is to provide a practical screen map for UI design, Figma wireframes and later frontend implementation.

Foodpilot is a self-hostable ordering platform for restaurants, takeaway businesses and food trucks. It is not a marketplace, delivery network or website builder.

---

## Scope

The MVP screens cover:

- restaurant authentication
- restaurant onboarding
- restaurant admin dashboard
- order management
- menu management
- location management
- delivery and driver operations
- website integration
- public customer ordering
- checkout
- order confirmation

The screen list focuses on the first usable product version.

---

## Screen Groups

```text
Restaurant Auth
Restaurant Onboarding
Restaurant Dashboard
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
Customer Ordering
Checkout
Confirmation
```

---

## Restaurant Auth

### Restaurant Signup

Purpose:

Create the first restaurant user account.

Main elements:

- name field
- email field
- password field
- submit button
- login link
- basic legal/privacy acknowledgement

Primary action:

```text
Create account
```

After success:

```text
redirect to onboarding
```

---

### Restaurant Login

Purpose:

Allow restaurant users to access the admin dashboard.

Main elements:

- email field
- password field
- login button
- password reset link
- signup link

Primary action:

```text
Log in
```

After success:

```text
redirect to dashboard
```

---

### Restaurant Password Reset

Purpose:

Allow restaurant users to request a password reset.

Main elements:

- email field
- submit button
- confirmation message

Primary action:

```text
Send reset link
```

---

## Restaurant Onboarding

### Onboarding Welcome

Purpose:

Explain the setup process after signup.

Main elements:

- short welcome text
- setup checklist preview
- start setup button

Primary action:

```text
Start setup
```

---

### Create Restaurant Profile

Purpose:

Create the restaurant business profile.

Main elements:

- restaurant name
- slug
- contact email
- contact phone
- business type
- currency

Primary action:

```text
Save restaurant profile
```

---

### Create First Location

Purpose:

Create the first operational location.

Main elements:

- location name
- display address
- structured address
- coordinates
- timezone
- fixed/mobile explanation

Primary action:

```text
Save location
```

---

### Configure Opening Hours

Purpose:

Define when customers can order.

Main elements:

- weekday schedule
- multiple intervals per day
- closed day toggle
- overnight interval support
- opening hour exceptions link

Primary action:

```text
Save opening hours
```

---

### Configure Pickup and Delivery

Purpose:

Choose available fulfilment types.

Main elements:

- pickup enabled toggle
- delivery enabled toggle
- delivery radius
- minimum order value
- delivery fee
- explanation of delivery requirements

Primary action:

```text
Save fulfilment settings
```

---

### Configure Payment

Purpose:

Configure payment readiness.

Main elements:

- PayPal setup status
- PayPal connect/configuration action
- cash payment toggle
- preorder payment explanation

Primary action:

```text
Save payment settings
```

---

### Create First Menu Category

Purpose:

Create the first menu structure.

Main elements:

- category name
- slug
- sort order
- active state

Primary action:

```text
Create category
```

---

### Create First Menu Item

Purpose:

Create the first orderable product.

Main elements:

- product name
- description
- category
- price
- availability status
- labels
- basic options
- optional inventory tracking

Primary action:

```text
Create menu item
```

---

### Website Integration Setup

Purpose:

Give the restaurant its public ordering link and button snippet.

Main elements:

- public ordering URL
- button preview
- button variant selector
- button label selector
- HTML snippet
- copy link button
- copy snippet button

Primary actions:

```text
Copy ordering link
Copy button snippet
Preview ordering page
```

---

### Onboarding Complete

Purpose:

Confirm that the restaurant can start using Foodpilot.

Main elements:

- readiness summary
- completed checklist
- open dashboard button
- preview ordering page button

Primary action:

```text
Go to dashboard
```

---

## Restaurant Dashboard

### Dashboard Home

Purpose:

Show current setup and daily operations status.

Main elements:

- setup checklist
- readiness state
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

Primary actions:

```text
Pause orders
Accept orders
Add menu item
Create delivery tour
Copy ordering link
Open ordering page preview
```

---

### Setup Checklist

Purpose:

Show what is required before the ordering page can accept checkout.

Main elements:

- restaurant profile status
- active location status
- opening hours status
- pickup/delivery status
- payment status
- menu status
- ordering slug status
- integration status
- delivery readiness if delivery is enabled
- preorder readiness if preorders are enabled

Primary action:

```text
Fix missing setup item
```

---

### Readiness Warnings

Purpose:

Show blocking issues and operational warnings.

Examples:

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

Primary action:

```text
Open related settings
```

---

## Orders

### Order List

Purpose:

Show incoming and historical orders.

Main elements:

- order number
- customer name
- order type
- status
- payment status
- estimated fulfilment time
- total amount
- created time
- delivery tour indicator

Filters:

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

Primary action:

```text
Open order
```

---

### Order Detail

Purpose:

Manage a single order.

Main elements:

- order number
- customer snapshot
- delivery address snapshot where relevant
- ordered items
- selected options
- total
- payment status
- fulfilment estimate
- status history
- delivery assignment history where relevant
- available actions

Actions:

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

---

### Failed Delivery Flow

Purpose:

Handle an unsuccessful delivery attempt.

Main elements:

- order information
- failed stop information
- customer contact information
- failure reason
- retry action
- cancel action

Primary actions:

```text
Plan retry
Cancel order
```

---

## Menu

### Menu Overview

Purpose:

Manage categories and menu items.

Main elements:

- category list
- product list
- availability indicators
- inventory warnings
- add category action
- add product action

Primary actions:

```text
Add category
Add menu item
```

---

### Category Editor

Purpose:

Create or edit a menu category.

Main elements:

- name
- slug
- sort order
- active state

Primary action:

```text
Save category
```

---

### Menu Item Editor

Purpose:

Create or edit a product.

Main elements:

- name
- description
- category
- price
- labels
- availability status
- unavailable reason
- product options
- optional inventory tracking

Primary action:

```text
Save menu item
```

---

### Product Availability Controls

Purpose:

Quickly change product availability.

Main elements:

- availability status selector
- unavailable reason
- unavailable since display
- restaurant-level display mode reference

Primary actions:

```text
Mark available
Mark temporarily unavailable
Mark inactive
```

---

## Locations

### Location List

Purpose:

Manage saved locations and active location.

Main elements:

- location name
- display address
- active state
- enabled state
- order acceptance status
- opening status

Primary actions:

```text
Add location
Set active location
Edit location
```

---

### Location Editor

Purpose:

Create or edit a location.

Main elements:

- location name
- display address
- structured address
- coordinates
- timezone
- enabled state
- order acceptance status
- pickup settings
- delivery settings
- preorder settings

Primary action:

```text
Save location
```

---

### Opening Hours Editor

Purpose:

Configure regular opening hours.

Main elements:

- weekday rows
- time intervals
- closed day toggle
- overnight interval support

Primary action:

```text
Save opening hours
```

---

### Opening Hour Exceptions

Purpose:

Manage special opening or closing days.

Main elements:

- exception date
- open/closed state
- custom intervals
- note

Primary action:

```text
Save exception
```

---

## Delivery

### Delivery Settings

Purpose:

Configure delivery and pickup availability.

Main elements:

- pickup enabled toggle
- delivery enabled toggle
- delivery radius
- minimum order value
- delivery fee
- delivery availability warnings

Primary action:

```text
Save delivery settings
```

---

### Delivery Tour List

Purpose:

Manage planned, active and completed tours.

Main groups:

```text
Planned
In progress
Returning
Completed
Cancelled
```

Main elements:

- tour status
- assigned driver
- stop count
- estimated return time
- loading status

Primary actions:

```text
Create tour
Open tour
```

---

### Delivery Tour Detail

Purpose:

Manage one delivery tour.

Main elements:

- status
- loading status
- assigned driver
- location snapshot
- stop list
- stop sequence
- order references
- delivery addresses
- estimated arrival times
- estimated return time

Actions:

```text
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

---

## Drivers

### Driver List

Purpose:

Manage delivery drivers.

Main elements:

- name
- phone
- vehicle type
- duty status
- active state
- current or next tour where relevant

Primary actions:

```text
Add driver
Edit driver
Set on duty
Set off duty
Pause driver
```

---

### Driver Editor

Purpose:

Create or edit a driver.

Main elements:

- name
- phone
- vehicle type
- duty status
- active state
- internal note

Primary action:

```text
Save driver
```

---

## Customers

### Customer List

Purpose:

Show customer records.

Main elements:

- display name
- email
- phone where available
- order count
- total spent
- last order date
- account status

Primary action:

```text
Open customer
```

---

### Customer Detail

Purpose:

Show customer information and history.

Main elements:

- customer details
- saved addresses where available
- order history
- successful order count
- review history where relevant

Primary action:

```text
View order history
```

---

## Reviews

### Review List

Purpose:

Show verified customer reviews.

Main elements:

- customer display name
- overall rating
- delivery rating where relevant
- comment
- item ratings where available
- related order
- published date
- moderation status

Primary action:

```text
Open review
```

---

### Review Detail

Purpose:

Show one review with context.

Main elements:

- review content
- related order
- customer reference
- item ratings
- moderation status
- deleted state where relevant

Restaurant users can view reviews but cannot hide negative reviews only because they are negative.

---

## Payments

### Payment Settings

Purpose:

Configure payment methods.

Main elements:

- PayPal status
- PayPal setup action
- cash payment toggle
- preorder payment warning
- cash restriction information

Primary actions:

```text
Configure PayPal
Enable cash
Disable cash
```

---

### Payment History

Purpose:

Show payment records.

Main elements:

- order number
- payment method
- amount
- status
- provider reference
- created date

Primary action:

```text
Open related order
```

---

## Settings

### Restaurant Profile Settings

Purpose:

Manage restaurant identity.

Main elements:

- restaurant name
- slug
- contact email
- contact phone
- business type
- currency

Primary action:

```text
Save profile
```

---

### Branding Settings

Purpose:

Manage basic ordering page appearance.

Main elements:

- logo
- brand color
- ordering page preview

Primary action:

```text
Save branding
```

---

### Legal Information Settings

Purpose:

Manage legal business information.

Main elements:

- legal business name
- legal address
- tax information where required
- imprint information
- privacy information

Primary action:

```text
Save legal information
```

---

### Restaurant User Management

Purpose:

Manage restaurant dashboard users.

Main elements:

- user list
- role
- active state
- last login

Roles:

```text
main_admin
operator
```

Primary actions:

```text
Invite user
Deactivate user
Change role
```

---

## Integration

### Integration Overview

Purpose:

Help the restaurant add Foodpilot ordering to an existing website.

Main elements:

- public ordering URL
- ordering page preview
- integration readiness status
- copied status
- button variant selector
- button preview
- button label selector
- language selector
- generated HTML snippet
- plain link

Primary actions:

```text
Copy ordering link
Copy HTML snippet
Preview ordering page
```

---

### Button Variant Selector

Purpose:

Choose a visual button variant.

Variants:

```text
Primary Button
Outline Button
Compact Button
Full Width Button
Branded Foodpilot Button
Plain Text Link
```

Each variant should show:

- preview
- use case
- example label
- generated HTML snippet

---

### Integration Instructions

Purpose:

Explain how to add the button to common website types.

Instruction groups:

```text
HTML website
WordPress
Website builder
Social media profile
```

The MVP uses simple `<a>` links and does not require a JavaScript widget.

---

## Customer Ordering

### Customer Menu

Purpose:

Show the public restaurant menu.

Main elements:

- restaurant name
- active location information
- opening or paused state
- category navigation
- product cards
- product labels
- availability state
- cart entry point
- pickup/delivery availability

Primary action:

```text
Select product
```

---

### Product Detail

Purpose:

Configure one product before adding it to the cart.

Main elements:

- product name
- description
- price
- labels
- option groups
- selectable options
- quantity selector
- availability state

Primary action:

```text
Add to cart
```

---

### Cart

Purpose:

Review selected products before checkout.

Main elements:

- cart items
- selected options
- quantities
- item prices
- subtotal
- delivery fee where relevant
- total
- unavailable item warnings

Actions:

```text
Edit item
Remove item
Change quantity
Continue to checkout
```

---

## Checkout

### Fulfilment Selection

Purpose:

Choose pickup or delivery.

Main elements:

- pickup option
- delivery option
- availability state
- estimated time where available

Primary action:

```text
Continue
```

---

### Delivery Address Form

Purpose:

Collect the delivery address.

Main elements:

- street
- house number
- zip
- city
- floor
- note

Primary action:

```text
Check delivery address
```

---

### Customer Details Form

Purpose:

Collect customer contact data.

Main elements:

- first name
- last name
- email
- phone
- guest checkout option
- login option

Phone is required for delivery orders.

Primary action:

```text
Continue
```

---

### Preorder Time Selection

Purpose:

Choose a future fulfilment time when preorders are enabled.

Main elements:

- next available time
- time slot selector
- unavailable slot state
- payment requirement note

Primary action:

```text
Select time
```

---

### Payment Selection

Purpose:

Choose a valid payment method.

Main elements:

- PayPal
- cash where allowed
- payment restriction messages

Primary action:

```text
Continue to summary
```

---

### Order Summary

Purpose:

Confirm the order before submission.

Main elements:

- restaurant name
- order type
- fulfilment time
- customer details
- delivery address where relevant
- items
- subtotal
- delivery fee
- total
- payment method

Primary action:

```text
Place order
```

---

## Confirmation

### Payment Step

Purpose:

Handle online payment when PayPal is selected.

Main elements:

- PayPal action
- payment status
- retry option if payment fails

Primary action:

```text
Pay with PayPal
```

---

### Order Confirmation

Purpose:

Confirm successful order placement.

Main elements:

- order number
- order status
- payment status
- estimated pickup or delivery time
- restaurant name
- pickup or delivery address
- ordered items
- total amount

Primary action:

```text
View order status
```

---

### Order Status View

Purpose:

Show simplified customer-facing order status.

Customer-facing labels:

```text
Waiting for payment
Order received
Accepted
Preparing
Ready for pickup
Out for delivery
Delivery issue
Delivered
Picked up
Rejected
Cancelled
Expired
```

The MVP does not include live GPS tracking.

---

### Review Form

Purpose:

Allow registered customers to review eligible completed orders.

Requirements:

```text
registered customer
order belongs to customer
order status is delivered or picked_up
order has no existing review
```

Main elements:

- overall rating
- delivery rating where relevant
- comment
- item ratings where available

Primary action:

```text
Submit review
```

---

## Priority for Figma

The first Figma phase should focus on these core screens:

```text
Restaurant Signup
Restaurant Login
Onboarding Welcome
Create Restaurant Profile
Create First Location
Configure Opening Hours
Configure Pickup and Delivery
Configure Payment
Create First Menu Item
Dashboard Home
Order List
Order Detail
Menu Overview
Menu Item Editor
Location Editor
Delivery Tour List
Delivery Tour Detail
Integration Overview
Customer Menu
Product Detail
Cart
Checkout
Order Confirmation
```

These screens cover the complete MVP path from restaurant setup to customer order.

---

## Summary

The MVP screen set is split into two major areas:

```text
Restaurant side
Customer side
```

The restaurant side covers signup, onboarding, dashboard operations, menu, orders, locations, delivery, drivers, payments and website integration.

The customer side covers menu browsing, product configuration, cart, checkout, payment, confirmation, order status and eligible reviews.

This screen list is intentionally practical and can be used directly as the starting point for UI design and Figma wireframes.
