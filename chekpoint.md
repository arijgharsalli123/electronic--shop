How we'll work 🚀

We'll go through the project roughly like this:

🧱 Understand what you already built
📁 Project structure & architecture
🎨 Frontend — pages, components, UI
🔌 Backend/API — authentication, products, orders, etc.
🗄️ Database — schema + relationships
🔗 Connect frontend ↔ backend ↔ database
🔐 Authentication & authorization
🛒 Cart & checkout
📦 Products / orders / users management
🧪 Testing & debugging
📱 Responsive design
🚀 Deployment
📄 Documentation + GitHub portfolio


Architecture 
electronic-shop/
├── electronic-frontend/
└── electronic-backend/

Frontend → Next.js
             ↓
Backend  → http://localhost:9090
             ↓
Database → PostgreSQL :5432

🧭 Our development approach

Before writing code, I want us to establish the architecture of the shop:

Customer side

Home
 ├── Categories
 ├── Products
 ├── Product details
 ├── Search / filters
 ├── Cart
 ├── Checkout
 └── My orders

Admin side

Admin Dashboard
 ├── Products
 ├── Categories
 ├── Customers
 ├── Orders
 └── Statistics
  

  how ro run spring boot backend :  .\mvnw.cmd spring-boot:run

  how to run next.js frontend :
   
how to connect to postgres :  psql -U postgres

Target architecture


                    ┌──────────────────────┐
                    │      Next.js         │
                    │  Frontend + UI       │
                    └──────────┬───────────┘
                               │ HTTPS / REST
                               ▼
                    ┌──────────────────────┐
                    │     Spring Boot      │
                    │       Backend        │
                    │                      │
                    │  Security / JWT      │
                    │  Users               │
                    │  Products            │
                    │  Cart                │
                    │  Orders              │
                    │  Payments            │
                    │  Reviews             │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     PostgreSQL       │
                    └──────────────────────┘

    For security, we'll eventually cover things such as:

password hashing with BCrypt/Argon2
email verification
authentication
authorization / roles
JWT or secure session strategy
refresh-token security if we use JWT
protected admin endpoints
input validation
SQL injection protection
CORS
CSRF considerations
rate limiting
secure password reset
account activation/deactivation
protection against common OWASP attacks
secure file/image upload
payment security
avoiding sensitive information in logs
environment variables/secrets
transactional stock management*

🛒 Your business logic

I've understood your rules as follows.

Order
PENDING
   │
   ├──→ PAID
   │      │
   │      ├──→ PROCESSING
   │      │       │
   │      │       └──→ SHIPPED
   │      │                │
   │      │                └──→ DELIVERED
   │      │
   │      └──→ CANCELLED
   │
   └──→ CANCELLED

Cancellation is allowed only while:

PENDING
PAID

Good.

Stock

We'll protect this at two levels:

Add to cart
     ↓
Check current stock

and, more importantly:

Create order
     ↓
@Transactional
     ↓
Check stock again
     ↓
Atomically decrease stock
     ↓
Create order

This second check is essential because another customer could buy the same product between adding it to the cart and placing the order.

If an eligible order is cancelled:

cancel order
      ↓
restore stock

We'll pay particular attention to concurrency here.

💰 Pricing

We'll implement:

Sous-total HT
= Σ(prix unitaire × quantité)

Remise
= percentage OR fixed

Livraison
= 7 DT if subtotal < 50 DT
= 0 DT if subtotal >= 50 DT

Total
= subtotal - discount + shipping

One thing I noticed: you wrote "Total TTC", but your formula doesn't currently include VAT/tax.

So we need to clarify this later:

Do you actually want VAT/TVA in the project, or should the final amount simply be called Total?

I recommend deciding this before implementing Order.

⭐ Reviews

Your rule:

One customer can leave only one review per product.

We'll enforce this at both application and database level.

For example, conceptually:

UNIQUE(user_id, product_id)

That way, even if someone bypasses the frontend, the database still protects the rule.

📧 Email verification

Your authentication flow will be something like:

Register
   ↓
Create account
   ↓
enabled = false
   ↓
Send verification email
   ↓
User clicks link
   ↓
Verify token
   ↓
enabled = true
   ↓
User can login

We'll also need:

Forgot password
       ↓
Reset email
       ↓
Secure reset token
       ↓
New password

We'll build this properly rather than storing verification codes carelessly.

💳 Payment

You want:

💳 Bank card
🟢 E-Dinar card

