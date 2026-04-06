# 🛍️ Instagram Thrift Creator Store

> A full-stack e-commerce platform tailored for Instagram thrift & creator-driven storefronts — enabling seamless product listings, order management, payments, and shipment tracking.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Database Schema](#database-schema)
  - [Core Entities](#core-entities)
  - [ER Diagram](#er-diagram)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Overview](#api-overview)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## 🧭 Overview

**Instagram Thrift Creator Store** is a commerce backend designed to power thrift & creator-economy businesses that primarily operate through Instagram. It provides:

- A rich product catalog supporting **regular**, **thrift (draft)**, and **handmade** product types
- Multi-variant product support (size, color, price overrides)
- Customer account management with Instagram handle tracking
- Full order lifecycle: cart → order → payment → shipment → review
- Discount coupons and coupon usage tracking
- Order status history and audit trails

---

## ✨ Features

| Feature | Description |
|---|---|
| 🛒 **Cart & Cart Items** | Persistent shopping cart per customer with expiry |
| 👤 **Customer Profiles** | Name, email, phone, Instagram handle |
| 📦 **Product Variants** | SKU, size, color, stock, price override |
| 🧵 **Handmade Products** | Production batch, materials, eco-friendly flags |
| 🏷️ **Thrift / Draft Products** | Authenticity codes, condition, unique item codes |
| 🏠 **Multiple Addresses** | Default billing/shipping address per customer |
| 📋 **Order Management** | Discount, tax, shipping charge, status lifecycle |
| 📜 **Order Status History** | Full audit trail of order status transitions |
| 💳 **Payments** | Gateway integration, refund tracking, transaction IDs |
| 🚚 **Shipments** | Carrier, tracking number, delivery dates, status |
| ⭐ **Reviews** | Rating, review text, images, verified purchase flag |
| 🎟️ **Discount Coupons** | Percentage/fixed discount, min order, usage limits |
| 📊 **Coupon Usage Tracking** | Per-order coupon application history |

---

## 🗄️ Database Schema

### Core Entities

#### 👤 `customer`
Stores customer account information.

| Column | Type | Notes |
|---|---|---|
| `customer_id` | INT PK | Primary key |
| `full_name` | VARCHAR | Customer's full name |
| `email` | VARCHAR | Unique email address |
| `phone` | VARCHAR | Contact number |
| `instagram_handle` | VARCHAR | Instagram username |
| `created_at` | DATETIME | Account creation time |
| `updated_at` | DATETIME | Last update time |

---

#### 🛒 `cart`
Tracks active shopping carts per customer.

| Column | Type | Notes |
|---|---|---|
| `cart_id` | INT PK | Primary key |
| `customer_id` | INT FK | References `customer` |
| `created_at` | DATETIME | Cart creation time |
| `updated_at` | DATETIME | Last update time |
| `expired_at` | DATETIME | Cart expiry timestamp |

---

#### 🛍️ `cart_item`
Line items inside a cart.

| Column | Type | Notes |
|---|---|---|
| `cart_item_id` | INT PK | Primary key |
| `cart_id` | INT FK | References `cart` |
| `variant_id` | INT FK | References `product_variant` |
| `quantity` | INT | Number of units |
| `added_at` | DATETIME | Time item was added |

---

#### 📦 `product`
The base product entity for all product types.

| Column | Type | Notes |
|---|---|---|
| `product_id` | INT PK | Primary key |
| `name` | VARCHAR | Product name |
| `description` | TEXT | Full description |
| `base_price` | DECIMAL | Starting price |
| `category` | VARCHAR | Product category |
| `product_type` | ENUM | `regular`, `thrift`, `handmade` |
| `brand` | VARCHAR | Brand name |
| `material` | VARCHAR | Material details |
| `care_instructions` | TEXT | Washing/care guide |
| `weight_grams` | DECIMAL | Product weight |
| `is_active` | BOOLEAN | Listing visibility |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |

---

#### 🏷️ `thrift_product`
Additional fields for second-hand/thrift items.

| Column | Type | Notes |
|---|---|---|
| `product_id` | INT FK | References `product` |
| `condition` | ENUM | Item condition rating |
| `sku` | VARCHAR | Stock Keeping Unit |
| `authenticity_code` | TEXT | Verification code |
| `is_one_of_a_kind` | BOOL | Unique item flag |
| `unique_item_code` | VARCHAR | Unique identifier |
| `resale_price` | DECIMAL | Discounted resale price |
| `item_acquired` | DATE | Date item was sourced |
| `cost_price` | DECIMAL | Purchase cost |

---

#### 🧵 `handmade_product`
Additional fields for handmade items.

| Column | Type | Notes |
|---|---|---|
| `product_id` | INT FK | References `product` |
| `is_restockable` | BOOLEAN | Can be re-ordered |
| `production_batch` | VARCHAR | Batch identifier |
| `batch_date` | DATE | Batch production date |
| `making_time_days` | INT | Days to produce |
| `materials_used` | TEXT | List of materials |
| `eco_friendly` | BOOL | Eco-friendly flag |
| `handmade_by` | VARCHAR | Maker's name |
| `maker_level` | VARCHAR | Skill level |
| `estimated_production` | INT | Expected output |
| `sustainability_cert` | VARCHAR | Certification details |

---

#### 🔀 `product_variant`
Variants of a product (size, color, etc.).

| Column | Type | Notes |
|---|---|---|
| `variant_id` | INT PK | Primary key |
| `product_id` | INT FK | References `product` |
| `size` | VARCHAR | Size label |
| `color` | VARCHAR | Color name |
| `stock_quantity` | INT | Available stock |
| `sku_code` | VARCHAR | Unique SKU per variant |
| `price_override` | DECIMAL | Override the base price |
| `weight_override` | DECIMAL | Override product weight |
| `variant_images` | TEXT | Image URLs (JSON/CSV) |
| `is_available` | BOOLEAN | Availability flag |
| `reserved_quantity` | INT | Units reserved by carts |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |

---

#### 🏠 `address`
Customer shipping/billing addresses.

| Column | Type | Notes |
|---|---|---|
| `address_id` | INT PK | Primary key |
| `customer_id` | INT FK | References `customer` |
| `line1` | VARCHAR | Address line 1 |
| `line2` | VARCHAR | Address line 2 |
| `city` | VARCHAR | City |
| `state` | VARCHAR | State |
| `pincode` | VARCHAR | Postal code |
| `country` | VARCHAR | Country |
| `is_default` | BOOLEAN | Default address flag |
| `address_type` | ENUM | `billing` / `shipping` |
| `created_at` | DATETIME | — |

---

#### 📋 `order`
A placed order by a customer.

| Column | Type | Notes |
|---|---|---|
| `order_id` | INT PK | Primary key |
| `customer_id` | INT FK | References `customer` |
| `address_id` | INT FK | References `address` |
| `order_date` | DATETIME | Date order was placed |
| `gst_amount` | DECIMAL | Tax component |
| `order_status` | ENUM | Current order status |
| `payment_status` | ENUM | Payment state |
| `shipping_status` | ENUM | Shipping state |
| `order_notes` | TEXT | Special instructions |
| `discount_amount` | DECIMAL | Applied discount |
| `shipping_charge` | DECIMAL | Delivery fee |
| `tax_amount` | DECIMAL | Total tax |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |

---

#### 📄 `order_item`
Individual line items within an order.

| Column | Type | Notes |
|---|---|---|
| `order_item_id` | INT PK | Primary key |
| `order_id` | INT FK | References `order` |
| `variant_id` | INT FK | References `product_variant` |
| `quantity` | INT | Units ordered |
| `unit_price` | DECIMAL | Price at time of order |
| `subtotal` | DECIMAL | Line total |
| `item_acquired` | DATETIME | — |
| `discount_applied` | DECIMAL | Per-item discount |
| `return_status` | ENUM | Return state |
| `created_at` | DATETIME | — |

---

#### 📜 `order_status_history`
Audit trail of order status changes.

| Column | Type | Notes |
|---|---|---|
| `history_id` | INT PK | Primary key |
| `order_id` | INT FK | References `order` |
| `old_status` | VARCHAR | Previous status |
| `new_status` | VARCHAR | New status |
| `changed_at` | DATETIME | Time of change |
| `changed_by` | VARCHAR | User/system actor |
| `notes` | TEXT | Optional remarks |

---

#### 💳 `payment`
Payment record linked to an order.

| Column | Type | Notes |
|---|---|---|
| `payment_id` | INT PK | Primary key |
| `order_id` | INT FK | References `order` |
| `amount` | DECIMAL | Payment amount |
| `payment_method` | ENUM | UPI, card, COD, etc. |
| `payment_status` | ENUM | Pending/completed/failed |
| `transaction_id` | VARCHAR | Gateway transaction ID |
| `payment_date` | DATETIME | Time of payment |
| `payment_gateway` | VARCHAR | Razorpay / Stripe, etc. |
| `gateway_response` | TEXT | Raw response JSON |
| `refund_amount` | DECIMAL | Refunded amount |
| `refund_transaction_id` | VARCHAR | Refund reference ID |
| `refund_date` | DATETIME | Date of refund |
| `created_at` | DATETIME | — |

---

#### 🚚 `shipment`
Shipment details for a delivered order.

| Column | Type | Notes |
|---|---|---|
| `shipment_id` | INT PK | Primary key |
| `order_id` | INT FK | References `order` |
| `courier_partner` | VARCHAR | Courier company name |
| `tracking_number` | VARCHAR | Tracking ID |
| `tracking_url` | VARCHAR | Live tracking link |
| `shipped_date` | DATETIME | Dispatch date |
| `estimated_delivery` | DATETIME | ETA for delivery |
| `actual_delivery_date` | DATETIME | Actual delivery date |
| `shipping_status` | ENUM | Current shipment state |
| `address_snapshot` | JSON | Address at time of ship |
| `courier_notes` | TEXT | Delivery remarks |
| `delivery_proof_url` | VARCHAR | Proof of delivery |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |

---

#### ⭐ `review`
Customer reviews for a product variant.

| Column | Type | Notes |
|---|---|---|
| `review_id` | INT PK | Primary key |
| `customer_id` | INT FK | References `customer` |
| `variant_id` | INT FK | References `product_variant` |
| `order_id` | INT FK | References `order` |
| `rating` | INT | Star rating (1–5) |
| `review_text` | TEXT | Written review |
| `review_images` | TEXT | Image URLs |
| `is_verified_purchase` | BOOLEAN | Bought before reviewing |
| `created_at` | DATETIME | — |
| `updated_at` | DATETIME | — |

---

#### 🎟️ `discount_coupon`
Store-wide or targeted discount codes.

| Column | Type | Notes |
|---|---|---|
| `coupon_id` | INT PK | Primary key |
| `code` | VARCHAR | Coupon code string |
| `discount_type` | ENUM | `percentage` / `fixed` |
| `discount_value` | DECIMAL | Discount amount/percent |
| `minimum_order_amount` | DECIMAL | Min cart value to apply |
| `maximum_discount` | DECIMAL | Cap on discount |
| `valid_from` | DATE | Start date |
| `valid_until` | DATE | Expiry date |
| `usage_limit` | INT | Max total uses |
| `used_count` | INT | Times used so far |
| `is_active` | BOOLEAN | Active flag |
| `created_by` | VARCHAR | Creator/admin |

---

#### 📊 `order_coupon_usage`
Tracks which coupon was applied to which order.

| Column | Type | Notes |
|---|---|---|
| `usage_id` | INT PK | Primary key |
| `coupon_id` | INT FK | References `discount_coupon` |
| `order_id` | INT FK | References `order` |
| `discount_applied` | DECIMAL | Actual discount given |
| `applied_at` | DATETIME | When coupon was used |

---

### ER Diagram

![Instagram Thrift Creator Store — ER Diagram](./Instagram%20ER%20Diagram.pdf)

> The ER diagram illustrates all the entity relationships described above, including foreign key links between `customer`, `cart`, `product`, `order`, `payment`, `shipment`, and `review` tables.

---

## 🛠️ Tech Stack

> *(Update this section based on your actual implementation)*

| Layer | Technology |
|---|---|
| **Backend** | Node.js / Express or Django / FastAPI |
| **Database** | MySQL / PostgreSQL |
| **ORM** | Sequelize / Prisma / SQLAlchemy |
| **Authentication** | JWT / OAuth2 |
| **Payment Gateway** | Razorpay / Stripe |
| **Storage** | AWS S3 / Cloudinary (for images) |
| **API Style** | RESTful API |

---

## 🚀 Getting Started

### Prerequisites

- Node.js >= 18 or Python >= 3.10
- MySQL >= 8.0 or PostgreSQL >= 14
- npm / pip / yarn

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/instagram-thrift-creator-store.git
cd instagram-thrift-creator-store

# Install dependencies
npm install        # For Node.js
# or
pip install -r requirements.txt   # For Python

# Set up environment variables
cp .env.example .env
# Edit .env with your database credentials and secrets

# Run database migrations
npm run migrate
# or
python manage.py migrate

# Start the development server
npm run dev
# or
python manage.py runserver
```

---

## 🔐 Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=instagram_thrift_store
DB_USER=your_db_user
DB_PASSWORD=your_db_password

# Authentication
JWT_SECRET=your_super_secret_key
JWT_EXPIRES_IN=7d

# Payment Gateway
RAZORPAY_KEY_ID=your_razorpay_key
RAZORPAY_KEY_SECRET=your_razorpay_secret

# Storage
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Server
PORT=3000
NODE_ENV=development
```

---

## 📡 API Overview

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Register a new customer |
| `POST` | `/api/auth/login` | Customer login |
| `GET` | `/api/products` | List all products |
| `GET` | `/api/products/:id` | Get product details |
| `POST` | `/api/cart` | Add item to cart |
| `GET` | `/api/cart` | View cart |
| `POST` | `/api/orders` | Place an order |
| `GET` | `/api/orders/:id` | Get order details |
| `POST` | `/api/payments` | Initiate payment |
| `GET` | `/api/shipments/:orderId` | Track shipment |
| `POST` | `/api/reviews` | Submit a review |
| `POST` | `/api/coupons/apply` | Apply a discount coupon |

> 📝 Full API documentation available in `/docs/api.md` *(coming soon)*

---

## 📁 Project Structure

```
instagram-thrift-creator-store/
├── src/
│   ├── controllers/        # Route handler logic
│   ├── models/             # Database models / ORM schemas
│   ├── routes/             # API route definitions
│   ├── middlewares/        # Auth, error handling, validation
│   ├── services/           # Business logic layer
│   ├── utils/              # Helper functions
│   └── config/             # DB, app, and env configuration
├── migrations/             # Database migration files
├── docs/
│   └── api.md              # API documentation
├── Instagram ER Diagram.pdf
├── .env.example
├── package.json
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Make your changes and commit: `git commit -m "feat: add your feature"`
4. Push to your branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

Please follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for commit messages.

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](./LICENSE) file for details.

---

<div align="center">
  Made with ❤️ for thrift creators & indie stores on Instagram
</div>
