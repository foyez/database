# Chapter 7: Real-World Projects
**Complete Implementations with SQL**

---

## Table of Contents
- [7.1 E-Commerce Platform (PostgreSQL)](#71-e-commerce-platform-postgresql)
  - [7.1.1 Requirements Analysis](#711-requirements-analysis)
  - [7.1.2 Schema Design](#712-schema-design)
  - [7.1.3 Common Queries](#713-common-queries)
  - [7.1.4 Transaction Examples](#714-transaction-examples)
  - [Practice Questions](#practice-questions)

---

## 7.1 E-Commerce Platform (PostgreSQL)

### Project Overview

**Features:**
- User registration and authentication
- Product catalog with categories
- Shopping cart
- Order processing
- Payment tracking
- Inventory management
- Reviews and ratings
- Order history

**Why PostgreSQL?**
- ✅ ACID transactions (critical for payments)
- ✅ Complex relationships (users, orders, products)
- ✅ Strong consistency (inventory management)
- ✅ Rich query capabilities (filtering, search)

---

## 7.1.1 Requirements Analysis

### Functional Requirements

**User Management:**
```
- Users can register and login
- Users have profiles with shipping addresses
- Users can have multiple addresses
- Users can save payment methods
```

**Product Management:**
```
- Products belong to categories
- Products have variants (size, color)
- Products have inventory tracking
- Products can have multiple images
- Products have reviews and ratings
```

**Shopping Cart:**
```
- Users can add/remove items
- Cart persists across sessions
- Cart shows real-time pricing
- Cart validates inventory
```

**Order Processing:**
```
- Create order from cart
- Process payment
- Update inventory
- Generate order confirmation
- Track order status
- Support refunds/cancellations
```

**Business Rules:**
```
- Cannot order out-of-stock items
- Prices are stored with orders (price changes don't affect past orders)
- Inventory decremented only on successful payment
- One active cart per user
- Orders have audit trail
```

---

### Non-Functional Requirements

**Performance:**
```
- Product search: < 100ms
- Checkout process: < 2 seconds
- Support 10,000 concurrent users
- Handle 1000 orders/minute (Black Friday)
```

**Data Integrity:**
```
- No overselling (inventory accurate)
- Payment idempotency (no double charges)
- Referential integrity (no orphaned records)
```

**Security:**
```
- Encrypted passwords
- PCI compliance for payments
- Row-level security for user data
- Audit logging for financial transactions
```

---

## 7.1.2 Schema Design

### Entity Relationship Diagram

```
Users (1) ──< (M) Addresses
Users (1) ──< (M) Orders
Users (1) ──< (M) Reviews
Users (1) ──< (1) Carts

Categories (1) ──< (M) Products
Products (1) ──< (M) Product_Variants
Products (1) ──< (M) Reviews
Products (1) ──< (M) Cart_Items

Orders (1) ──< (M) Order_Items
Orders (M) ──> (1) Addresses (shipping)
Orders (M) ──> (1) Payment_Methods

Carts (1) ──< (M) Cart_Items
Cart_Items (M) ──> (1) Product_Variants
```

---

### Complete Schema

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  phone VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  email_verified BOOLEAN DEFAULT false
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Addresses table
CREATE TABLE addresses (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  address_type VARCHAR(20) CHECK (address_type IN ('shipping', 'billing')),
  street_address VARCHAR(255) NOT NULL,
  city VARCHAR(100) NOT NULL,
  state VARCHAR(100) NOT NULL,
  postal_code VARCHAR(20) NOT NULL,
  country VARCHAR(100) NOT NULL,
  is_default BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_addresses_user_id ON addresses(user_id);

-- Categories table
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(100) NOT NULL UNIQUE,
  slug VARCHAR(100) NOT NULL UNIQUE,
  description TEXT,
  parent_id UUID REFERENCES categories(id) ON DELETE CASCADE,
  image_url VARCHAR(500),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);

-- Products table
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL UNIQUE,
  description TEXT,
  base_price DECIMAL(10, 2) NOT NULL CHECK (base_price >= 0),
  sku VARCHAR(100) UNIQUE,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_sku ON products(sku);
CREATE INDEX idx_products_name ON products USING gin(to_tsvector('english', name));

-- Product variants (size, color, etc.)
CREATE TABLE product_variants (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  sku VARCHAR(100) UNIQUE NOT NULL,
  variant_name VARCHAR(100) NOT NULL,  -- "Large / Red"
  price_adjustment DECIMAL(10, 2) DEFAULT 0,
  stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_product_variants_product_id ON product_variants(product_id);
CREATE INDEX idx_product_variants_sku ON product_variants(sku);

-- Product images
CREATE TABLE product_images (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  image_url VARCHAR(500) NOT NULL,
  alt_text VARCHAR(255),
  display_order INTEGER DEFAULT 0,
  is_primary BOOLEAN DEFAULT false
);

CREATE INDEX idx_product_images_product_id ON product_images(product_id);

-- Reviews
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  title VARCHAR(255),
  comment TEXT,
  is_verified_purchase BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(product_id, user_id)  -- One review per product per user
);

CREATE INDEX idx_reviews_product_id ON reviews(product_id);
CREATE INDEX idx_reviews_user_id ON reviews(user_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);

-- Carts
CREATE TABLE carts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID UNIQUE NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_carts_user_id ON carts(user_id);

-- Cart items
CREATE TABLE cart_items (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cart_id UUID NOT NULL REFERENCES carts(id) ON DELETE CASCADE,
  product_variant_id UUID NOT NULL REFERENCES product_variants(id) ON DELETE CASCADE,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(cart_id, product_variant_id)
);

CREATE INDEX idx_cart_items_cart_id ON cart_items(cart_id);

-- Payment methods
CREATE TABLE payment_methods (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  method_type VARCHAR(20) CHECK (method_type IN ('credit_card', 'debit_card', 'paypal')),
  -- Never store full card numbers in production!
  last_four VARCHAR(4),
  card_brand VARCHAR(20),
  expiry_month INTEGER,
  expiry_year INTEGER,
  is_default BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_payment_methods_user_id ON payment_methods(user_id);

-- Orders
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  order_number VARCHAR(50) UNIQUE NOT NULL,
  status VARCHAR(20) DEFAULT 'pending' CHECK (status IN 
    ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded')),
  
  -- Pricing
  subtotal DECIMAL(10, 2) NOT NULL,
  tax DECIMAL(10, 2) NOT NULL,
  shipping_cost DECIMAL(10, 2) NOT NULL,
  total DECIMAL(10, 2) NOT NULL,
  
  -- Shipping
  shipping_address_id UUID NOT NULL REFERENCES addresses(id),
  
  -- Payment
  payment_method_id UUID NOT NULL REFERENCES payment_methods(id),
  payment_status VARCHAR(20) DEFAULT 'pending' CHECK (payment_status IN 
    ('pending', 'authorized', 'captured', 'failed', 'refunded')),
  payment_transaction_id VARCHAR(100),
  
  -- Timestamps
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  shipped_at TIMESTAMP,
  delivered_at TIMESTAMP,
  cancelled_at TIMESTAMP
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_order_number ON orders(order_number);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);

-- Order items (snapshot of products at time of order)
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE RESTRICT,
  product_variant_id UUID NOT NULL REFERENCES product_variants(id) ON DELETE RESTRICT,
  
  -- Snapshot of product data at time of order
  product_name VARCHAR(255) NOT NULL,
  variant_name VARCHAR(100) NOT NULL,
  sku VARCHAR(100) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,  -- Price at time of order
  
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  subtotal DECIMAL(10, 2) NOT NULL
);

CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- Inventory transactions (audit trail)
CREATE TABLE inventory_transactions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  product_variant_id UUID NOT NULL REFERENCES product_variants(id),
  transaction_type VARCHAR(20) CHECK (transaction_type IN 
    ('purchase', 'sale', 'return', 'adjustment')),
  quantity_change INTEGER NOT NULL,
  previous_quantity INTEGER NOT NULL,
  new_quantity INTEGER NOT NULL,
  order_id UUID REFERENCES orders(id),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_inventory_transactions_variant_id ON inventory_transactions(product_variant_id);
CREATE INDEX idx_inventory_transactions_created_at ON inventory_transactions(created_at);
```

---

### Triggers for Automatic Updates

```sql
-- Update updated_at timestamp automatically
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER products_updated_at
  BEFORE UPDATE ON products
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER carts_updated_at
  BEFORE UPDATE ON carts
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER orders_updated_at
  BEFORE UPDATE ON orders
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at();
```

---

### Views for Common Queries

```sql
-- Product catalog with reviews
CREATE VIEW product_catalog AS
SELECT 
  p.id,
  p.name,
  p.slug,
  p.description,
  p.base_price,
  c.name AS category_name,
  c.slug AS category_slug,
  COUNT(DISTINCT r.id) AS review_count,
  COALESCE(AVG(r.rating), 0) AS avg_rating,
  COUNT(DISTINCT pv.id) AS variant_count,
  SUM(pv.stock_quantity) AS total_stock,
  pi.image_url AS primary_image
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN reviews r ON p.id = r.product_id
LEFT JOIN product_variants pv ON p.id = pv.product_id AND pv.is_active = true
LEFT JOIN product_images pi ON p.id = pi.product_id AND pi.is_primary = true
WHERE p.is_active = true
GROUP BY p.id, p.name, p.slug, p.description, p.base_price, 
         c.name, c.slug, pi.image_url;

-- User order history
CREATE VIEW user_order_history AS
SELECT 
  o.id AS order_id,
  o.user_id,
  o.order_number,
  o.status,
  o.total,
  o.created_at,
  COUNT(oi.id) AS item_count,
  STRING_AGG(oi.product_name, ', ') AS products
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
GROUP BY o.id, o.user_id, o.order_number, o.status, o.total, o.created_at;

-- Cart summary
CREATE VIEW cart_summary AS
SELECT 
  c.id AS cart_id,
  c.user_id,
  COUNT(ci.id) AS item_count,
  SUM(ci.quantity) AS total_items,
  SUM((p.base_price + pv.price_adjustment) * ci.quantity) AS subtotal
FROM carts c
LEFT JOIN cart_items ci ON c.id = ci.cart_id
LEFT JOIN product_variants pv ON ci.product_variant_id = pv.id
LEFT JOIN products p ON pv.product_id = p.id
GROUP BY c.id, c.user_id;
```

---

### Sample Data

```sql
-- Insert sample categories
INSERT INTO categories (name, slug, description) VALUES
  ('Electronics', 'electronics', 'Electronic devices and accessories'),
  ('Clothing', 'clothing', 'Fashion and apparel'),
  ('Books', 'books', 'Books and literature'),
  ('Home & Garden', 'home-garden', 'Home improvement and gardening');

-- Insert sample products
INSERT INTO products (category_id, name, slug, description, base_price, sku)
SELECT 
  (SELECT id FROM categories WHERE slug = 'electronics'),
  'Wireless Headphones',
  'wireless-headphones',
  'High-quality wireless headphones with noise cancellation',
  99.99,
  'ELEC-HEAD-001'
UNION ALL
SELECT 
  (SELECT id FROM categories WHERE slug = 'clothing'),
  'Cotton T-Shirt',
  'cotton-tshirt',
  'Comfortable 100% cotton t-shirt',
  19.99,
  'CLOTH-TSHIRT-001';

-- Insert product variants
INSERT INTO product_variants (product_id, sku, variant_name, price_adjustment, stock_quantity)
SELECT 
  (SELECT id FROM products WHERE slug = 'wireless-headphones'),
  'ELEC-HEAD-001-BLK',
  'Black',
  0,
  100
UNION ALL
SELECT 
  (SELECT id FROM products WHERE slug = 'wireless-headphones'),
  'ELEC-HEAD-001-WHT',
  'White',
  5,
  75
UNION ALL
SELECT 
  (SELECT id FROM products WHERE slug = 'cotton-tshirt'),
  'CLOTH-TSHIRT-001-S-BLU',
  'Small / Blue',
  0,
  50
UNION ALL
SELECT 
  (SELECT id FROM products WHERE slug = 'cotton-tshirt'),
  'CLOTH-TSHIRT-001-M-BLU',
  'Medium / Blue',
  0,
  100;

-- Insert sample user
INSERT INTO users (email, password_hash, first_name, last_name, phone, email_verified)
VALUES (
  'alice@example.com',
  crypt('password123', gen_salt('bf')),
  'Alice',
  'Johnson',
  '+1-555-0100',
  true
);

-- Insert address
INSERT INTO addresses (user_id, address_type, street_address, city, state, postal_code, country, is_default)
SELECT 
  id,
  'shipping',
  '123 Main St',
  'New York',
  'NY',
  '10001',
  'USA',
  true
FROM users WHERE email = 'alice@example.com';
```

---

## 7.1.3 Common Queries

### User Registration and Authentication

```javascript
const { Pool } = require('pg');
const bcrypt = require('bcrypt');

const pool = new Pool({ /* config */ });

// Register new user
async function registerUser(email, password, firstName, lastName) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // Hash password
    const passwordHash = await bcrypt.hash(password, 12);
    
    // Insert user
    const userResult = await client.query(`
      INSERT INTO users (email, password_hash, first_name, last_name)
      VALUES ($1, $2, $3, $4)
      RETURNING id, email, first_name, last_name, created_at
    `, [email, passwordHash, firstName, lastName]);
    
    const user = userResult.rows[0];
    
    // Create cart for user
    await client.query(`
      INSERT INTO carts (user_id)
      VALUES ($1)
    `, [user.id]);
    
    await client.query('COMMIT');
    
    return user;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// Login
async function loginUser(email, password) {
  const result = await pool.query(`
    SELECT id, email, password_hash, first_name, last_name
    FROM users
    WHERE email = $1 AND is_active = true
  `, [email]);
  
  if (result.rows.length === 0) {
    throw new Error('Invalid credentials');
  }
  
  const user = result.rows[0];
  const isValid = await bcrypt.compare(password, user.password_hash);
  
  if (!isValid) {
    throw new Error('Invalid credentials');
  }
  
  // Update last login
  await pool.query(`
    UPDATE users SET last_login = CURRENT_TIMESTAMP WHERE id = $1
  `, [user.id]);
  
  // Don't return password hash
  delete user.password_hash;
  return user;
}
```

---

### Product Catalog Queries

**Get all products with filters:**

```sql
-- Search products by name, filter by category, price range, rating
SELECT 
  p.id,
  p.name,
  p.slug,
  p.description,
  p.base_price,
  c.name AS category_name,
  COALESCE(AVG(r.rating), 0) AS avg_rating,
  COUNT(DISTINCT r.id) AS review_count,
  MIN(p.base_price + pv.price_adjustment) AS min_price,
  MAX(p.base_price + pv.price_adjustment) AS max_price,
  SUM(pv.stock_quantity) AS total_stock,
  pi.image_url AS primary_image
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN reviews r ON p.id = r.product_id
LEFT JOIN product_variants pv ON p.id = pv.product_id AND pv.is_active = true
LEFT JOIN product_images pi ON p.id = pi.product_id AND pi.is_primary = true
WHERE 
  p.is_active = true
  AND ($1::TEXT IS NULL OR p.name ILIKE '%' || $1 || '%')  -- Search term
  AND ($2::UUID IS NULL OR p.category_id = $2)              -- Category filter
  AND ($3::DECIMAL IS NULL OR p.base_price >= $3)           -- Min price
  AND ($4::DECIMAL IS NULL OR p.base_price <= $4)           -- Max price
GROUP BY p.id, p.name, p.slug, p.description, p.base_price, c.name, pi.image_url
HAVING 
  ($5::INTEGER IS NULL OR COALESCE(AVG(r.rating), 0) >= $5)  -- Min rating
ORDER BY 
  CASE WHEN $6 = 'price_asc' THEN p.base_price END ASC,
  CASE WHEN $6 = 'price_desc' THEN p.base_price END DESC,
  CASE WHEN $6 = 'rating' THEN COALESCE(AVG(r.rating), 0) END DESC,
  CASE WHEN $6 = 'popular' THEN COUNT(r.id) END DESC,
  p.created_at DESC
LIMIT $7 OFFSET $8;

-- Parameters:
-- $1: searchTerm (nullable)
-- $2: categoryId (nullable)
-- $3: minPrice (nullable)
-- $4: maxPrice (nullable)
-- $5: minRating (nullable)
-- $6: sortBy ('price_asc', 'price_desc', 'rating', 'popular', 'newest')
-- $7: limit (e.g., 20)
-- $8: offset (e.g., 0 for page 1)
```

**Get product details:**

```sql
-- Complete product information including variants, images, reviews
WITH product_stats AS (
  SELECT 
    product_id,
    COUNT(*) AS review_count,
    AVG(rating) AS avg_rating,
    COUNT(*) FILTER (WHERE rating = 5) AS five_star,
    COUNT(*) FILTER (WHERE rating = 4) AS four_star,
    COUNT(*) FILTER (WHERE rating = 3) AS three_star,
    COUNT(*) FILTER (WHERE rating = 2) AS two_star,
    COUNT(*) FILTER (WHERE rating = 1) AS one_star
  FROM reviews
  GROUP BY product_id
)
SELECT 
  p.id,
  p.name,
  p.slug,
  p.description,
  p.base_price,
  p.sku,
  c.name AS category_name,
  c.slug AS category_slug,
  
  -- Variants
  json_agg(DISTINCT jsonb_build_object(
    'id', pv.id,
    'sku', pv.sku,
    'name', pv.variant_name,
    'price', p.base_price + pv.price_adjustment,
    'stock', pv.stock_quantity,
    'is_active', pv.is_active
  )) FILTER (WHERE pv.id IS NOT NULL) AS variants,
  
  -- Images
  json_agg(DISTINCT jsonb_build_object(
    'url', pi.image_url,
    'alt', pi.alt_text,
    'is_primary', pi.is_primary
  ) ORDER BY pi.display_order) FILTER (WHERE pi.id IS NOT NULL) AS images,
  
  -- Reviews stats
  COALESCE(ps.review_count, 0) AS review_count,
  COALESCE(ps.avg_rating, 0) AS avg_rating,
  COALESCE(ps.five_star, 0) AS five_star,
  COALESCE(ps.four_star, 0) AS four_star,
  COALESCE(ps.three_star, 0) AS three_star,
  COALESCE(ps.two_star, 0) AS two_star,
  COALESCE(ps.one_star, 0) AS one_star
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN product_variants pv ON p.id = pv.product_id
LEFT JOIN product_images pi ON p.id = pi.product_id
LEFT JOIN product_stats ps ON p.id = ps.product_id
WHERE p.slug = $1
GROUP BY p.id, p.name, p.slug, p.description, p.base_price, p.sku,
         c.name, c.slug, ps.review_count, ps.avg_rating,
         ps.five_star, ps.four_star, ps.three_star, ps.two_star, ps.one_star;
```

---

### Shopping Cart Operations

```javascript
// Add item to cart
async function addToCart(userId, productVariantId, quantity) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // Get or create cart
    let cartResult = await client.query(`
      INSERT INTO carts (user_id)
      VALUES ($1)
      ON CONFLICT (user_id) DO UPDATE SET updated_at = CURRENT_TIMESTAMP
      RETURNING id
    `, [userId]);
    
    const cartId = cartResult.rows[0].id;
    
    // Check stock availability
    const stockResult = await client.query(`
      SELECT stock_quantity FROM product_variants WHERE id = $1 AND is_active = true
    `, [productVariantId]);
    
    if (stockResult.rows.length === 0) {
      throw new Error('Product variant not found or inactive');
    }
    
    const availableStock = stockResult.rows[0].stock_quantity;
    if (availableStock < quantity) {
      throw new Error(`Only ${availableStock} items in stock`);
    }
    
    // Add or update cart item
    await client.query(`
      INSERT INTO cart_items (cart_id, product_variant_id, quantity)
      VALUES ($1, $2, $3)
      ON CONFLICT (cart_id, product_variant_id) 
      DO UPDATE SET 
        quantity = cart_items.quantity + EXCLUDED.quantity,
        added_at = CURRENT_TIMESTAMP
    `, [cartId, productVariantId, quantity]);
    
    await client.query('COMMIT');
    
    return { success: true };
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// Get cart contents
async function getCart(userId) {
  const result = await pool.query(`
    SELECT 
      ci.id AS cart_item_id,
      ci.quantity,
      p.id AS product_id,
      p.name AS product_name,
      p.slug AS product_slug,
      pv.id AS variant_id,
      pv.variant_name,
      pv.sku,
      p.base_price + pv.price_adjustment AS price,
      pv.stock_quantity,
      (p.base_price + pv.price_adjustment) * ci.quantity AS subtotal,
      pi.image_url
    FROM carts c
    JOIN cart_items ci ON c.id = ci.cart_id
    JOIN product_variants pv ON ci.product_variant_id = pv.id
    JOIN products p ON pv.product_id = p.id
    LEFT JOIN product_images pi ON p.id = pi.product_id AND pi.is_primary = true
    WHERE c.user_id = $1
    ORDER BY ci.added_at DESC
  `, [userId]);
  
  const items = result.rows;
  const total = items.reduce((sum, item) => sum + parseFloat(item.subtotal), 0);
  
  return {
    items,
    itemCount: items.length,
    total: total.toFixed(2)
  };
}

// Update cart item quantity
async function updateCartItem(cartItemId, quantity) {
  if (quantity <= 0) {
    // Remove item
    await pool.query('DELETE FROM cart_items WHERE id = $1', [cartItemId]);
  } else {
    // Update quantity
    await pool.query(`
      UPDATE cart_items 
      SET quantity = $1 
      WHERE id = $2
    `, [quantity, cartItemId]);
  }
}

// Clear cart
async function clearCart(userId) {
  await pool.query(`
    DELETE FROM cart_items
    WHERE cart_id = (SELECT id FROM carts WHERE user_id = $1)
  `, [userId]);
}
```

---

### Product Reviews

```javascript
// Add review
async function addReview(userId, productId, rating, title, comment) {
  // Check if user purchased product
  const purchaseCheck = await pool.query(`
    SELECT EXISTS (
      SELECT 1 FROM order_items oi
      JOIN orders o ON oi.order_id = o.id
      JOIN product_variants pv ON oi.product_variant_id = pv.id
      WHERE o.user_id = $1 
        AND pv.product_id = $2
        AND o.status IN ('delivered', 'completed')
    ) AS has_purchased
  `, [userId, productId]);
  
  const hasPurchased = purchaseCheck.rows[0].has_purchased;
  
  const result = await pool.query(`
    INSERT INTO reviews (product_id, user_id, rating, title, comment, is_verified_purchase)
    VALUES ($1, $2, $3, $4, $5, $6)
    ON CONFLICT (product_id, user_id) 
    DO UPDATE SET 
      rating = EXCLUDED.rating,
      title = EXCLUDED.title,
      comment = EXCLUDED.comment,
      updated_at = CURRENT_TIMESTAMP
    RETURNING *
  `, [productId, userId, rating, title, comment, hasPurchased]);
  
  return result.rows[0];
}

// Get product reviews
async function getProductReviews(productId, limit = 10, offset = 0) {
  const result = await pool.query(`
    SELECT 
      r.id,
      r.rating,
      r.title,
      r.comment,
      r.is_verified_purchase,
      r.created_at,
      u.first_name,
      u.last_name
    FROM reviews r
    JOIN users u ON r.user_id = u.id
    WHERE r.product_id = $1
    ORDER BY r.created_at DESC
    LIMIT $2 OFFSET $3
  `, [productId, limit, offset]);
  
  return result.rows;
}
```

---

## 7.1.4 Transaction Examples

### Complete Checkout Process

```javascript
async function checkout(userId, shippingAddressId, paymentMethodId) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // 1. Get cart items
    const cartResult = await client.query(`
      SELECT 
        ci.product_variant_id,
        ci.quantity,
        p.name AS product_name,
        pv.variant_name,
        pv.sku,
        p.base_price + pv.price_adjustment AS price,
        pv.stock_quantity
      FROM carts c
      JOIN cart_items ci ON c.id = ci.cart_id
      JOIN product_variants pv ON ci.product_variant_id = pv.id
      JOIN products p ON pv.product_id = p.id
      WHERE c.user_id = $1 AND pv.is_active = true
    `, [userId]);
    
    if (cartResult.rows.length === 0) {
      throw new Error('Cart is empty');
    }
    
    const cartItems = cartResult.rows;
    
    // 2. Validate stock for all items
    for (const item of cartItems) {
      if (item.stock_quantity < item.quantity) {
        throw new Error(
          `Insufficient stock for ${item.product_name}. ` +
          `Available: ${item.stock_quantity}, Requested: ${item.quantity}`
        );
      }
    }
    
    // 3. Calculate totals
    const subtotal = cartItems.reduce(
      (sum, item) => sum + (parseFloat(item.price) * item.quantity), 
      0
    );
    const tax = subtotal * 0.08;  // 8% tax
    const shippingCost = subtotal > 50 ? 0 : 9.99;  // Free shipping over $50
    const total = subtotal + tax + shippingCost;
    
    // 4. Generate order number
    const orderNumber = `ORD-${Date.now()}-${Math.random().toString(36).substr(2, 9).toUpperCase()}`;
    
    // 5. Create order
    const orderResult = await client.query(`
      INSERT INTO orders (
        user_id, order_number, status, subtotal, tax, shipping_cost, total,
        shipping_address_id, payment_method_id, payment_status
      )
      VALUES ($1, $2, 'pending', $3, $4, $5, $6, $7, $8, 'pending')
      RETURNING id, order_number
    `, [userId, orderNumber, subtotal, tax, shippingCost, total, shippingAddressId, paymentMethodId]);
    
    const order = orderResult.rows[0];
    
    // 6. Create order items and update inventory
    for (const item of cartItems) {
      // Insert order item
      await client.query(`
        INSERT INTO order_items (
          order_id, product_variant_id, product_name, variant_name, 
          sku, price, quantity, subtotal
        )
        VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
      `, [
        order.id,
        item.product_variant_id,
        item.product_name,
        item.variant_name,
        item.sku,
        item.price,
        item.quantity,
        parseFloat(item.price) * item.quantity
      ]);
      
      // Update inventory
      const inventoryResult = await client.query(`
        UPDATE product_variants
        SET stock_quantity = stock_quantity - $1
        WHERE id = $2
        RETURNING stock_quantity
      `, [item.quantity, item.product_variant_id]);
      
      const newQuantity = inventoryResult.rows[0].stock_quantity;
      
      // Log inventory transaction
      await client.query(`
        INSERT INTO inventory_transactions (
          product_variant_id, transaction_type, quantity_change,
          previous_quantity, new_quantity, order_id
        )
        VALUES ($1, 'sale', $2, $3, $4, $5)
      `, [
        item.product_variant_id,
        -item.quantity,
        newQuantity + item.quantity,
        newQuantity,
        order.id
      ]);
    }
    
    // 7. Process payment (simplified - would use Stripe/PayPal in production)
    const paymentResult = await processPayment(total, paymentMethodId);
    
    if (!paymentResult.success) {
      throw new Error('Payment failed: ' + paymentResult.error);
    }
    
    // 8. Update order with payment details
    await client.query(`
      UPDATE orders
      SET 
        payment_status = 'captured',
        payment_transaction_id = $1,
        status = 'processing'
      WHERE id = $2
    `, [paymentResult.transactionId, order.id]);
    
    // 9. Clear cart
    await client.query(`
      DELETE FROM cart_items
      WHERE cart_id = (SELECT id FROM carts WHERE user_id = $1)
    `, [userId]);
    
    await client.query('COMMIT');
    
    return {
      success: true,
      orderId: order.id,
      orderNumber: order.order_number,
      total: total.toFixed(2)
    };
    
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// Mock payment processing (replace with Stripe/PayPal)
async function processPayment(amount, paymentMethodId) {
  // In production, integrate with payment gateway
  return {
    success: true,
    transactionId: `TXN-${Date.now()}`
  };
}
```

---

### Order Cancellation with Refund

```javascript
async function cancelOrder(orderId, userId) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // 1. Verify order belongs to user and is cancellable
    const orderResult = await client.query(`
      SELECT status, payment_status, total
      FROM orders
      WHERE id = $1 AND user_id = $2
    `, [orderId, userId]);
    
    if (orderResult.rows.length === 0) {
      throw new Error('Order not found');
    }
    
    const order = orderResult.rows[0];
    
    if (!['pending', 'processing'].includes(order.status)) {
      throw new Error('Order cannot be cancelled');
    }
    
    // 2. Get order items
    const itemsResult = await client.query(`
      SELECT product_variant_id, quantity
      FROM order_items
      WHERE order_id = $1
    `, [orderId]);
    
    // 3. Restore inventory
    for (const item of itemsResult.rows) {
      const inventoryResult = await client.query(`
        UPDATE product_variants
        SET stock_quantity = stock_quantity + $1
        WHERE id = $2
        RETURNING stock_quantity
      `, [item.quantity, item.product_variant_id]);
      
      const newQuantity = inventoryResult.rows[0].stock_quantity;
      
      // Log inventory transaction
      await client.query(`
        INSERT INTO inventory_transactions (
          product_variant_id, transaction_type, quantity_change,
          previous_quantity, new_quantity, order_id, notes
        )
        VALUES ($1, 'return', $2, $3, $4, $5, 'Order cancelled')
      `, [
        item.product_variant_id,
        item.quantity,
        newQuantity - item.quantity,
        newQuantity,
        orderId
      ]);
    }
    
    // 4. Process refund if payment was captured
    if (order.payment_status === 'captured') {
      // In production, process refund via payment gateway
      // await processRefund(order.total, order.payment_transaction_id);
    }
    
    // 5. Update order status
    await client.query(`
      UPDATE orders
      SET 
        status = 'cancelled',
        payment_status = 'refunded',
        cancelled_at = CURRENT_TIMESTAMP
      WHERE id = $1
    `, [orderId]);
    
    await client.query('COMMIT');
    
    return { success: true, message: 'Order cancelled successfully' };
    
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

### Inventory Adjustment

```javascript
async function adjustInventory(productVariantId, quantityChange, reason) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // Get current quantity
    const result = await client.query(`
      SELECT stock_quantity FROM product_variants WHERE id = $1
    `, [productVariantId]);
    
    if (result.rows.length === 0) {
      throw new Error('Product variant not found');
    }
    
    const previousQuantity = result.rows[0].stock_quantity;
    const newQuantity = previousQuantity + quantityChange;
    
    if (newQuantity < 0) {
      throw new Error('Adjustment would result in negative inventory');
    }
    
    // Update inventory
    await client.query(`
      UPDATE product_variants
      SET stock_quantity = $1
      WHERE id = $2
    `, [newQuantity, productVariantId]);
    
    // Log transaction
    await client.query(`
      INSERT INTO inventory_transactions (
        product_variant_id, transaction_type, quantity_change,
        previous_quantity, new_quantity, notes
      )
      VALUES ($1, 'adjustment', $2, $3, $4, $5)
    `, [productVariantId, quantityChange, previousQuantity, newQuantity, reason]);
    
    await client.query('COMMIT');
    
    return {
      success: true,
      previousQuantity,
      newQuantity
    };
    
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

---

### Practice Questions

<details>
<summary><strong>View Questions</strong></summary>

#### Fill in the Blanks

1. E-commerce platforms use PostgreSQL for ________ transactions to ensure payment consistency.
2. The ________ table stores a snapshot of product data at the time of order to preserve historical pricing.
3. ________ ________ ________ ensures that inventory quantities are always non-negative.
4. A ________ is used to automatically update the updated_at timestamp on record changes.
5. The cart_id and product_variant_id have a ________ constraint to prevent duplicate items.

<details>
<summary><strong>View Answers</strong></summary>

1. ACID
2. order_items
3. CHECK constraint (or constraint)
4. trigger
5. UNIQUE

</details>

#### Multiple Choice

**Q1: Why is it important to create order_items with a snapshot of product data?**

    A) To save database space
    B) To preserve historical pricing and product details
    C) To improve query performance
    D) To enable product deletion

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - To preserve historical pricing and product details**

Explanation: Product prices and details may change over time. Storing a snapshot ensures that orders always reflect the price and product information at the time of purchase, which is critical for financial records and customer service.

</details>

---

**Q2: What happens if stock validation fails during checkout?**

    A) The order is created with partial items
    B) The transaction is rolled back and no changes are made
    C) Only the available quantity is ordered
    D) The payment is processed anyway

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - The transaction is rolled back and no changes are made**

Explanation: The checkout process uses a database transaction. If any step fails (including stock validation), ROLLBACK ensures all changes are reverted and the database remains in a consistent state.

</details>

</details>

---