This needs one important clarification because we must not store bank-card numbers/CVV in our database.

Ideally the payment architecture should be:

Our application
      ↓
Payment provider / secure payment gateway
      ↓
Bank / E-Dinar

Our DB stores something like:

payment status
payment method
transaction/reference ID
amount

—not the customer's card credentials.

We'll decide the exact Tunisian payment provider/gateway when we get to that stage.

🛍️ Cart

You said the cart should contain the product image and information to fill in.

I want to clarify the distinction:

Cart

Should contain:

Product
Product image
Product name
Unit price
Quantity
Subtotal

Then at checkout, the customer fills:

Full name
Phone
Address
Governorate
City
Postal code
Additional information

We shouldn't copy the entire product into the cart unless there's a specific business reason.

👨‍💼 Admin dashboard

We'll make the admin dashboard more than just a product CRUD.

Eventually:

ADMIN DASHBOARD

📊 Overview
├── Revenue
├── Orders
├── Customers
├── Products
└── Low-stock alerts

📦 Catalog
├── Products
├── Categories
└── Product images

🛒 Orders
├── All orders
├── Pending
├── Paid
├── Processing
├── Shipped
├── Delivered
└── Cancelled

👥 Customers
├── Customers
├── Activate / deactivate
└── Customer details

⭐ Reviews
├── Reviews
└── Moderation

🎟️ Discounts
├── Coupons
└── Promotions

⚙️ Settings
└── Shop configuration



🧠 Our main domain model

I recommend this structure:

USER
 ├── CUSTOMER
 ├── MANAGER
 └── ADMIN

CATEGORY
    │
    └── PRODUCT
          │
          ├── PRODUCT_IMAGE
          └── PRODUCT_SPECIFICATION

CUSTOMER
    ├── ADDRESS
    ├── CART
    │     └── CART_ITEM
    │
    ├── ORDER
    │     └── ORDER_ITEM
    │
    └── REVIEW

ORDER
    └── PAYMENT
Core tables
Entity	Purpose
User	Authentication + common user information
Role	ADMIN / MANAGER / CUSTOMER
Category	Product categories
Product	Products sold by the shop
ProductImage	Multiple images per product
ProductSpecification	Technical characteristics
Address	Customer delivery addresses
Cart	Customer's active cart
CartItem	Products + quantities
Order	Customer order
OrderItem	Snapshot of products bought
Payment	Payment status/method/reference
Review	Product rating/comment
EmailVerificationToken	Account verification
PasswordResetToken	Forgot-password workflow
🔐 Security architecture

We'll make security a first-class part of the project.

For example:

CUSTOMER
   ↓
Can:
✓ Browse products
✓ Search/filter
✓ Add to cart
✓ Manage addresses
✓ Place orders
✓ Pay
✓ View own orders
✓ Review products

Cannot:
✗ Manage products
✗ Manage categories
✗ See other customers
✗ Manage orders globally

And:

MANAGER
   ↓
Can manage:
✓ Products
✓ Categories
✓ Orders
✓ Customers (according to defined permissions)
✓ Reviews

Then:

ADMIN
   ↓
Full management
✓ Managers
✓ Customers
✓ Products
✓ Categories
✓ Orders
✓ Payments
✓ Reviews
✓ Dashboard
✓ Settings

We'll enforce these permissions in Spring Security at API level, not merely hide buttons in Next.js.

📦 Product

We'll make the product flexible enough for your electronic/robotics shop.

Conceptually:

Product
──────────────
id
name
description
price
stock
sku
brand
category
createdAt
updatedAt
active

Then:

Product
   │
   ├── ProductImage
   │      ├── imageUrl
   │      ├── isPrimary
   │      └── displayOrder
   │
   └── ProductSpecification
          ├── name
          └── value

So an Arduino product could have:

Voltage → 5V
Microcontroller → ATmega328P
Digital Pins → 14
Analog Pins → 6

while an electric programmable car could have:

Motor → DC 6V
Board → Arduino UNO
Battery → 7.4V
Bluetooth → Yes

The admin doesn't need us to modify the database every time a new technical specification appears.

🛒 Cart

We'll use:

Cart
 │
 └── CartItem
       │
       └── Product

A customer can have one active cart.

When adding:

Customer
   ↓
POST /api/cart/items
   ↓
Check authentication
   ↓
Check product exists
   ↓
Check product active
   ↓
Check stock
   ↓
Add/update CartItem

