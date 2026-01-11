# Chapter 6: Production Best Practices
**Security, Monitoring, Backup, and Testing**

---

## Table of Contents
- [6.1 Database Security](#61-database-security)
  - [6.1.1 Principle of Least Privilege](#611-principle-of-least-privilege)
  - [6.1.2 SQL Injection Prevention](#612-sql-injection-prevention)
  - [6.1.3 Password Security](#613-password-security)
  - [6.1.4 Encryption (At Rest & In Transit)](#614-encryption-at-rest--in-transit)
  - [6.1.5 Environment Variables & Secrets](#615-environment-variables--secrets)
- [6.2 Monitoring & Observability](#62-monitoring--observability)
  - [6.2.1 Key Metrics](#621-key-metrics)
  - [6.2.2 Monitoring Tools](#622-monitoring-tools)
  - [6.2.3 Alert Configuration](#623-alert-configuration)
- [6.3 Backup & Recovery](#63-backup--recovery)
  - [6.3.1 Backup Strategies](#631-backup-strategies)
  - [6.3.2 Recovery Scenarios](#632-recovery-scenarios)
  - [6.3.3 Best Practices](#633-best-practices)
- [6.4 Database Testing](#64-database-testing)
  - [6.4.1 Unit Testing](#641-unit-testing)
  - [6.4.2 Integration Testing](#642-integration-testing)
  - [6.4.3 Load Testing](#643-load-testing)
- [6.5 Database Migrations](#65-database-migrations)

---

## 6.1 Database Security

### Why Database Security Matters

**Real-World Breaches:**
```
2017: Equifax - 147M records exposed (SQL injection)
2019: Capital One - 100M records exposed (misconfigured firewall)
2021: T-Mobile - 50M records exposed (compromised credentials)

Cost: $4.24M average per breach (IBM 2021)
```

**Security Layers:**
```
Application Layer → [Authentication, Input Validation]
Network Layer → [Firewall, VPN, TLS]
Database Layer → [Access Control, Encryption, Auditing]
Physical Layer → [Data Center Security]
```

---

## 6.1.1 Principle of Least Privilege

**Definition:** Give users/applications only the minimum permissions needed.

---

### PostgreSQL User Roles

**Bad Practice:**
```sql
-- ❌ Application using superuser (DANGEROUS!)
CREATE USER app_user WITH SUPERUSER PASSWORD 'password';

-- Superuser can:
-- - Drop databases
-- - Delete all data
-- - Modify system tables
-- - Create new superusers
```

**Good Practice:**
```sql
-- ✅ Create role with minimal permissions
CREATE ROLE app_readonly;
GRANT CONNECT ON DATABASE mydb TO app_readonly;
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

-- Create user with readonly role
CREATE USER app_reader WITH PASSWORD 'strong_password';
GRANT app_readonly TO app_reader;

-- ✅ Create role for application (read/write on specific tables)
CREATE ROLE app_readwrite;
GRANT CONNECT ON DATABASE mydb TO app_readwrite;
GRANT USAGE ON SCHEMA public TO app_readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON users, orders, products TO app_readwrite;

CREATE USER app_user WITH PASSWORD 'strong_password';
GRANT app_readwrite TO app_user;

-- ✅ Create role for reporting (read-only analytics)
CREATE ROLE analytics_readonly;
GRANT CONNECT ON DATABASE mydb TO analytics_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_readonly;

CREATE USER analytics_user WITH PASSWORD 'strong_password';
GRANT analytics_readonly TO analytics_user;
```

---

### Row-Level Security (RLS)

**Scenario:** Multi-tenant application where users should only see their own data.

```sql
-- Enable RLS on table
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policy: Users can only see their own orders
CREATE POLICY user_orders_policy ON orders
  FOR SELECT
  USING (user_id = current_user_id());

-- Helper function to get current user ID
CREATE FUNCTION current_user_id() RETURNS INTEGER AS $$
  SELECT current_setting('app.user_id')::INTEGER;
$$ LANGUAGE SQL STABLE;

-- Application sets user context
SET app.user_id = 123;

-- Now queries are automatically filtered
SELECT * FROM orders;
-- Only returns orders where user_id = 123
```

**Advanced RLS:**
```sql
-- Policy for INSERT (users can only create orders for themselves)
CREATE POLICY user_orders_insert_policy ON orders
  FOR INSERT
  WITH CHECK (user_id = current_user_id());

-- Policy for UPDATE (users can only update their own orders)
CREATE POLICY user_orders_update_policy ON orders
  FOR UPDATE
  USING (user_id = current_user_id())
  WITH CHECK (user_id = current_user_id());

-- Policy for DELETE (admins only)
CREATE POLICY admin_orders_delete_policy ON orders
  FOR DELETE
  USING (current_user IN (SELECT username FROM admins));
```

---

### MongoDB User Roles

```javascript
// ❌ Bad: Application using root/admin
use admin
db.createUser({
  user: "app_user",
  pwd: "password",
  roles: ["root"]  // DANGEROUS!
});

// ✅ Good: Minimal permissions
use mydb
db.createUser({
  user: "app_user",
  pwd: "strong_password",
  roles: [
    {
      role: "readWrite",
      db: "mydb"
    }
  ]
});

// ✅ Better: Custom role with specific permissions
use admin
db.createRole({
  role: "appRole",
  privileges: [
    {
      resource: { db: "mydb", collection: "users" },
      actions: ["find", "insert", "update"]
    },
    {
      resource: { db: "mydb", collection: "orders" },
      actions: ["find", "insert", "update", "remove"]
    }
  ],
  roles: []
});

db.createUser({
  user: "app_user",
  pwd: "strong_password",
  roles: ["appRole"]
});

// ✅ Read-only user for analytics
use mydb
db.createUser({
  user: "analytics_user",
  pwd: "strong_password",
  roles: [
    {
      role: "read",
      db: "mydb"
    }
  ]
});
```

---

### Practice Questions: Least Privilege

#### Fill in the Blanks

1. The principle of ________ ________ states users should have only minimum required permissions.
2. PostgreSQL ________ ________ ________ allows filtering rows based on user context.
3. A ________ user in PostgreSQL has unrestricted access and should never be used by applications.
4. In MongoDB, a custom ________ can define specific permissions on collections.
5. ________ users should have only SELECT permissions, never INSERT/UPDATE/DELETE.

<details>
<summary><strong>View Answers</strong></summary>

1. least privilege
2. Row Level Security (RLS)
3. superuser
4. role
5. Read-only (or analytics)

</details>

---

## 6.1.2 SQL Injection Prevention

### What is SQL Injection?

**Attack Example:**
```javascript
// ❌ VULNERABLE CODE
const email = req.body.email;  // User input: "alice@example.com' OR '1'='1"
const query = `SELECT * FROM users WHERE email = '${email}'`;
await db.query(query);

// Resulting SQL:
// SELECT * FROM users WHERE email = 'alice@example.com' OR '1'='1'
// Returns ALL users! 💀
```

**Real Attack:**
```javascript
// Input: "'; DROP TABLE users; --"
const query = `DELETE FROM sessions WHERE token = '${token}'`;

// Resulting SQL:
// DELETE FROM sessions WHERE token = ''; DROP TABLE users; --'
// Deletes entire users table! 💀💀💀
```

---

### Prevention: Parameterized Queries

**PostgreSQL (Node.js with pg):**

```javascript
// ✅ SAFE: Parameterized query
const email = req.body.email;
const result = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);

// PostgreSQL treats $1 as DATA, not SQL code
// Input "alice@example.com' OR '1'='1" is just a string
```

**Prepared Statements:**
```javascript
// ✅ Prepared statement (even safer, cached)
const prepared = {
  name: 'get-user-by-email',
  text: 'SELECT * FROM users WHERE email = $1',
  values: [email]
};

const result = await db.query(prepared);
```

---

**MongoDB (Mongoose):**

```javascript
// ❌ VULNERABLE: String concatenation
const email = req.body.email;  // Input: {"$gt": ""}
const user = await User.findOne({ email: email });

// If email is object {"$gt": ""}, query becomes:
// db.users.findOne({ email: { $gt: "" } })
// Returns first user! 💀

// ✅ SAFE: Validate input type
const email = req.body.email;
if (typeof email !== 'string') {
  throw new Error('Invalid email');
}

const user = await User.findOne({ email: email });

// ✅ SAFER: Use strict schema validation
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    validate: {
      validator: (v) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
      message: 'Invalid email format'
    }
  }
});
```

---

### Input Validation

```javascript
// ✅ Validate and sanitize all inputs
const validator = require('validator');

function validateEmail(email) {
  // Check type
  if (typeof email !== 'string') {
    throw new Error('Email must be a string');
  }
  
  // Check format
  if (!validator.isEmail(email)) {
    throw new Error('Invalid email format');
  }
  
  // Sanitize
  return validator.normalizeEmail(email);
}

function validateUserId(userId) {
  // Check type
  if (typeof userId !== 'string') {
    throw new Error('User ID must be a string');
  }
  
  // Check format (UUID)
  if (!validator.isUUID(userId)) {
    throw new Error('Invalid user ID format');
  }
  
  return userId;
}

// Usage
app.post('/users/:id/email', async (req, res) => {
  try {
    const userId = validateUserId(req.params.id);
    const email = validateEmail(req.body.email);
    
    await db.query(
      'UPDATE users SET email = $1 WHERE id = $2',
      [email, userId]
    );
    
    res.json({ success: true });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
```

---

### ORM Query Builders (Safe by Default)

**Sequelize (PostgreSQL):**
```javascript
// ✅ SAFE: ORM handles escaping
const users = await User.findAll({
  where: {
    email: req.body.email,
    status: 'active'
  }
});

// Generates: SELECT * FROM users WHERE email = $1 AND status = $2
```

**Mongoose (MongoDB):**
```javascript
// ✅ SAFE: Schema validation + type checking
const user = await User.findOne({
  email: req.body.email
});
```

**Prisma:**
```javascript
// ✅ SAFE: Type-safe queries
const users = await prisma.user.findMany({
  where: {
    email: req.body.email
  }
});
```

---

### Defense in Depth

**Layer 1: Input Validation**
```javascript
// Validate type, format, length
if (!validator.isEmail(email)) throw new Error('Invalid email');
if (email.length > 255) throw new Error('Email too long');
```

**Layer 2: Parameterized Queries**
```javascript
// Use $1, $2 placeholders
await db.query('SELECT * FROM users WHERE email = $1', [email]);
```

**Layer 3: Least Privilege**
```sql
-- Application user can't DROP tables
REVOKE DROP ON ALL TABLES FROM app_user;
```

**Layer 4: Database Firewall**
```
-- Block suspicious patterns
-- Monitor for: UNION, DROP, --, /*
```

---

### Practice Questions: SQL Injection

#### True/False

1. String concatenation for SQL queries is safe if you validate the input format.
2. Parameterized queries prevent SQL injection by treating user input as data, not code.
3. ORMs like Sequelize and Mongoose are always safe from injection attacks.
4. MongoDB is immune to injection attacks because it's NoSQL.
5. Prepared statements are safer and faster than regular parameterized queries.

<details>
<summary><strong>View Answers</strong></summary>

1. FALSE (still vulnerable, always use parameterized queries)
2. TRUE
3. FALSE (can still be vulnerable if you use raw queries or bypass validation)
4. FALSE (vulnerable to NoSQL injection if inputs aren't validated)
5. TRUE

</details>

---

#### Coding Challenge: Fix Vulnerable Code

**Identify and fix the security issues:**

```javascript
app.get('/users/search', async (req, res) => {
  const name = req.query.name;
  const role = req.query.role;
  
  const query = `
    SELECT * FROM users 
    WHERE name LIKE '%${name}%' 
    AND role = '${role}'
  `;
  
  const users = await db.query(query);
  res.json(users.rows);
});
```

<details>
<summary><strong>View Solution</strong></summary>

**Issues:**
1. ❌ String concatenation (SQL injection vulnerable)
2. ❌ No input validation
3. ❌ No sanitization
4. ❌ Returns all columns (information leakage)
5. ❌ No rate limiting

**Fixed Code:**
```javascript
const validator = require('validator');

app.get('/users/search', 
  rateLimiter({ windowMs: 60000, max: 100 }),  // Rate limit
  async (req, res) => {
    try {
      // Validate inputs
      const name = req.query.name;
      const role = req.query.role;
      
      if (typeof name !== 'string' || name.length > 100) {
        return res.status(400).json({ error: 'Invalid name' });
      }
      
      if (typeof role !== 'string' || !['admin', 'user', 'guest'].includes(role)) {
        return res.status(400).json({ error: 'Invalid role' });
      }
      
      // Sanitize
      const sanitizedName = validator.escape(name);
      
      // Parameterized query
      const result = await db.query(
        `SELECT id, name, email, role 
         FROM users 
         WHERE name ILIKE $1 AND role = $2
         LIMIT 100`,
        [`%${sanitizedName}%`, role]
      );
      
      res.json(result.rows);
      
    } catch (error) {
      console.error('Search error:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  }
);
```

**Security Improvements:**
1. ✅ Parameterized query ($1, $2)
2. ✅ Input validation (type, length, whitelist)
3. ✅ Input sanitization (escape)
4. ✅ Limited columns (id, name, email, role only)
5. ✅ LIMIT clause (prevent large result sets)
6. ✅ Rate limiting
7. ✅ Error handling (don't expose details)

</details>

---

## 6.1.3 Password Security

### Password Hashing

**❌ Never Store Passwords in Plain Text:**
```sql
-- NEVER DO THIS!
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  password VARCHAR(255)  -- Plain text password 💀
);

INSERT INTO users (email, password) VALUES ('alice@example.com', 'password123');
-- If database is breached, all passwords are exposed!
```

**✅ Always Hash Passwords:**
```javascript
const bcrypt = require('bcrypt');

// Hash password before storing
async function createUser(email, password) {
  const saltRounds = 12;
  const hashedPassword = await bcrypt.hash(password, saltRounds);
  
  await db.query(
    'INSERT INTO users (email, password_hash) VALUES ($1, $2)',
    [email, hashedPassword]
  );
}

// Verify password
async function verifyPassword(email, password) {
  const result = await db.query(
    'SELECT password_hash FROM users WHERE email = $1',
    [email]
  );
  
  if (result.rows.length === 0) {
    return false;
  }
  
  const hashedPassword = result.rows[0].password_hash;
  return await bcrypt.compare(password, hashedPassword);
}
```

---

### Password Hashing Algorithms

**Comparison:**

| Algorithm | Speed | Security | Recommendation |
|-----------|-------|----------|----------------|
| MD5 | Very Fast | ❌ Broken | Never use |
| SHA-256 | Fast | ❌ Too fast (brute-force) | Never for passwords |
| bcrypt | Slow | ✅ Good | Recommended |
| Argon2 | Configurable | ✅ Best | Best choice |
| PBKDF2 | Slow | ✅ Good | Acceptable |

**Why Slow is Good:**
```
Fast hash (SHA-256):
- Attacker: 10 billion hashes/second
- Brute force 8-char password: < 1 hour

Slow hash (bcrypt):
- Attacker: 10,000 hashes/second
- Brute force 8-char password: > 1,000 years
```

---

### Argon2 (Best Practice)

```javascript
const argon2 = require('argon2');

// Hash password
async function hashPassword(password) {
  return await argon2.hash(password, {
    type: argon2.argon2id,  // Hybrid (best)
    memoryCost: 65536,      // 64 MB
    timeCost: 3,            // 3 iterations
    parallelism: 4          // 4 threads
  });
}

// Verify password
async function verifyPassword(hash, password) {
  try {
    return await argon2.verify(hash, password);
  } catch (error) {
    return false;
  }
}

// Usage
const hash = await hashPassword('user_password');
// $argon2id$v=19$m=65536,t=3,p=4$...

const isValid = await verifyPassword(hash, 'user_password');
// true
```

---

### Password Policy

```javascript
function validatePassword(password) {
  const errors = [];
  
  // Length
  if (password.length < 12) {
    errors.push('Password must be at least 12 characters');
  }
  
  // Uppercase
  if (!/[A-Z]/.test(password)) {
    errors.push('Password must contain at least one uppercase letter');
  }
  
  // Lowercase
  if (!/[a-z]/.test(password)) {
    errors.push('Password must contain at least one lowercase letter');
  }
  
  // Number
  if (!/[0-9]/.test(password)) {
    errors.push('Password must contain at least one number');
  }
  
  // Special character
  if (!/[!@#$%^&*(),.?":{}|<>]/.test(password)) {
    errors.push('Password must contain at least one special character');
  }
  
  // Common passwords (use library like `zxcvbn`)
  const commonPasswords = ['password', '123456', 'qwerty', 'admin'];
  if (commonPasswords.includes(password.toLowerCase())) {
    errors.push('Password is too common');
  }
  
  return {
    valid: errors.length === 0,
    errors
  };
}

// Usage
const result = validatePassword('MyP@ssw0rd123');
if (!result.valid) {
  throw new Error(result.errors.join(', '));
}
```

---

### Password Reset Security

```javascript
const crypto = require('crypto');

// Generate secure reset token
function generateResetToken() {
  return crypto.randomBytes(32).toString('hex');
}

// Request password reset
async function requestPasswordReset(email) {
  const user = await db.query(
    'SELECT id FROM users WHERE email = $1',
    [email]
  );
  
  if (user.rows.length === 0) {
    // Don't reveal if email exists (timing attack prevention)
    return { success: true };
  }
  
  const token = generateResetToken();
  const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
  const expiresAt = new Date(Date.now() + 3600000);  // 1 hour
  
  await db.query(
    `INSERT INTO password_reset_tokens (user_id, token_hash, expires_at)
     VALUES ($1, $2, $3)`,
    [user.rows[0].id, tokenHash, expiresAt]
  );
  
  // Send email with token
  await sendEmail(email, `Reset link: https://example.com/reset?token=${token}`);
  
  return { success: true };
}

// Reset password
async function resetPassword(token, newPassword) {
  const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
  
  const result = await db.query(
    `SELECT user_id FROM password_reset_tokens
     WHERE token_hash = $1 AND expires_at > NOW()
     AND used = FALSE`,
    [tokenHash]
  );
  
  if (result.rows.length === 0) {
    throw new Error('Invalid or expired token');
  }
  
  // Validate new password
  const validation = validatePassword(newPassword);
  if (!validation.valid) {
    throw new Error(validation.errors.join(', '));
  }
  
  // Hash new password
  const passwordHash = await argon2.hash(newPassword);
  
  // Update password
  await db.query('BEGIN');
  
  await db.query(
    'UPDATE users SET password_hash = $1 WHERE id = $2',
    [passwordHash, result.rows[0].user_id]
  );
  
  await db.query(
    'UPDATE password_reset_tokens SET used = TRUE WHERE token_hash = $1',
    [tokenHash]
  );
  
  await db.query('COMMIT');
  
  return { success: true };
}
```

---

### Practice Questions: Password Security

#### Fill in the Blanks

1. ________ is a slow hashing algorithm recommended for password storage.
2. ________ is currently the best password hashing algorithm, winning the Password Hashing Competition.
3. Password reset tokens should have an ________ time (typically 1 hour).
4. Passwords should never be stored in ________ ________ in the database.
5. A strong password policy typically requires at least ________ characters.

<details>
<summary><strong>View Answers</strong></summary>

1. bcrypt
2. Argon2
3. expiration
4. plain text
5. 12 (or 8-10 minimum)

</details>

---

## 6.1.4 Encryption (At Rest & In Transit)

### Why Encryption?

**Scenarios Where Encryption Matters:**
```
1. Database backup stolen from S3
   → Without encryption: All data exposed
   → With encryption: Data unreadable

2. Server hard drive stolen
   → Without encryption: Data extracted
   → With encryption: Data protected

3. Network traffic intercepted (man-in-the-middle)
   → Without TLS: Credentials and data exposed
   → With TLS: Traffic encrypted
```

---

### Encryption At Rest

**Definition:** Encrypting data stored on disk.

---

#### PostgreSQL Encryption At Rest

**Option 1: Full Disk Encryption (OS Level)**

```bash
# Linux LUKS encryption
cryptsetup luksFormat /dev/sdb
cryptsetup open /dev/sdb pgdata
mkfs.ext4 /dev/mapper/pgdata
mount /dev/mapper/pgdata /var/lib/postgresql/data

# All PostgreSQL data automatically encrypted
```

**Pros:** ✅ Transparent, protects entire disk
**Cons:** ❌ Keys in memory, doesn't protect from application-level attacks

---

**Option 2: pgcrypto Extension (Column-Level)**

```sql
-- Enable pgcrypto
CREATE EXTENSION pgcrypto;

-- Create table with encrypted column
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),
  ssn BYTEA  -- Encrypted column
);

-- Encrypt data
INSERT INTO users (email, ssn) VALUES (
  'alice@example.com',
  pgp_sym_encrypt('123-45-6789', 'encryption_key')
);

-- Decrypt data
SELECT 
  email,
  pgp_sym_decrypt(ssn, 'encryption_key') AS ssn
FROM users;

-- Output:
-- email              | ssn
-- alice@example.com  | 123-45-6789
```

**Key Management:**
```sql
-- ❌ Bad: Hardcoded key
pgp_sym_encrypt(data, 'my_secret_key')

-- ✅ Good: Key from environment variable
pgp_sym_encrypt(data, current_setting('app.encryption_key'))

-- Set key at connection time
SET app.encryption_key = 'key_from_env';
```

---

**Option 3: Transparent Data Encryption (PostgreSQL 15+)**

```bash
# Initialize with encryption
initdb -D /var/lib/postgresql/data --data-checksums --wal-log-hints -k

# postgresql.conf
data_encryption = on
```

---

#### MongoDB Encryption At Rest

**WiredTiger Encryption:**

```yaml
# mongod.conf
security:
  enableEncryption: true
  encryptionKeyFile: /path/to/keyfile

storage:
  dbPath: /var/lib/mongodb
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2
```

**Generate Encryption Key:**
```bash
openssl rand -base64 32 > /path/to/keyfile
chmod 600 /path/to/keyfile
chown mongodb:mongodb /path/to/keyfile
```

**Key Management with KMIP:**
```yaml
# mongod.conf with external key manager
security:
  enableEncryption: true
  kmip:
    serverName: kmip.example.com
    port: 5696
    clientCertificateFile: /path/to/client-cert.pem
```

---

**Field-Level Encryption (Client-Side):**

```javascript
const { ClientEncryption } = require('mongodb-client-encryption');

// Setup encryption
const encryption = new ClientEncryption(client, {
  keyVaultNamespace: 'encryption.__keyVault',
  kmsProviders: {
    local: {
      key: Buffer.from(process.env.MASTER_KEY, 'base64')
    }
  }
});

// Create data encryption key
const dataKeyId = await encryption.createDataKey('local');

// Encrypt field
const encryptedSSN = await encryption.encrypt(
  '123-45-6789',
  {
    keyId: dataKeyId,
    algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
  }
);

// Store encrypted data
await db.collection('users').insertOne({
  email: 'alice@example.com',
  ssn: encryptedSSN
});

// Decrypt field
const user = await db.collection('users').findOne({ email: 'alice@example.com' });
const decryptedSSN = await encryption.decrypt(user.ssn);
console.log(decryptedSSN);  // 123-45-6789
```

---

### Encryption In Transit (TLS/SSL)

---

#### PostgreSQL TLS

**Generate SSL Certificates:**

```bash
# Generate self-signed certificate (development only)
openssl req -new -x509 -days 365 -nodes -text \
  -out server.crt \
  -keyout server.key \
  -subj "/CN=postgres.example.com"

chmod 600 server.key
chown postgres:postgres server.key server.crt
```

**Configure PostgreSQL:**

```bash
# postgresql.conf
ssl = on
ssl_cert_file = '/path/to/server.crt'
ssl_key_file = '/path/to/server.key'

# pg_hba.conf (require SSL)
hostssl all all 0.0.0.0/0 md5
```

**Client Connection:**

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: 'postgres.example.com',
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  ssl: {
    rejectUnauthorized: true,
    ca: fs.readFileSync('/path/to/ca.crt').toString(),
  }
});
```

---

#### MongoDB TLS

**Generate Certificates:**

```bash
# Generate CA
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt

# Generate server certificate
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key \
  -set_serial 01 -out server.crt

# Combine for MongoDB
cat server.key server.crt > server.pem
```

**Configure MongoDB:**

```yaml
# mongod.conf
net:
  port: 27017
  tls:
    mode: requireTLS
    certificateKeyFile: /path/to/server.pem
    CAFile: /path/to/ca.crt
```

**Client Connection:**

```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017/mydb', {
  tls: true,
  tlsCAFile: '/path/to/ca.crt',
  tlsCertificateKeyFile: '/path/to/client.pem'
});
```

---

### Key Rotation

**PostgreSQL:**

```sql
-- Rotate encryption key
-- 1. Create new encrypted column
ALTER TABLE users ADD COLUMN ssn_new BYTEA;

-- 2. Re-encrypt with new key
UPDATE users SET ssn_new = pgp_sym_encrypt(
  pgp_sym_decrypt(ssn, 'old_key'),
  'new_key'
);

-- 3. Drop old column, rename new
ALTER TABLE users DROP COLUMN ssn;
ALTER TABLE users RENAME COLUMN ssn_new TO ssn;
```

**MongoDB:**

```javascript
// Rotate data encryption keys
const { ClientEncryption } = require('mongodb-client-encryption');

async function rotateDataKey(oldKeyId) {
  // Create new key
  const newKeyId = await encryption.createDataKey('local');
  
  // Re-encrypt all documents
  const cursor = db.collection('users').find({ encryptedField: { $exists: true } });
  
  while (await cursor.hasNext()) {
    const doc = await cursor.next();
    
    // Decrypt with old key
    const decrypted = await encryption.decrypt(doc.encryptedField);
    
    // Encrypt with new key
    const encrypted = await encryption.encrypt(decrypted, {
      keyId: newKeyId,
      algorithm: 'AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic'
    });
    
    // Update document
    await db.collection('users').updateOne(
      { _id: doc._id },
      { $set: { encryptedField: encrypted } }
    );
  }
  
  // Delete old key
  await db.collection('encryption.__keyVault').deleteOne({ _id: oldKeyId });
}
```

---

### Practice Questions: Encryption

#### Fill in the Blanks

1. ________ ________ ________ encrypts data stored on disk.
2. ________ ________ ________ encrypts data during transmission over the network.
3. PostgreSQL's ________ extension provides column-level encryption.
4. MongoDB's ________ engine supports transparent data encryption.
5. ________ certificates are used to establish encrypted connections.

<details>
<summary><strong>View Answers</strong></summary>

1. Encryption at rest
2. Encryption in transit (or TLS/SSL)
3. pgcrypto
4. WiredTiger
5. TLS/SSL

</details>

---

## 6.1.5 Environment Variables & Secrets

### Why Not Hardcode Secrets?

**❌ Bad Practice:**
```javascript
// NEVER DO THIS!
const dbConfig = {
  host: 'db.example.com',
  user: 'admin',
  password: 'SuperSecret123!',  // 💀 Exposed in code
  database: 'production'
};

// If pushed to Git:
// - Visible in commit history (forever)
// - Accessible to all developers
// - Public if repo is public
```

---

### Environment Variables

**✅ Good Practice:**

```javascript
// Load from environment
require('dotenv').config();

const dbConfig = {
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
};
```

**.env file (NOT in Git):**
```bash
# .env
DB_HOST=db.example.com
DB_USER=app_user
DB_PASSWORD=SuperSecret123!
DB_NAME=production

# Add to .gitignore
echo ".env" >> .gitignore
```

---

### Docker Secrets

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: myapp:latest
    secrets:
      - db_password
    environment:
      DB_HOST: postgres
      DB_USER: app_user
      DB_PASSWORD_FILE: /run/secrets/db_password

  postgres:
    image: postgres:15
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Application Code:**
```javascript
const fs = require('fs');

// Read password from file
const passwordFile = process.env.DB_PASSWORD_FILE;
const password = fs.readFileSync(passwordFile, 'utf8').trim();

const dbConfig = {
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: password,
  database: process.env.DB_NAME
};
```

---

### Kubernetes Secrets

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YXBwX3VzZXI=  # base64 encoded
  password: U3VwZXJTZWNyZXQxMjMh  # base64 encoded

---
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
```

**Create Secret:**
```bash
# From literal
kubectl create secret generic db-credentials \
  --from-literal=username=app_user \
  --from-literal=password=SuperSecret123!

# From file
echo -n 'SuperSecret123!' > password.txt
kubectl create secret generic db-credentials \
  --from-file=password=password.txt

# Verify
kubectl get secret db-credentials -o yaml
```

---

### Secret Management Services

#### AWS Secrets Manager

```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager({ region: 'us-east-1' });

async function getSecret(secretName) {
  const data = await secretsManager.getSecretValue({ SecretId: secretName }).promise();
  
  if ('SecretString' in data) {
    return JSON.parse(data.SecretString);
  }
  
  const buff = Buffer.from(data.SecretBinary, 'base64');
  return JSON.parse(buff.toString('ascii'));
}

// Usage
const dbCredentials = await getSecret('prod/database/credentials');

const dbConfig = {
  host: dbCredentials.host,
  user: dbCredentials.username,
  password: dbCredentials.password,
  database: dbCredentials.database
};
```

---

#### HashiCorp Vault

```javascript
const vault = require('node-vault')({
  endpoint: 'http://vault.example.com:8200',
  token: process.env.VAULT_TOKEN
});

async function getDBCredentials() {
  const result = await vault.read('secret/data/database/prod');
  return result.data.data;
}

// Usage
const credentials = await getDBCredentials();

const dbConfig = {
  host: credentials.host,
  user: credentials.username,
  password: credentials.password,
  database: credentials.database
};
```

**Vault Dynamic Secrets (PostgreSQL):**

```bash
# Configure PostgreSQL secrets engine
vault secrets enable database

vault write database/config/postgresql \
  plugin_name=postgresql-database-plugin \
  allowed_roles="app-role" \
  connection_url="postgresql://{{username}}:{{password}}@postgres:5432/mydb" \
  username="vault" \
  password="vault-password"

# Create role with TTL
vault write database/roles/app-role \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"
```

**Application:**
```javascript
// Get dynamic credentials
const creds = await vault.read('database/creds/app-role');

// Credentials valid for 1 hour, auto-rotated
const dbConfig = {
  host: 'postgres.example.com',
  user: creds.data.username,      // vault-root-1234567890
  password: creds.data.password,  // Random generated password
  database: 'mydb'
};

// Renew lease before expiration
await vault.write('sys/leases/renew', {
  lease_id: creds.lease_id
});
```

---

### Secret Rotation

**Automated Rotation Script:**

```javascript
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

async function rotateSecret(secretName) {
  // 1. Generate new password
  const newPassword = generateSecurePassword();
  
  // 2. Update database user
  await db.query(
    'ALTER USER app_user WITH PASSWORD $1',
    [newPassword]
  );
  
  // 3. Update secret
  await secretsManager.updateSecret({
    SecretId: secretName,
    SecretString: JSON.stringify({
      username: 'app_user',
      password: newPassword
    })
  }).promise();
  
  console.log('Secret rotated successfully');
}

function generateSecurePassword() {
  const crypto = require('crypto');
  return crypto.randomBytes(32).toString('base64');
}

// Schedule rotation (every 90 days)
const cron = require('node-cron');
cron.schedule('0 0 1 */3 *', () => {
  rotateSecret('prod/database/credentials');
});
```

---

### Best Practices

**1. Never Commit Secrets to Git**
```bash
# .gitignore
.env
.env.*
secrets/
*.key
*.pem
config/production.json
```

**2. Use Different Secrets Per Environment**
```bash
# Development
DB_PASSWORD=dev_password_123

# Staging
DB_PASSWORD=staging_password_456

# Production
DB_PASSWORD=prod_password_789_super_secure
```

**3. Rotate Secrets Regularly**
```
- Development: Every 6 months
- Staging: Every 3 months
- Production: Every 90 days (automated)
```

**4. Audit Secret Access**
```javascript
// Log all secret retrievals
async function getSecret(secretName) {
  console.log(`[AUDIT] Secret accessed: ${secretName} by ${process.env.USER}`);
  // ... retrieve secret
}
```

**5. Encrypt Secrets at Rest**
```bash
# Encrypt .env file (development)
gpg --symmetric --cipher-algo AES256 .env

# Decrypt
gpg --decrypt .env.gpg > .env
```

---

### Practice Questions: Secrets Management

#### Multiple Choice

**Q1: Where should database passwords be stored in a production application?**

A) Hardcoded in the source code
B) In a .env file committed to Git
C) In environment variables or a secret manager
D) In a configuration file in the repository

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - In environment variables or a secret manager**

Explanation: Secrets should never be in source code or version control. Use environment variables, Docker secrets, Kubernetes secrets, or dedicated secret managers (AWS Secrets Manager, HashiCorp Vault).

</details>

---

**Q2: What is the main advantage of using HashiCorp Vault's dynamic secrets?**

A) Secrets are stored encrypted
B) Secrets are automatically rotated with short TTLs
C) Secrets can be shared across teams
D) Secrets are backed up automatically

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Secrets are automatically rotated with short TTLs**

Explanation: Vault dynamic secrets are generated on-demand with short time-to-live (TTL), and automatically expire and rotate, reducing the window of exposure if credentials are compromised.

</details>

---

## 6.2 Monitoring & Observability

### Why Monitor Databases?

**Without Monitoring:**
```
10:00 AM - Database slow
10:30 AM - Users complaining
11:00 AM - Database crashes
11:30 AM - You notice the problem
12:00 PM - Start investigation
???     - Find root cause
```

**With Monitoring:**
```
09:55 AM - Alert: CPU at 90%
09:56 AM - Alert: Slow queries detected
09:57 AM - Investigate and fix
10:00 AM - Problem resolved
         - Users unaffected
```

---

## 6.2.1 Key Metrics

### The Four Golden Signals

**1. Latency (Response Time)**
```sql
-- PostgreSQL: Average query time
SELECT 
  mean_exec_time,
  query
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

**Thresholds:**
- ✅ Good: < 100ms
- ⚠️ Warning: 100-500ms
- 🚨 Critical: > 500ms

---

**2. Traffic (Throughput)**
```sql
-- PostgreSQL: Queries per second
SELECT 
  datname,
  xact_commit + xact_rollback AS total_transactions,
  tup_returned AS rows_read,
  tup_inserted + tup_updated + tup_deleted AS rows_modified
FROM pg_stat_database;
```

**MongoDB:**
```javascript
db.serverStatus().opcounters;
// {
//   insert: 12450,
//   query: 45678,
//   update: 8901,
//   delete: 234
// }
```

---

**3. Errors**
```sql
-- PostgreSQL: Connection errors
SELECT 
  datname,
  deadlocks,
  conflicts
FROM pg_stat_database;
```

**Application Logging:**
```javascript
// Log database errors
pool.on('error', (err) => {
  console.error('Database error:', err);
  metrics.increment('db.errors');
});
```

---

**4. Saturation (Resource Usage)**

**CPU:**
```bash
# System CPU
top -b -n 1 | grep postgres

# PostgreSQL query: CPU-intensive queries
SELECT 
  pid,
  state,
  query,
  (now() - query_start) AS duration
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;
```

**Memory:**
```sql
-- PostgreSQL: Buffer cache hit ratio
SELECT 
  sum(heap_blks_hit) / 
  NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0) * 100 
  AS cache_hit_ratio
FROM pg_stattuple_approx('users');

-- Target: > 99%
```

**Disk:**
```bash
# Disk usage
df -h /var/lib/postgresql

# I/O wait
iostat -x 1
```

**Connections:**
```sql
-- PostgreSQL: Connection usage
SELECT 
  count(*) AS current_connections,
  max_conn,
  count(*) * 100.0 / max_conn AS connection_usage_percent
FROM pg_stat_activity,
  (SELECT setting::int AS max_conn FROM pg_settings WHERE name = 'max_connections') mc
GROUP BY max_conn;

-- Alert if > 80%
```

---

### Database-Specific Metrics

#### PostgreSQL

```sql
-- Table bloat
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
  n_dead_tup AS dead_tuples,
  n_live_tup AS live_tuples,
  ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;

-- Alert if dead_ratio > 20%
```

```sql
-- Long-running queries
SELECT 
  pid,
  now() - query_start AS duration,
  state,
  query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;
```

```sql
-- Replication lag
SELECT 
  client_addr,
  state,
  sent_lsn,
  write_lsn,
  flush_lsn,
  replay_lsn,
  (sent_lsn - replay_lsn) AS lag_bytes
FROM pg_stat_replication;

-- Alert if lag_bytes > 1GB
```

---

#### MongoDB

```javascript
// Server status
const status = db.serverStatus();

// Key metrics
console.log({
  connections: status.connections,
  opcounters: status.opcounters,
  mem: status.mem,
  network: status.network
});

// Slow queries
db.setProfilingLevel(2);  // Profile all queries
db.system.profile.find({ millis: { $gt: 100 } }).sort({ ts: -1 }).limit(10);

// Replication lag
rs.printSlaveReplicationInfo();

// Lock percentage
db.serverStatus().globalLock.currentQueue;
```

---

### Application-Level Metrics

```javascript
const client = require('prom-client');

// Connection pool metrics
const poolSizeGauge = new client.Gauge({
  name: 'db_pool_size',
  help: 'Current database connection pool size'
});

const poolIdleGauge = new client.Gauge({
  name: 'db_pool_idle',
  help: 'Idle connections in pool'
});

// Query metrics
const queryDuration = new client.Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query duration',
  labelNames: ['operation', 'table']
});

// Update metrics
setInterval(() => {
  poolSizeGauge.set(pool.totalCount);
  poolIdleGauge.set(pool.idleCount);
}, 5000);

// Measure query duration
async function query(sql, params) {
  const end = queryDuration.startTimer();
  try {
    const result = await pool.query(sql, params);
    end({ operation: 'select', table: 'users' });
    return result;
  } catch (error) {
    end({ operation: 'error', table: 'users' });
    throw error;
  }
}
```

---

### Practice Questions: Monitoring Metrics

#### Fill in the Blanks

1. The four golden signals of monitoring are latency, traffic, errors, and ________.
2. PostgreSQL's ________ ________ ________ ratio should be above 99% for optimal performance.
3. ________ -running queries in PostgreSQL can be identified using pg_stat_activity.
4. MongoDB's ________ level 2 profiles all queries for performance analysis.
5. Connection pool ________ percentage should trigger an alert if it exceeds 80%.

<details>
<summary><strong>View Answers</strong></summary>

1. saturation
2. buffer cache hit
3. Long
4. profiling
5. usage

</details>

---

## 6.2.2 Monitoring Tools

### Prometheus + Grafana

**Architecture:**
```
Application → [Metrics Endpoint] ← Prometheus (scrapes)
                                        ↓
                                    Storage
                                        ↓
                                    Grafana (visualizes)
                                        ↓
                                    Dashboards
```

---

#### Setup Prometheus for PostgreSQL

**1. Install postgres_exporter:**

```bash
# Download and install
wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.11.1/postgres_exporter-0.11.1.linux-amd64.tar.gz
tar xvfz postgres_exporter-0.11.1.linux-amd64.tar.gz
cd postgres_exporter-0.11.1.linux-amd64

# Create monitoring user
psql -U postgres -c "CREATE USER postgres_exporter WITH PASSWORD 'password';"
psql -U postgres -c "GRANT pg_monitor TO postgres_exporter;"

# Run exporter
export DATA_SOURCE_NAME="postgresql://postgres_exporter:password@localhost:5432/postgres?sslmode=disable"
./postgres_exporter
```

**2. Configure Prometheus:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'postgresql'
    static_configs:
      - targets: ['localhost:9187']
```

**3. Create Grafana Dashboard:**

```json
{
  "dashboard": {
    "title": "PostgreSQL Monitoring",
    "panels": [
      {
        "title": "Connections",
        "targets": [
          {
            "expr": "pg_stat_database_numbackends{datname='mydb'}"
          }
        ]
      },
      {
        "title": "Transactions per Second",
        "targets": [
          {
            "expr": "rate(pg_stat_database_xact_commit{datname='mydb'}[1m])"
          }
        ]
      },
      {
        "title": "Cache Hit Ratio",
        "targets": [
          {
            "expr": "pg_stat_database_blks_hit / (pg_stat_database_blks_hit + pg_stat_database_blks_read)"
          }
        ]
      }
    ]
  }
}
```

---

#### Setup Prometheus for MongoDB

**1. Install mongodb_exporter:**

```bash
wget https://github.com/percona/mongodb_exporter/releases/download/v0.37.0/mongodb_exporter-0.37.0.linux-amd64.tar.gz
tar xvfz mongodb_exporter-0.37.0.linux-amd64.tar.gz

# Run exporter
./mongodb_exporter --mongodb.uri="mongodb://localhost:27017"
```

**2. Configure Prometheus:**

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'mongodb'
    static_configs:
      - targets: ['localhost:9216']
```

**3. Key Metrics:**

```promql
# Operations per second
rate(mongodb_op_counters_total[1m])

# Connection count
mongodb_connections{state="current"}

# Memory usage
mongodb_memory{type="resident"}

# Replication lag
mongodb_replset_member_replication_lag
```

---

### Application Performance Monitoring (APM)

#### New Relic

```javascript
// Install
npm install newrelic

// newrelic.js
exports.config = {
  app_name: ['My Application'],
  license_key: 'your_license_key',
  logging: {
    level: 'info'
  }
};

// index.js (first line)
require('newrelic');

// Automatic instrumentation for:
// - PostgreSQL (pg)
// - MongoDB (mongodb)
// - MySQL (mysql2)
// - Database query timing
// - Slow query detection
```

---

#### Datadog

```javascript
// Install
npm install dd-trace

// index.js
const tracer = require('dd-trace').init({
  hostname: 'datadog-agent',
  port: 8126,
  service: 'myapp',
  env: 'production'
});

// Automatic PostgreSQL tracing
const { Pool } = require('pg');
const pool = new Pool(/* config */);

// All queries automatically traced
const result = await pool.query('SELECT * FROM users WHERE id = $1', [123]);
```

---

### Database-Specific Tools

#### pgAdmin (PostgreSQL)

```bash
# Install via Docker
docker run -p 5050:80 \
  -e PGADMIN_DEFAULT_EMAIL=admin@example.com \
  -e PGADMIN_DEFAULT_PASSWORD=admin \
  dpage/pgadmin4

# Access: http://localhost:5050
```

**Features:**
- Query tool with EXPLAIN visualization
- Server activity dashboard
- Lock monitoring
- Table statistics

---

#### MongoDB Ops Manager

```yaml
# Docker Compose
version: '3.8'
services:
  ops-manager:
    image: mongodb/mongodb-ops-manager:latest
    ports:
      - "8080:8080"
    environment:
      - MMS_CENTRAL_URL=http://localhost:8080
```

**Features:**
- Real-time performance metrics
- Query profiler
- Index suggestions
- Automated backups

---

### Log Aggregation

#### ELK Stack (Elasticsearch, Logstash, Kibana)

**1. Configure PostgreSQL Logging:**

```bash
# postgresql.conf
log_destination = 'csvlog'
logging_collector = on
log_directory = '/var/log/postgresql'
log_filename = 'postgresql-%Y-%m-%d.log'
log_rotation_age = 1d
log_min_duration_statement = 100  # Log queries > 100ms
```

**2. Logstash Configuration:**

```ruby
# logstash.conf
input {
  file {
    path => "/var/log/postgresql/*.log"
    type => "postgresql"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
}

filter {
  if [type] == "postgresql" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp},%{DATA:user},%{DATA:database},%{NUMBER:pid},%{DATA:connection},%{DATA:session},%{DATA:transaction},%{DATA:error_severity},%{DATA:sql_state},%{GREEDYDATA:message}" 
      }
    }
    
    if [sql_state] == "00000" {
      mutate { add_tag => ["success"] }
    } else {
      mutate { add_tag => ["error"] }
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "postgresql-%{+YYYY.MM.dd}"
  }
}
```

**3. Kibana Visualizations:**

```json
{
  "title": "PostgreSQL Errors by Type",
  "type": "pie",
  "query": {
    "query_string": {
      "query": "type:postgresql AND tags:error"
    }
  },
  "aggs": {
    "error_types": {
      "terms": {
        "field": "error_severity.keyword"
      }
    }
  }
}
```

---

### Practice Questions: Monitoring Tools

#### Fill in the Blanks

1. ________ is a time-series database commonly used with Grafana for visualization.
2. ________ is an exporter that exposes PostgreSQL metrics for Prometheus.
3. ________ APM tools provide automatic instrumentation for database queries.
4. The ________ stack (Elasticsearch, Logstash, Kibana) is used for log aggregation.
5. MongoDB ________ ________ provides real-time performance metrics and automated backups.

<details>
<summary><strong>View Answers</strong></summary>

1. Prometheus
2. postgres_exporter
3. Application Performance Monitoring (APM) or New Relic/Datadog
4. ELK
5. Ops Manager

</details>

---

## 6.2.3 Alert Configuration

### Alert Principles

**Good Alert:**
```
✅ Actionable (you can do something about it)
✅ Urgent (requires immediate attention)
✅ Specific (clear what's wrong)
✅ Rare (not noisy)
```

**Bad Alert:**
```
❌ "Database is slow" (vague)
❌ Daily disk usage report (not urgent)
❌ Metric fluctuations within normal range (noisy)
```

---

### Prometheus Alerting

**1. Define Alert Rules:**

```yaml
# alerts.yml
groups:
  - name: postgresql
    interval: 30s
    rules:
      # High connection usage
      - alert: PostgreSQLHighConnections
        expr: |
          (pg_stat_database_numbackends / 
           (SELECT setting FROM pg_settings WHERE name = 'max_connections')) 
          > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "PostgreSQL connection usage above 80%"
          description: "Database {{ $labels.datname }} has {{ $value }}% connections used"
      
      # Slow queries
      - alert: PostgreSQLSlowQueries
        expr: |
          rate(pg_stat_statements_mean_exec_time[5m]) > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow queries detected"
          description: "Average query time is {{ $value }}ms"
      
      # Replication lag
      - alert: PostgreSQLReplicationLag
        expr: |
          (pg_replication_lag_bytes / 1024 / 1024) > 100
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Replication lag exceeds 100MB"
          description: "Replica {{ $labels.instance }} is {{ $value }}MB behind"
      
      # Low cache hit ratio
      - alert: PostgreSQLLowCacheHitRatio
        expr: |
          pg_stat_database_blks_hit / 
          (pg_stat_database_blks_hit + pg_stat_database_blks_read) < 0.95
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Cache hit ratio below 95%"
          description: "Cache hit ratio is {{ $value }}"
      
      # Database down
      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "PostgreSQL instance is down"
          description: "Database {{ $labels.instance }} is unreachable"
```

---

**2. Configure Alertmanager:**

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  
  routes:
    # Critical alerts -> PagerDuty
    - match:
        severity: critical
      receiver: pagerduty
      continue: true
    
    # Warnings -> Slack
    - match:
        severity: warning
      receiver: slack

receivers:
  - name: 'default'
    email_configs:
      - to: 'team@example.com'
        from: 'alerts@example.com'
        smarthost: smtp.gmail.com:587
        auth_username: 'alerts@example.com'
        auth_password: 'password'
  
  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#database-alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
  
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}'

inhibit_rules:
  # Don't alert on replication lag if primary is down
  - source_match:
      alertname: 'PostgreSQLDown'
    target_match:
      alertname: 'PostgreSQLReplicationLag'
    equal: ['cluster']
```

---

### MongoDB Alerts

```yaml
# MongoDB-specific alerts
groups:
  - name: mongodb
    interval: 30s
    rules:
      # High memory usage
      - alert: MongoDBHighMemory
        expr: mongodb_memory{type="resident"} / 1024 / 1024 > 4000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "MongoDB memory usage above 4GB"
          description: "Instance {{ $labels.instance }} using {{ $value }}MB"
      
      # Replication lag
      - alert: MongoDBReplicationLag
        expr: mongodb_replset_member_replication_lag > 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "MongoDB replication lag exceeds 10 seconds"
          description: "Member {{ $labels.member }} is {{ $value }}s behind"
      
      # Connection saturation
      - alert: MongoDBHighConnections
        expr: |
          mongodb_connections{state="current"} / 
          mongodb_connections{state="available"} > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "MongoDB connection usage above 80%"
      
      # Too many cursors
      - alert: MongoDBTooManyCursors
        expr: mongodb_metrics_cursor_open{state="total"} > 10000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Too many open cursors"
          description: "{{ $value }} open cursors on {{ $labels.instance }}"
```

---

### Application-Level Alerts

```javascript
const client = require('prom-client');

// Define metrics
const queryErrors = new client.Counter({
  name: 'db_query_errors_total',
  help: 'Total database query errors',
  labelNames: ['type']
});

const slowQueries = new client.Counter({
  name: 'db_slow_queries_total',
  help: 'Total slow queries (>1s)',
  labelNames: ['table']
});

// Instrument queries
async function query(sql, params) {
  const start = Date.now();
  
  try {
    const result = await pool.query(sql, params);
    const duration = Date.now() - start;
    
    // Track slow queries
    if (duration > 1000) {
      slowQueries.inc({ table: extractTable(sql) });
    }
    
    return result;
  } catch (error) {
    queryErrors.inc({ type: error.code || 'unknown' });
    throw error;
  }
}

// Alert rules (Prometheus)
// - alert: HighQueryErrorRate
//   expr: rate(db_query_errors_total[5m]) > 0.05
//   annotations:
//     summary: "Query error rate above 5%"
```

---

### On-Call Runbooks

**Example Runbook: High Connection Usage**

```markdown
# Alert: PostgreSQL High Connection Usage

## Severity: Warning

## Description
Connection usage has exceeded 80% of max_connections.

## Impact
- New connections may be rejected
- Application errors likely
- Potential downtime if connections exhausted

## Investigation Steps

1. Check current connections:
   ```sql
   SELECT count(*), state 
   FROM pg_stat_activity 
   GROUP BY state;
   ```

2. Identify connection sources:
   ```sql
   SELECT 
     client_addr, 
     count(*) 
   FROM pg_stat_activity 
   GROUP BY client_addr 
   ORDER BY count DESC;
   ```

3. Find idle connections:
   ```sql
   SELECT 
     pid, 
     state, 
     now() - state_change AS idle_time,
     query
   FROM pg_stat_activity
   WHERE state = 'idle'
   ORDER BY idle_time DESC;
   ```

## Remediation

### Immediate (< 5 min)
1. Increase max_connections temporarily:
   ```sql
   ALTER SYSTEM SET max_connections = 200;
   SELECT pg_reload_conf();
   ```

2. Terminate idle connections:
   ```sql
   SELECT pg_terminate_backend(pid)
   FROM pg_stat_activity
   WHERE state = 'idle'
     AND now() - state_change > interval '1 hour';
   ```

### Short-term (< 1 day)
1. Enable connection pooling (PgBouncer)
2. Tune application connection pool size
3. Fix connection leaks in application

### Long-term (< 1 week)
1. Investigate why connections are high
2. Optimize application architecture
3. Consider read replicas for scaling
4. Implement connection limits per application

## Escalation
If connections reach 95%, page on-call DBA immediately.

## Related Alerts
- PostgreSQL Down
- High Query Error Rate
```

---

### Alert Fatigue Prevention

**1. Tune Alert Thresholds:**
```yaml
# Too sensitive (alert fatigue)
- alert: HighCPU
  expr: cpu_usage > 50%  # ❌ Fires constantly

# Better
- alert: HighCPU
  expr: cpu_usage > 80%  # ✅ Only for real issues
  for: 5m               # ✅ Sustained, not transient
```

**2. Use Alert Grouping:**
```yaml
route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s       # Wait to group similar alerts
  group_interval: 10s   # Send grouped alerts together
  repeat_interval: 12h  # Don't re-alert too often
```

**3. Implement Alert Silencing:**
```bash
# Silence alerts during maintenance
amtool silence add \
  --alertname="PostgreSQLDown" \
  --duration="2h" \
  --comment="Scheduled maintenance"
```

**4. Progressive Alerts:**
```yaml
# Warning first
- alert: HighConnections_Warning
  expr: connection_usage > 0.7
  labels:
    severity: warning

# Critical later
- alert: HighConnections_Critical
  expr: connection_usage > 0.9
  labels:
    severity: critical
```

---

### Practice Questions: Alerting

#### Multiple Choice

**Q1: What makes a good alert?**

A) Alerts for every metric change
B) Daily summary emails
C) Actionable, urgent, specific, and rare
D) Alerts that fire every few minutes

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Actionable, urgent, specific, and rare**

Explanation: Good alerts require immediate action, are specific about the problem, and don't fire so frequently that they're ignored. Daily summaries aren't urgent, and constant alerts lead to alert fatigue.

</details>

---

**Q2: What is the purpose of the "for" clause in Prometheus alerts?**

A) To specify how long the alert should fire
B) To wait before firing an alert to avoid transient issues
C) To repeat the alert every N minutes
D) To send the alert to specific people

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - To wait before firing an alert to avoid transient issues**

Explanation: The "for" clause ensures the condition is sustained for a specified duration before firing, preventing alerts for brief, transient issues.

</details>

---

## 6.3 Backup & Recovery

### Why Backups?

**Disaster Scenarios:**
```
1. Accidental data deletion
   → DELETE FROM orders; -- Missing WHERE clause

2. Ransomware attack
   → Database encrypted, ransom demanded

3. Hardware failure
   → Disk crash, data corruption

4. Application bug
   → Bad migration, corrupted data

5. Natural disaster
   → Data center flood, fire
```

**Without Backups:** Data loss, business closure
**With Backups:** Recovery in hours

---

## 6.3.1 Backup Strategies

### Backup Types

#### 1. Full Backup

**Definition:** Complete copy of entire database.

**PostgreSQL:**
```bash
# pg_dump (logical backup)
pg_dump -h localhost -U postgres -F c -b -v -f /backups/mydb_full.dump mydb

# Flags:
# -F c: Custom format (compressed, parallel restore)
# -b: Include large objects
# -v: Verbose
# -f: Output file

# pg_basebackup (physical backup)
pg_basebackup -h localhost -U postgres -D /backups/base -Ft -z -P

# Flags:
# -D: Target directory
# -Ft: Tar format
# -z: Gzip compression
# -P: Progress
```

**MongoDB:**
```bash
# mongodump
mongodump --host=localhost --port=27017 --out=/backups/mongodb_full

# With authentication
mongodump \
  --host=localhost \
  --port=27017 \
  --username=backup_user \
  --password=password \
  --authenticationDatabase=admin \
  --out=/backups/mongodb_full
```

**Pros:** ✅ Complete data copy, easy to restore
**Cons:** ❌ Time-consuming, large storage, database lock during backup

---

#### 2. Incremental Backup

**Definition:** Backs up only changed data since last backup.

**PostgreSQL WAL Archiving:**

```bash
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backups/wal_archive/%f'
max_wal_senders = 3

# Base backup
pg_basebackup -D /backups/base_backup -Ft -z -P

# WAL files continuously archived to /backups/wal_archive/
```

**MongoDB Oplog:**

```bash
# Continuous backup using oplog
mongodump \
  --host=localhost \
  --port=27017 \
  --oplog \
  --out=/backups/incremental

# Restore to specific point in time
mongorestore \
  --host=localhost \
  --port=27017 \
  --oplogReplay \
  --oplogLimit="1640995200:1" \
  /backups/incremental
```

**Pros:** ✅ Faster, less storage
**Cons:** ❌ Complex restore, requires full + all incrementals

---

#### 3. Differential Backup

**Definition:** Backs up changes since last full backup.

```
Sunday: Full backup (10 GB)
Monday: Differential (1 GB - changes since Sunday)
Tuesday: Differential (2 GB - changes since Sunday)
Wednesday: Differential (3 GB - changes since Sunday)
```

**Restore:** Full + Latest Differential (faster than incremental)

---

### Backup Frequency

**3-2-1 Rule:**
```
3 copies of data:
  - Production database
  - Local backup
  - Remote backup

2 different media:
  - Disk
  - Cloud (S3, GCS)

1 off-site backup:
  - Different region/data center
```

**Recommended Schedule:**

| Frequency | Type | Retention |
|-----------|------|-----------|
| **Continuous** | WAL/Oplog | 7 days |
| **Hourly** | Incremental | 24 hours |
| **Daily** | Full | 30 days |
| **Weekly** | Full | 12 weeks |
| **Monthly** | Full | 12 months |
| **Yearly** | Full | 7 years |

---

### Automated Backup Script

**PostgreSQL Backup Script:**

```bash
#!/bin/bash
# backup_postgres.sh

set -e

# Configuration
DB_HOST="localhost"
DB_USER="postgres"
DB_NAME="mydb"
BACKUP_DIR="/backups/postgresql"
S3_BUCKET="s3://my-backups/postgresql"
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_DIR

# Generate filename with timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.dump"

# Create backup
echo "Starting backup at $(date)"
pg_dump -h $DB_HOST -U $DB_USER -F c -b -v -f $BACKUP_FILE $DB_NAME

# Compress
gzip $BACKUP_FILE
BACKUP_FILE="${BACKUP_FILE}.gz"

# Verify backup
if [ -f "$BACKUP_FILE" ]; then
  SIZE=$(du -h $BACKUP_FILE | cut -f1)
  echo "Backup created: $BACKUP_FILE ($SIZE)"
else
  echo "ERROR: Backup failed!"
  exit 1
fi

# Upload to S3
echo "Uploading to S3..."
aws s3 cp $BACKUP_FILE $S3_BUCKET/

# Verify S3 upload
if aws s3 ls $S3_BUCKET/$(basename $BACKUP_FILE) > /dev/null 2>&1; then
  echo "Upload successful"
else
  echo "ERROR: Upload failed!"
  exit 1
fi

# Clean up old backups (local)
echo "Cleaning up old backups..."
find $BACKUP_DIR -name "*.dump.gz" -mtime +$RETENTION_DAYS -delete

# Clean up old backups (S3)
aws s3 ls $S3_BUCKET/ | while read -r line; do
  FILE_DATE=$(echo $line | awk '{print $1" "$2}')
  FILE_NAME=$(echo $line | awk '{print $4}')
  FILE_EPOCH=$(date -d "$FILE_DATE" +%s)
  CURRENT_EPOCH=$(date +%s)
  DAYS_OLD=$(( ($CURRENT_EPOCH - $FILE_EPOCH) / 86400 ))
  
  if [ $DAYS_OLD -gt $RETENTION_DAYS ]; then
    echo "Deleting old backup: $FILE_NAME"
    aws s3 rm $S3_BUCKET/$FILE_NAME
  fi
done

echo "Backup completed at $(date)"
```

**Cron Schedule:**
```bash
# crontab -e
# Daily backup at 2 AM
0 2 * * * /scripts/backup_postgres.sh >> /var/log/backup.log 2>&1

# Weekly full backup at Sunday 3 AM
0 3 * * 0 /scripts/backup_postgres_full.sh >> /var/log/backup.log 2>&1
```

---

**MongoDB Backup Script:**

```bash
#!/bin/bash
# backup_mongodb.sh

set -e

MONGO_HOST="localhost"
MONGO_PORT="27017"
MONGO_USER="backup_user"
MONGO_PASS="password"
BACKUP_DIR="/backups/mongodb"
S3_BUCKET="s3://my-backups/mongodb"

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/$TIMESTAMP"

# Create backup
echo "Starting MongoDB backup..."
mongodump \
  --host=$MONGO_HOST \
  --port=$MONGO_PORT \
  --username=$MONGO_USER \
  --password=$MONGO_PASS \
  --authenticationDatabase=admin \
  --out=$BACKUP_PATH

# Compress
tar czf ${BACKUP_PATH}.tar.gz -C $BACKUP_DIR $TIMESTAMP
rm -rf $BACKUP_PATH

# Upload to S3
aws s3 cp ${BACKUP_PATH}.tar.gz $S3_BUCKET/

# Clean up old backups
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed"
```

---

### Testing Backups

**Critical:** Test backups regularly!

```bash
#!/bin/bash
# test_restore.sh

# Restore to test database
echo "Testing backup restore..."

# PostgreSQL
pg_restore -h localhost -U postgres -d test_db -v /backups/latest.dump

# Verify data
psql -h localhost -U postgres -d test_db -c "SELECT count(*) FROM users;"

# MongoDB
mongorestore --host=localhost --port=27018 --drop /backups/latest

# Verify
mongo localhost:27018/mydb --eval "db.users.count()"

echo "Restore test completed successfully"
```

**Schedule:** Test weekly or monthly

---

### Practice Questions: Backup Strategies

#### Fill in the Blanks

1. A ________ backup creates a complete copy of the entire database.
2. ________ backups only include changes since the last full backup.
3. The ________ rule states you should have 3 copies on 2 media with 1 off-site.
4. PostgreSQL ________ archiving enables point-in-time recovery.
5. Backups should be ________ regularly to ensure they can be restored successfully.

<details>
<summary><strong>View Answers</strong></summary>

1. full
2. Differential (or Incremental)
3. 3-2-1
4. WAL
5. tested

</details>

---

## 6.3.2 Recovery Scenarios

### Scenario 1: Accidental Data Deletion

**Problem:**
```sql
-- Developer runs in production (meant for dev)
DELETE FROM orders;
-- 1,000,000 rows deleted! 💀
```

---

#### Recovery: Point-in-Time Recovery (PITR)

**PostgreSQL:**

```bash
# 1. Stop PostgreSQL
sudo systemctl stop postgresql

# 2. Move corrupted data directory
mv /var/lib/postgresql/data /var/lib/postgresql/data_corrupted

# 3. Restore base backup
tar xzf /backups/base_backup.tar.gz -C /var/lib/postgresql/data

# 4. Create recovery.conf
cat > /var/lib/postgresql/data/recovery.conf << EOF
restore_command = 'cp /backups/wal_archive/%f %p'
recovery_target_time = '2024-01-15 14:30:00'  # Before deletion
recovery_target_action = 'promote'
EOF

# 5. Start PostgreSQL (recovery mode)
sudo systemctl start postgresql

# 6. Monitor recovery
tail -f /var/log/postgresql/postgresql.log

# 7. Verify data
psql -U postgres -d mydb -c "SELECT count(*) FROM orders;"

# 8. Promote to production
psql -U postgres -c "SELECT pg_wal_replay_resume();"
```

---

**MongoDB:**

```bash
# 1. Stop MongoDB
sudo systemctl stop mongod

# 2. Restore backup
mongorestore \
  --host=localhost \
  --port=27017 \
  --drop \
  /backups/mongodb_full

# 3. Restore oplog to specific time
mongorestore \
  --host=localhost \
  --port=27017 \
  --oplogReplay \
  --oplogLimit="1640995200:1" \  # Timestamp before deletion
  /backups/oplog

# 4. Verify
mongo mydb --eval "db.orders.count()"

# 5. Start MongoDB
sudo systemctl start mongod
```

---

### Scenario 2: Corrupted Database

**Problem:**
```
Hardware failure → Disk corruption → Database won't start
```

**PostgreSQL Recovery:**

```bash
# 1. Try automatic recovery
sudo systemctl start postgresql

# If fails:
# 2. Check logs
tail -f /var/log/postgresql/postgresql.log
# ERROR: could not read block 123 in file "base/16384/12345": read only 0 of 8192 bytes

# 3. Enable zero-damaged pages (last resort)
cat >> /var/lib/postgresql/data/postgresql.conf << EOF
zero_damaged_pages = on
EOF

sudo systemctl start postgresql

# 4. Dump recoverable data
pg_dump -h localhost -U postgres -Fc mydb > /backups/recovered.dump

# 5. Recreate database from backup
dropdb mydb
createdb mydb
pg_restore -d mydb /backups/latest_good_backup.dump

# 6. Compare with recovered dump
# Manually restore any missing data
```

---

### Scenario 3: Ransomware Attack

**Problem:**
```
Database encrypted by ransomware
Ransom demanded: 10 BTC
```

**Recovery:**

```bash
# 1. Isolate affected systems
# Disconnect from network immediately

# 2. Do NOT pay ransom

# 3. Restore from off-site backup
# Use backup from before infection (check timestamps)

# 4. Verify backup integrity
pg_restore --list /backups/s3/mydb_20240115.dump | head -20

# 5. Restore to new server
pg_restore -h new-server -U postgres -d mydb /backups/s3/mydb_20240115.dump

# 6. Verify data integrity
psql -h new-server -U postgres -d mydb << EOF
SELECT count(*) FROM users;
SELECT count(*) FROM orders;
SELECT max(created_at) FROM orders;  -- Check latest data
EOF

# 7. Update application to point to new server

# 8. Investigate infection source
# Scan all systems, update security
```

---

### Scenario 4: Replica Promotion (Primary Failure)

**PostgreSQL:**

```bash
# Primary server fails

# 1. Promote replica to primary
pg_ctl promote -D /var/lib/postgresql/data

# Or
sudo -u postgres psql -c "SELECT pg_promote();"

# 2. Update application connection string
# Point writes to new primary

# 3. Verify promotion
psql -U postgres -c "SELECT pg_is_in_recovery();"
# Should return: f (false - not in recovery)

# 4. Set up new replica from new primary
pg_basebackup -h new-primary -D /var/lib/postgresql/replica -P -R

# 5. Start new replica
sudo systemctl start postgresql@replica
```

---

**MongoDB:**

```javascript
// Primary fails - automatic failover

// 1. Check replica set status
rs.status();

// New primary elected automatically (< 10 seconds)

// 2. Application automatically reconnects
// (if using replica set connection string)

// 3. Investigate old primary
// When it comes back, it becomes secondary

// 4. If manual intervention needed
rs.stepDown(60);  // Force primary to step down

// 5. Force specific member to primary
cfg = rs.conf();
cfg.members[1].priority = 2;  // Higher priority
rs.reconfig(cfg);
```

---

### Scenario 5: Partial Data Loss

**Problem:**
```sql
-- Migration gone wrong
UPDATE users SET email = NULL;  -- Missing WHERE clause
-- All emails set to NULL!
```

**Recovery Options:**

**Option 1: From Backup + Manual Fix**
```bash
# 1. Restore to temp database
pg_restore -h localhost -U postgres -d temp_recovery /backups/latest.dump

# 2. Extract lost data
psql -h localhost -U postgres -d temp_recovery -c "
  COPY (SELECT id, email FROM users) 
  TO '/tmp/emails.csv' CSV HEADER;
"

# 3. Update production
psql -h localhost -U postgres -d mydb << EOF
CREATE TEMP TABLE temp_emails (id INTEGER, email VARCHAR);
\COPY temp_emails FROM '/tmp/emails.csv' CSV HEADER;

UPDATE users u
SET email = t.email
FROM temp_emails t
WHERE u.id = t.id;
EOF

# 4. Verify
psql -h localhost -U postgres -d mydb -c "
  SELECT count(*) FILTER (WHERE email IS NULL) AS null_emails,
         count(*) AS total_users
  FROM users;
"
```

**Option 2: From Audit Log (if enabled)**
```sql
-- If you have audit logging
SELECT 
  user_id,
  old_email,
  timestamp
FROM audit_log
WHERE table_name = 'users'
  AND column_name = 'email'
  AND action = 'UPDATE'
  AND timestamp > '2024-01-15 14:00:00'
ORDER BY timestamp;

-- Restore from audit
UPDATE users u
SET email = a.old_email
FROM audit_log a
WHERE u.id = a.user_id
  AND a.table_name = 'users'
  AND a.column_name = 'email';
```

---

### Practice Questions: Recovery Scenarios

#### Multiple Choice

**Q1: What is Point-in-Time Recovery (PITR)?**

A) Restoring to the exact moment of failure
B) Restoring to a specific time before corruption
C) Restoring the most recent backup
D) Restoring only specific tables

<details>
<summary><strong>View Answer</strong></summary>

**Answer: B - Restoring to a specific time before corruption**

Explanation: PITR uses base backups plus transaction logs (WAL/oplog) to restore the database to any point in time, typically just before a problem occurred.

</details>

---

**Q2: What should you do if ransomware encrypts your database?**

A) Pay the ransom immediately
B) Try to decrypt the data
C) Restore from an off-site backup before infection
D) Recreate all data manually

<details>
<summary><strong>View Answer</strong></summary>

**Answer: C - Restore from an off-site backup before infection**

Explanation: Never pay ransoms. Restore from a clean off-site backup taken before the infection. This is why off-site backups and the 3-2-1 rule are critical.

</details>

---

## 6.3.3 Backup Best Practices

### 1. Automate Everything

```bash
# Bad: Manual backups
# "I'll remember to run the backup script"
# Result: Backups forgotten, data lost

# Good: Automated cron jobs
0 2 * * * /scripts/backup_postgres.sh
0 3 * * 0 /scripts/backup_postgres_full.sh
0 4 * * * /scripts/backup_mongodb.sh
```

---

### 2. Verify Backups

```bash
# Verification script
#!/bin/bash

BACKUP_FILE="/backups/latest.dump"

# 1. Check file exists
if [ ! -f "$BACKUP_FILE" ]; then
  echo "ERROR: Backup file not found!"
  exit 1
fi

# 2. Check file size (should be > 100MB)
SIZE=$(stat -f%z "$BACKUP_FILE" 2>/dev/null || stat -c%s "$BACKUP_FILE")
if [ $SIZE -lt 104857600 ]; then
  echo "ERROR: Backup file too small ($SIZE bytes)"
  exit 1
fi

# 3. Test restore to temp DB
pg_restore -h localhost -U postgres -d test_restore -c $BACKUP_FILE

# 4. Verify record counts
PROD_COUNT=$(psql -h localhost -U postgres -d mydb -t -c "SELECT count(*) FROM users")
TEST_COUNT=$(psql -h localhost -U postgres -d test_restore -t -c "SELECT count(*) FROM users")

if [ "$PROD_COUNT" != "$TEST_COUNT" ]; then
  echo "ERROR: Record count mismatch!"
  exit 1
fi

# 5. Clean up
dropdb test_restore

echo "Backup verification successful"
```

---

### 3. Encrypt Backups

```bash
# Encrypt backup before upload
gpg --symmetric --cipher-algo AES256 /backups/mydb.dump

# Upload encrypted backup
aws s3 cp /backups/mydb.dump.gpg s3://backups/

# Decrypt for restore
gpg --decrypt /backups/mydb.dump.gpg > /backups/mydb.dump
```

---

### 4. Monitor Backup Success

```javascript
// Send backup status to monitoring
const axios = require('axios');

async function notifyBackupStatus(status, details) {
  await axios.post('https://monitoring.example.com/webhooks/backup', {
    database: 'production',
    status: status,  // 'success' or 'failed'
    timestamp: new Date().toISOString(),
    size: details.size,
    duration: details.duration,
    location: details.location
  });
}

// In backup script
try {
  const startTime = Date.now();
  await performBackup();
  const duration = Date.now() - startTime;
  
  await notifyBackupStatus('success', {
    size: '5.2 GB',
    duration: `${duration}ms`,
    location: 's3://backups/mydb_20240115.dump'
  });
} catch (error) {
  await notifyBackupStatus('failed', {
    error: error.message
  });
  throw error;
}
```

---

### 5. Document Recovery Procedures

```markdown
# Database Recovery Playbook

## Contact Information
- On-Call DBA: +1-555-0100
- Team Lead: +1-555-0101
- AWS Support: 1-866-720-5050

## Recovery Time Objectives (RTO)
- Critical: 1 hour
- High: 4 hours
- Normal: 24 hours

## Recovery Point Objectives (RPO)
- Critical: 5 minutes (oplog/WAL)
- High: 1 hour (incremental)
- Normal: 24 hours (daily backup)

## Backup Locations
- Local: /backups/postgresql
- S3: s3://prod-backups/postgresql
- Glacier: s3://prod-backups-archive/postgresql

## Recovery Procedures
See scenarios above...

## Post-Recovery Checklist
- [ ] Verify data integrity
- [ ] Update monitoring
- [ ] Notify stakeholders
- [ ] Document incident
- [ ] Update recovery procedures
```

---

### Practice Questions: Backup Best Practices

#### Fill in the Blanks

1. Backups should be ________ to prevent human error and ensure consistency.
2. ________ backups before storing them to protect sensitive data.
3. Recovery Time Objective (______) defines how quickly you must restore.
4. Recovery Point Objective (______) defines how much data loss is acceptable.
5. Always ________ backup integrity with test restores.

<details>
<summary><strong>View Answers</strong></summary>

1. automated
2. Encrypt
3. RTO
4. RPO
5. verify (or test)

</details>

---

## 6.4 Database Testing

### Why Test Databases?

**Without Testing:**
```sql
-- Deploy migration to production
ALTER TABLE users DROP COLUMN email;

-- Application crashes - email column required!
-- Rollback is difficult
-- Data potentially lost
```

**With Testing:**
```
Test → Staging → Production
Each step catches issues before production
```

---

## 6.4.1 Unit Testing

### Testing Database Queries

**Jest + PostgreSQL:**

```javascript
const { Pool } = require('pg');
const pool = new Pool({
  host: 'localhost',
  database: 'test_db',
  user: 'test_user',
  password: 'test_password'
});

describe('User Repository', () => {
  beforeEach(async () => {
    // Clean database before each test
    await pool.query('TRUNCATE users RESTART IDENTITY CASCADE');
  });
  
  afterAll(async () => {
    await pool.end();
  });
  
  test('should create user', async () => {
    const result = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Alice', 'alice@example.com']
    );
    
    expect(result.rows[0].name).toBe('Alice');
    expect(result.rows[0].email).toBe('alice@example.com');
  });
  
  test('should enforce unique email', async () => {
    await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2)',
      ['Alice', 'alice@example.com']
    );
    
    await expect(
      pool.query(
        'INSERT INTO users (name, email) VALUES ($1, $2)',
        ['Bob', 'alice@example.com']
      )
    ).rejects.toThrow();
  });
  
  test('should cascade delete orders when user deleted', async () => {
    // Create user
    const userResult = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id',
      ['Alice', 'alice@example.com']
    );
    const userId = userResult.rows[0].id;
    
    // Create order
    await pool.query(
      'INSERT INTO orders (user_id, total) VALUES ($1, $2)',
      [userId, 99.99]
    );
    
    // Delete user
    await pool.query('DELETE FROM users WHERE id = $1', [userId]);
    
    // Verify orders deleted
    const ordersResult = await pool.query(
      'SELECT * FROM orders WHERE user_id = $1',
      [userId]
    );
    expect(ordersResult.rows.length).toBe(0);
  });
});
```

---

**Mocha + MongoDB:**

```javascript
const { MongoClient } = require('mongodb');
const { expect } = require('chai');

describe('User Repository', () => {
  let client;
  let db;
  
  before(async () => {
    client = new MongoClient('mongodb://localhost:27017');
    await client.connect();
    db = client.db('test_db');
  });
  
  beforeEach(async () => {
    await db.collection('users').deleteMany({});
  });
  
  after(async () => {
    await client.close();
  });
  
  it('should create user', async () => {
    const result = await db.collection('users').insertOne({
      name: 'Alice',
      email: 'alice@example.com'
    });
    
    expect(result.insertedId).to.exist;
    
    const user = await db.collection('users').findOne({
      _id: result.insertedId
    });
    
    expect(user.name).to.equal('Alice');
    expect(user.email).to.equal('alice@example.com');
  });
  
  it('should enforce unique email index', async () => {
    await db.collection('users').createIndex(
      { email: 1 },
      { unique: true }
    );
    
    await db.collection('users').insertOne({
      name: 'Alice',
      email: 'alice@example.com'
    });
    
    try {
      await db.collection('users').insertOne({
        name: 'Bob',
        email: 'alice@example.com'
      });
      expect.fail('Should have thrown error');
    } catch (error) {
      expect(error.code).to.equal(11000);  // Duplicate key error
    }
  });
});
```

---

### Testing with Docker

**docker-compose.test.yml:**

```yaml
version: '3.8'

services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
    ports:
      - "5433:5432"
    tmpfs:
      - /var/lib/postgresql/data  # In-memory DB for speed
  
  test-runner:
    build: .
    command: npm test
    environment:
      DB_HOST: test-db
      DB_PORT: 5432
      DB_NAME: test_db
      DB_USER: test_user
      DB_PASSWORD: test_password
    depends_on:
      - test-db
```

**Run tests:**
```bash
docker-compose -f docker-compose.test.yml up --abort-on-container-exit
```

---

## 6.4.2 Integration Testing

### Testing Full Application Flow

```javascript
const request = require('supertest');
const app = require('../app');
const { Pool } = require('pg');

const pool = new Pool({ /* test db config */ });

describe('User API Integration Tests', () => {
  beforeEach(async () => {
    await pool.query('TRUNCATE users RESTART IDENTITY CASCADE');
  });
  
  test('POST /users should create user and return 201', async () => {
    const response = await request(app)
      .post('/users')
      .send({
        name: 'Alice',
        email: 'alice@example.com'
      })
      .expect(201);
    
    expect(response.body.id).toBeDefined();
    expect(response.body.name).toBe('Alice');
    
    // Verify in database
    const result = await pool.query(
      'SELECT * FROM users WHERE id = $1',
      [response.body.id]
    );
    expect(result.rows[0].email).toBe('alice@example.com');
  });
  
  test('GET /users/:id should return user', async () => {
    // Create user
    const createResult = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Alice', 'alice@example.com']
    );
    const userId = createResult.rows[0].id;
    
    // Get user via API
    const response = await request(app)
      .get(`/users/${userId}`)
      .expect(200);
    
    expect(response.body.name).toBe('Alice');
    expect(response.body.email).toBe('alice@example.com');
  });
  
  test('DELETE /users/:id should delete user', async () => {
    const createResult = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Alice', 'alice@example.com']
    );
    const userId = createResult.rows[0].id;
    
    await request(app)
      .delete(`/users/${userId}`)
      .expect(204);
    
    // Verify deletion
    const result = await pool.query(
      'SELECT * FROM users WHERE id = $1',
      [userId]
    );
    expect(result.rows.length).toBe(0);
  });
});
```

---

## 6.4.3 Load Testing

### Why Load Test?

**Scenarios:**
```
Normal: 100 requests/sec → Works fine
Black Friday: 10,000 requests/sec → Database crashes

Without load testing: Find out during Black Friday
With load testing: Find out beforehand, optimize
```

---

### k6 Load Testing

**Install:**
```bash
brew install k6  # macOS
# or
wget https://github.com/grafana/k6/releases/download/v0.45.0/k6-v0.45.0-linux-amd64.tar.gz
```

**Load Test Script:**

```javascript
// load_test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 50 },   // Ramp up to 50 users
    { duration: '3m', target: 50 },   // Stay at 50 users
    { duration: '1m', target: 100 },  // Ramp up to 100 users
    { duration: '3m', target: 100 },  // Stay at 100 users
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests < 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  // Create user
  const createPayload = JSON.stringify({
    name: 'Test User',
    email: `test${__VU}_${__ITER}@example.com`
  });
  
  const createRes = http.post('http://localhost:3000/users', createPayload, {
    headers: { 'Content-Type': 'application/json' },
  });
  
  check(createRes, {
    'user created': (r) => r.status === 201,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  const userId = JSON.parse(createRes.body).id;
  
  // Get user
  const getRes = http.get(`http://localhost:3000/users/${userId}`);
  
  check(getRes, {
    'user retrieved': (r) => r.status === 200,
    'correct user': (r) => JSON.parse(r.body).id === userId,
  });
  
  sleep(1);
}
```

**Run:**
```bash
k6 run load_test.js
```

**Output:**
```
     ✓ user created
     ✓ response time < 500ms
     ✓ user retrieved
     ✓ correct user

     checks.........................: 100.00% ✓ 20000      ✗ 0
     data_received..................: 15 MB   50 kB/s
     data_sent......................: 12 MB   40 kB/s
     http_req_duration..............: avg=145ms  min=50ms  med=120ms  max=850ms  p(95)=350ms
     http_reqs......................: 10000   33.33/s
     vus............................: 100     min=0       max=100
```

---

### Database-Specific Load Testing

**PostgreSQL:**

```bash
# pgbench - built-in load testing tool
# Initialize test database
pgbench -i -s 50 testdb

# Run test: 10 clients, 100 transactions each
pgbench -c 10 -t 100 testdb

# Output:
# transaction type: <builtin: TPC-B (sort of)>
# scaling factor: 50
# query mode: simple
# number of clients: 10
# number of threads: 1
# number of transactions per client: 100
# number of transactions actually processed: 1000/1000
# latency average = 15.844 ms
# tps = 631.191804 (including connections establishing)
```

**Custom pgbench script:**

```sql
-- test_queries.sql
\set user_id random(1, 1000000)

BEGIN;
SELECT * FROM users WHERE id = :user_id;
UPDATE users SET last_login = now() WHERE id = :user_id;
INSERT INTO user_activity (user_id, action, created_at) 
  VALUES (:user_id, 'login', now());
COMMIT;
```

```bash
pgbench -c 50 -t 1000 -f test_queries.sql testdb
```

---

**MongoDB:**

```bash
# Install mongo-perf
git clone https://github.com/mongodb-labs/mongo-perf.git
cd mongo-perf

# Run load test
node benchmark.js \
  --host localhost:27017 \
  --threads 10 \
  --operations 10000 \
  --test insert
```

---

### Analyzing Load Test Results

**Key Metrics:**

1. **Response Time Percentiles:**
```
p50 (median): 120ms  ✅ Good
p95: 350ms           ✅ Acceptable
p99: 850ms           ⚠️  Investigate
```

2. **Throughput:**
```
Current: 631 TPS (transactions/sec)
Target: 1000 TPS
Action: Optimize queries, add indexes
```

3. **Error Rate:**
```
Errors: 0.5%  ✅ Good
Target: < 1%
```

4. **Resource Usage:**
```
CPU: 75%      ✅ Good headroom
Memory: 60%   ✅ Good
Connections: 45/100  ✅ Good
Disk I/O: 80% ⚠️  Consider faster disk
```

---

### Practice Questions: Database Testing

#### Fill in the Blanks

1. ________ testing verifies individual database queries work correctly.
2. ________ testing verifies the entire application flow including database.
3. ________ testing simulates high traffic to find performance bottlenecks.
4. The ________ tool is PostgreSQL's built-in load testing utility.
5. Load tests should verify response time ________ like p95 and p99.

<details>
<summary><strong>View Answers</strong></summary>

1. Unit
2. Integration
3. Load
4. pgbench
5. percentiles

</details>

---

## 6.5 Database Migrations

### What are Migrations?

**Definition:** Versioned changes to database schema.

**Without Migrations:**
```
Developer A: ALTER TABLE users ADD COLUMN phone VARCHAR(20);
Developer B: Doesn't know about change
Production: Schema mismatch, errors
```

**With Migrations:**
```
Migration file: 001_add_phone_to_users.sql
Tracked in version control
Applied in order: dev → staging → production
Everyone has same schema
```

---

### Migration Tools

#### Node.js: node-pg-migrate

**Install:**
```bash
npm install node-pg-migrate
```

**Create Migration:**
```bash
npx node-pg-migrate create add-phone-to-users
```

**Migration File (migrations/1640000000000_add-phone-to-users.js):**

```javascript
exports.up = (pgm) => {
  pgm.addColumn('users', {
    phone: {
      type: 'varchar(20)',
      notNull: false
    }
  });
  
  pgm.createIndex('users', 'phone');
};

exports.down = (pgm) => {
  pgm.dropIndex('users', 'phone');
  pgm.dropColumn('users', 'phone');
};
```

**Run Migrations:**
```bash
# Apply all pending migrations
npx node-pg-migrate up

# Rollback last migration
npx node-pg-migrate down

# Rollback to specific migration
npx node-pg-migrate down --to 1640000000000
```

---

#### Sequelize Migrations

**Create Migration:**
```bash
npx sequelize-cli migration:generate --name add-phone-to-users
```

**Migration File:**

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('users', 'phone', {
      type: Sequelize.STRING(20),
      allowNull: true
    });
    
    await queryInterface.addIndex('users', ['phone']);
  },
  
  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeIndex('users', ['phone']);
    await queryInterface.removeColumn('users', 'phone');
  }
};
```

---

#### Flyway (Java/Multi-language)

**Migration File (V1__add_phone_to_users.sql):**

```sql
-- V1__add_phone_to_users.sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
CREATE INDEX idx_users_phone ON users(phone);
```

**Run:**
```bash
flyway migrate -url=jdbc:postgresql://localhost:5432/mydb -user=postgres
```

---

### Migration Best Practices

#### 1. Always Reversible

```javascript
// ✅ Good: Reversible
exports.up = (pgm) => {
  pgm.addColumn('users', { phone: 'varchar(20)' });
};

exports.down = (pgm) => {
  pgm.dropColumn('users', 'phone');
};

// ❌ Bad: Not reversible
exports.up = (pgm) => {
  pgm.sql('UPDATE users SET deleted = true WHERE active = false');
  pgm.dropColumn('users', 'active');
};

exports.down = (pgm) => {
  // Can't recover deleted data!
  pgm.addColumn('users', { active: 'boolean' });
};
```

---

#### 2. Backward Compatible

```javascript
// ❌ Bad: Breaking change
exports.up = (pgm) => {
  // Old code expects 'name', new code expects 'first_name'
  pgm.renameColumn('users', 'name', 'first_name');
};

// ✅ Good: Gradual migration
// Migration 1: Add new column
exports.up = (pgm) => {
  pgm.addColumn('users', { first_name: 'varchar(100)' });
  pgm.sql('UPDATE users SET first_name = name');
};

// Migration 2: (After deploying new code)
exports.up = (pgm) => {
  pgm.dropColumn('users', 'name');
};
```

---

#### 3. Data Migrations Separate from Schema

```javascript
// Schema migration: 001_add_status_column.js
exports.up = (pgm) => {
  pgm.addColumn('orders', {
    status: {
      type: 'varchar(20)',
      notNull: false,  // Initially nullable
      default: 'pending'
    }
  });
};

// Data migration: 002_populate_status.js
exports.up = (pgm) => {
  pgm.sql(`
    UPDATE orders 
    SET status = 
      CASE 
        WHEN shipped_at IS NOT NULL THEN 'shipped'
        WHEN cancelled_at IS NOT NULL THEN 'cancelled'
        ELSE 'pending'
      END
  `);
};

// Constraint migration: 003_make_status_required.js
exports.up = (pgm) => {
  pgm.alterColumn('orders', 'status', {
    notNull: true
  });
};
```

---

#### 4. Test Migrations

```javascript
// test/migrations.test.js
describe('Migrations', () => {
  it('should run up and down without errors', async () => {
    // Run all migrations
    await runMigrations('up');
    
    // Verify schema
    const result = await pool.query(`
      SELECT column_name 
      FROM information_schema.columns 
      WHERE table_name = 'users'
    `);
    
    const columns = result.rows.map(r => r.column_name);
    expect(columns).toContain('phone');
    
    // Rollback
    await runMigrations('down');
    
    // Verify rollback
    const result2 = await pool.query(`
      SELECT column_name 
      FROM information_schema.columns 
      WHERE table_name = 'users'
    `);
    
    const columns2 = result2.rows.map(r => r.column_name);
    expect(columns2).not.toContain('phone');
  });
});
```

---

#### 5. Lock-Free Migrations

```sql
-- ❌ Bad: Locks table during migration
ALTER TABLE users ADD COLUMN phone VARCHAR(20) NOT NULL DEFAULT '';

-- ✅ Good: No lock (PostgreSQL 11+)
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- Deploy new code that handles NULL
UPDATE users SET phone = '' WHERE phone IS NULL;  -- In batches
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
```

---

### MongoDB Migrations

**migrate-mongo:**

```bash
npm install migrate-mongo
npx migrate-mongo init
```

**Migration File (migrations/20240115000000-add-phone-to-users.js):**

```javascript
module.exports = {
  async up(db, client) {
    // Add phone field to all users
    await db.collection('users').updateMany(
      { phone: { $exists: false } },
      { $set: { phone: null } }
    );
    
    // Create index
    await db.collection('users').createIndex({ phone: 1 });
  },
  
  async down(db, client) {
    // Remove index
    await db.collection('users').dropIndex({ phone: 1 });
    
    // Remove phone field
    await db.collection('users').updateMany(
      {},
      { $unset: { phone: '' } }
    );
  }
};
```

**Run:**
```bash
npx migrate-mongo up
npx migrate-mongo down
```

---

### Practice Questions: Database Migrations

#### Fill in the Blanks

1. Database ________ are versioned changes to the database schema.
2. Every migration should have an ________ function to reverse changes.
3. ________ migrations allow old and new code to work simultaneously.
4. ________ migrations should be separate from schema migrations.
5. Migrations should be ________ in development, staging, and production.

<details>
<summary><strong>View Answers</strong></summary>

1. migrations
2. down (or rollback)
3. Backward compatible
4. Data
5. tested

</details>

---

## Chapter Summary

### Key Concepts Covered

#### 6.1 Database Security

**Least Privilege:**
- Minimal permissions per user/application
- Row-level security (PostgreSQL RLS)
- Custom roles (MongoDB)

**SQL Injection Prevention:**
- Parameterized queries (always)
- Input validation
- ORMs for safety

**Password Security:**
- Hash with bcrypt or Argon2 (never plain text)
- Strong password policies (12+ chars)
- Secure password reset (tokens, expiration)

**Encryption:**
- At rest: pgcrypto, WiredTiger, disk encryption
- In transit: TLS/SSL certificates
- Key rotation strategies

**Secrets Management:**
- Environment variables (.env)
- Docker/Kubernetes secrets
- AWS Secrets Manager, HashiCorp Vault
- Never commit secrets to Git

---

#### 6.2 Monitoring & Observability

**Key Metrics (4 Golden Signals):**
- Latency: Query response time
- Traffic: Queries per second
- Errors: Failed queries
- Saturation: Resource usage (CPU, memory, disk, connections)

**Monitoring Tools:**
- Prometheus + Grafana
- APM (New Relic, Datadog)
- Database-specific (pgAdmin, Ops Manager)
- ELK stack (logs)

**Alerting:**
- Actionable, urgent, specific, rare
- Prometheus alert rules
- Alertmanager (Slack, PagerDuty)
- On-call runbooks
- Alert fatigue prevention

---

#### 6.3 Backup & Recovery

**Backup Strategies:**
- Full, incremental, differential
- 3-2-1 rule (3 copies, 2 media, 1 off-site)
- Automated daily/weekly/monthly
- Encrypted backups

**Recovery Scenarios:**
- Accidental deletion: PITR
- Corruption: Restore from backup
- Ransomware: Off-site backup (never pay)
- Primary failure: Replica promotion
- Partial loss: Restore + merge

**Best Practices:**
- Automate everything
- Verify backups (test restores)
- Encrypt backups
- Monitor backup success
- Document recovery procedures

---

#### 6.4 Database Testing

**Unit Testing:**
- Test individual queries
- Jest/Mocha + test database
- Docker for isolation

**Integration Testing:**
- Test full application flow
- API + database
- Verify end-to-end

**Load Testing:**
- k6, pgbench, mongo-perf
- Simulate production traffic
- Find performance bottlenecks
- Analyze percentiles (p95, p99)

---

#### 6.5 Database Migrations

**Tools:**
- node-pg-migrate, Sequelize, Flyway
- migrate-mongo (MongoDB)

**Best Practices:**
- Always reversible (up/down)
- Backward compatible
- Separate data from schema
- Test migrations
- Lock-free when possible

---

### Production Checklist

**Security:**
- [ ] Least privilege roles configured
- [ ] SQL injection prevention (parameterized queries)
- [ ] Passwords hashed (Argon2/bcrypt)
- [ ] Encryption at rest enabled
- [ ] TLS/SSL for connections
- [ ] Secrets in vault (not code)
- [ ] Firewall configured
- [ ] Audit logging enabled

**Monitoring:**
- [ ] Prometheus + Grafana dashboards
- [ ] APM configured (New Relic/Datadog)
- [ ] Alerts configured (critical + warning)
- [ ] On-call runbooks written
- [ ] Log aggregation (ELK)
- [ ] Performance baselines established

**Backup:**
- [ ] Automated daily backups
- [ ] Off-site backups (S3/Glacier)
- [ ] Backup verification automated
- [ ] Recovery procedures documented
- [ ] PITR enabled (WAL/oplog)
- [ ] RTO/RPO defined

**Testing:**
- [ ] Unit tests for queries
- [ ] Integration tests for API
- [ ] Load tests run regularly
- [ ] Test database in CI/CD
- [ ] Migration tests automated

**Operations:**
- [ ] Connection pooling configured
- [ ] Migrations automated
- [ ] Monitoring dashboards created
- [ ] Alert escalation defined
- [ ] Documentation up-to-date

---

**Next Chapter:** [Real-World Projects →](./sections)