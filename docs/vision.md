# Vision

## Why Foodpilot exists

Many restaurants rely on software solutions that are expensive, inflexible, or limit access to their own customer and operational data.

Foodpilot aims to provide a modern, open-source platform that gives restaurants full control over their ordering, pickup, delivery, and business operations.

## Product Focus

Foodpilot provides a standalone ordering interface that can be linked from an existing restaurant website through a simple call-to-action button.

```text
existing restaurant website
→ order button
→ Foodpilot menu
→ cart
→ delivery or pickup
→ customer details
→ payment
→ order confirmation
````

Foodpilot is an ordering and delivery management platform, not a restaurant website builder.

The default setup is intentionally simple:

```text
one restaurant
one active location
one menu
one customer database
one order flow
```

The active location can represent a fixed restaurant address or a mobile operating point, such as the current location of a food truck.

## Goals

* Standalone ordering interface
* Simple integration into existing restaurant websites
* Modern customer ordering experience
* Restaurant management
* Menu management
* Order management
* Pickup management
* Delivery management
* Driver and delivery-tour management
* Customer management
* Verified customer reviews
* Support for fixed and mobile operating locations
* Optional support for multiple saved locations
* Statistics and reporting
* Open API
* Extensible architecture
* Self-hostable deployment
* Full ownership of business and customer data

## Non Goals

* Restaurant marketplace
* Restaurant discovery platform
* Food delivery network
* Social network
* Full restaurant website builder
* Content management system
* Restaurant-independent driver pool
* Live GPS tracking in the MVP
* Complex route-optimization system in the MVP

## MVP Direction

The MVP focuses on small restaurants, snack bars, takeaway businesses, and food trucks.

It includes:

* one active location at a time
* delivery and pickup orders
* guest checkout and customer accounts
* PayPal and cash payment
* optional product-level inventory tracking
* delivery tours with one or more stops
* verified reviews from registered customers after completed orders
* a restaurant admin area for daily operations

The architecture remains extensible so that additional payment providers, advanced routing, live GPS tracking, embedded widgets, and multi-location workflows can be added later.

```