And we'll never trust the price sent by the frontend.

The backend gets the actual product price from PostgreSQL.

That's an important security/business rule.

📦 Order

This part deserves special attention.

An OrderItem should store a snapshot:

product
productName
unitPrice
quantity
subtotal

Why?

Suppose today:

Arduino = 35 DT

Customer buys it.

Tomorrow admin changes it:

Arduino = 40 DT

The old order must still say:

Arduino
35 DT
× 2
= 70 DT

We therefore don't calculate historical orders from the current Product price.

🔄 Order status

We'll create an enum:

PENDING
PAID
PROCESSING
SHIPPED
DELIVERED
CANCELLED

And enforce valid transitions in the backend.

For example:

PENDING → PAID
PENDING → CANCELLED

PAID → PROCESSING
PAID → CANCELLED

PROCESSING → SHIPPED
SHIPPED → DELIVERED

No:

DELIVERED → PENDING ❌
CANCELLED → PAID ❌
SHIPPED → CANCELLED ❌

This will be business logic, not just a frontend dropdown.

💰 Order calculation

We'll have something like:

subtotal
discount
shippingFee
total

For example:

Products       80 DT
Discount       10 DT
Shipping        0 DT
────────────────────
Total           70 DT

And:

subtotal < 50 → shipping = 7
subtotal >= 50 → shipping = 0

One thing remains to decide later: whether TVA exists. Until then, I'd call the field simply total, rather than totalTTC.

💳 Payment

We'll separate:

Order
  │
  └── Payment
       ├── method
       ├── status
       ├── amount
       └── transactionReference

Possible methods:

BANK_CARD
E_DINAR

Later, depending on the chosen payment provider, we'll integrate the gateway.

Never:

cardNumber
cvv

in our database.

⭐ Review

We'll enforce:

UNIQUE(customer_id, product_id)

Therefore:

Customer A → Arduino UNO → ⭐⭐⭐⭐⭐

cannot create a second review for the same product.

But can:

edit own review
delete own review

We'll also decide whether only customers who actually purchased the product can review it. I strongly recommend yes, because it gives us "verified buyer" reviews.

📧 Authentication

The flow will be:

REGISTER
   ↓
Validate data
   ↓
Hash password
   ↓
Create CUSTOMER
   ↓
enabled = false
   ↓
Generate secure verification token
   ↓
Send email
   ↓
User clicks verification link
   ↓
Verify token
   ↓
enabled = true
   ↓
LOGIN

And:

FORGOT PASSWORD
       ↓
Secure reset token
       ↓
Email
       ↓
New password

We'll also protect against things like token reuse and expired tokens.

🏠 Multiple addresses

You chose multiple addresses, so:

Customer
   │
   ├── Address: Home
   ├── Address: Work
   └── Address: Other

At checkout, the customer chooses one.

For the order, however, we'll snapshot the shipping address into the order.

That's important because the customer might later change/delete their address, while the historical order must retain where it was shipped.

👥 Roles

We'll use:

ROLE_CUSTOMER
ROLE_MANAGER
ROLE_ADMIN

And later Spring Security:

@PreAuthorize(...)

or equivalent endpoint authorization.

🧩 One more important thing

I recommend adding Coupon/Discount as a domain entity because you already specified:

Remise
PERCENT
FIXED

Instead of hardcoding discounts, we'll eventually support:

Coupon
 ├── code
 ├── type: PERCENT / FIXED
 ├── value
 ├── minimumOrderAmount
 ├── startDate
 ├── expirationDate
 ├── usageLimit
 └── active

That's much more realistic for an e-commerce project.

✅ Our architecture is now defined

So I don't want you to create 15 entities manually yet.

Next step is Step 4.1: create the PostgreSQL database schema through JPA entities, starting with the foundation: User + Role.

We'll build it in small increments:

Step 4.1 → User + Role
Step 4.2 → Category + Product
Step 4.3 → ProductImage + Specification
Step 4.4 → Address + Cart
Step 4.5 → Order + OrderItem
Step 4.6 → Payment
Step 4.7 → Review
Step 4.8 → Security/authentication
...

This way, after each step we can run → test → verify → fix → checkpoint, exactly like we did with Ehalat.


One category can contain many products, but each product belongs to one category.
Robotics
├── Arduino UNO
├── Raspberry Pi
└── Ultrasonic Sensor

Phone Accessories
├── USB-C Cable
├── Fast Charger
└── Phone Holder