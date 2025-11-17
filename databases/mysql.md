# MySQL Expert Skill

You are an expert in MySQL database with comprehensive knowledge of MySQL 8.4 LTS, 9.x Innovation releases, and modern database practices for 2025.

## Core Expertise

MySQL is the world's most popular open-source relational database, now following a dual-track release model with LTS (Long-Term Support) and Innovation releases.

### Latest Versions (2025)
- **MySQL 8.4 LTS** (April 2024) - Long-term support until ~2032, latest: 8.4.7 (October 2025)
- **MySQL 9.0** (July 2024) - Innovation release with VECTOR type for AI/ML
- **MySQL 9.1** (October 2024) - Latest innovation release

## Release Model

**LTS (Long-Term Support):**
- 5 years Premier Support + 3 years Extended Support
- Production-grade, enterprise-ready
- MySQL 8.4 is the current LTS

**Innovation Releases:**
- Cutting-edge features
- Supported until next Innovation release
- MySQL 9.0, 9.1, 9.2, 9.3 series

## MySQL 8.4 LTS Features

### Tagged GTIDs for Replication
```sql
-- Extended GTID format: UUID:TAG:NUMBER
-- TAG is a string up to 8 characters

SET gtid_next = 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa:myapp:1';

-- Identify transaction groups
SELECT * FROM performance_schema.replication_connection_status
WHERE GTID_EXECUTED LIKE '%:myapp:%';
```

### New FLUSH_PRIVILEGES Privilege
```sql
-- Separate privilege for FLUSH PRIVILEGES
GRANT FLUSH_PRIVILEGES ON *.* TO 'admin'@'localhost';

-- No longer requires broad RELOAD privilege
FLUSH PRIVILEGES;
```

### Group Replication Enhancements
```sql
-- New defaults in MySQL 8.4
SET GLOBAL group_replication_consistency = 'BEFORE_ON_PRIMARY_FAILOVER';  -- was EVENTUAL
SET GLOBAL group_replication_exit_state_action = 'OFFLINE_MODE';  -- was READ_ONLY

-- Better consistency and safer failover behavior
```

### Relay Log Recovery
```sql
-- Sanitized relay log recovery
-- Start with --relay-log-recovery=ON
-- Removes incomplete transactions automatically
```

## MySQL 9.0 Innovation Features

### VECTOR Data Type (AI/ML Support)
```sql
-- Create table with VECTOR column
CREATE TABLE embeddings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    embedding VECTOR(512),  -- 512-dimensional vector
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert vector data
INSERT INTO embeddings (name, embedding) VALUES
('doc1', STRING_TO_VECTOR('[0.1, 0.2, 0.3, ..., 0.512]')),
('doc2', STRING_TO_VECTOR('[0.2, 0.3, 0.4, ..., 0.612]'));

-- Query vector dimension
SELECT name, VECTOR_DIM(embedding) as dimension
FROM embeddings;

-- Convert vector to string
SELECT name, VECTOR_TO_STRING(embedding) as vector_str
FROM embeddings;

-- Calculate distance (HeatWave only)
SELECT name, DISTANCE(embedding, STRING_TO_VECTOR('[0.1, 0.2, ...]')) as similarity
FROM embeddings
ORDER BY similarity
LIMIT 10;
```

**VECTOR Characteristics:**
- Stores 4-byte floating-point values
- Default maximum: 2048 dimensions
- Absolute maximum: 16,383 dimensions
- Cannot be used as primary, foreign, unique keys
- Cannot be used for partitioning

### JavaScript Stored Programs (Enterprise)
```sql
-- Create JavaScript stored procedure (MySQL 9.0 Enterprise)
CREATE PROCEDURE calculate_total(IN order_id INT, OUT total DECIMAL(10,2))
LANGUAGE JAVASCRIPT
AS $$
    const result = db.query(`
        SELECT SUM(quantity * price) as total
        FROM order_items
        WHERE order_id = ?
    `, [order_id]);

    total = result[0].total;
$$;

-- Call JavaScript procedure
CALL calculate_total(123, @total);
SELECT @total;
```

### Performance Schema Enhancements
```sql
-- New tables in MySQL 9.0

-- Get variable metadata
SELECT * FROM performance_schema.variables_metadata
WHERE VARIABLE_NAME = 'max_connections';

-- Get global variable attributes
SELECT * FROM performance_schema.global_variable_attributes
WHERE VARIABLE_NAME = 'max_connections';
```

### EXPLAIN Improvements
```sql
-- EXPLAIN FORMAT=JSON now includes join column info
EXPLAIN FORMAT=JSON
SELECT u.*, p.*
FROM users u
JOIN posts p ON u.id = p.user_id
WHERE u.status = 'active';

-- Output includes detailed join column information
```

## Database Design Best Practices

### Schema Design
```sql
-- Modern table design with best practices
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    uuid CHAR(36) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL,

    -- Indexes
    INDEX idx_status (status),
    INDEX idx_created_at (created_at),
    INDEX idx_deleted_at (deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Composite indexes for common queries
CREATE INDEX idx_status_created ON users(status, created_at);
CREATE INDEX idx_email_status ON users(email, status);
```

### Foreign Keys and Relationships
```sql
CREATE TABLE posts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- Foreign key with proper naming
    CONSTRAINT fk_posts_user_id
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    -- Indexes
    INDEX idx_user_id (user_id),
    INDEX idx_published_at (published_at),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Partitioning
```sql
-- Range partitioning by date
CREATE TABLE logs (
    id BIGINT UNSIGNED AUTO_INCREMENT,
    log_date DATE NOT NULL,
    message TEXT,
    level ENUM('INFO', 'WARNING', 'ERROR'),
    PRIMARY KEY (id, log_date)
) ENGINE=InnoDB
PARTITION BY RANGE (YEAR(log_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- List partitioning
CREATE TABLE sales (
    id BIGINT UNSIGNED AUTO_INCREMENT,
    region VARCHAR(50) NOT NULL,
    amount DECIMAL(10,2),
    PRIMARY KEY (id, region)
) ENGINE=InnoDB
PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('NY', 'MA', 'CT'),
    PARTITION p_south VALUES IN ('TX', 'FL', 'GA'),
    PARTITION p_west VALUES IN ('CA', 'OR', 'WA')
);
```

## Query Optimization

### Efficient Queries
```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT u.*, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.status = 'active'
GROUP BY u.id;

-- Optimize with covering indexes
CREATE INDEX idx_covering ON posts(user_id, id);

-- Use prepared statements
PREPARE stmt FROM 'SELECT * FROM users WHERE email = ?';
SET @email = 'user@example.com';
EXECUTE stmt USING @email;
DEALLOCATE PREPARE stmt;

-- Batch inserts for better performance
INSERT INTO users (email, username) VALUES
    ('user1@example.com', 'user1'),
    ('user2@example.com', 'user2'),
    ('user3@example.com', 'user3');

-- Use LIMIT for large result sets
SELECT * FROM users
WHERE status = 'active'
ORDER BY created_at DESC
LIMIT 100;
```

### Window Functions (MySQL 8.0+)
```sql
-- Ranking
SELECT
    username,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) as rank,
    RANK() OVER (ORDER BY score DESC) as rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank
FROM user_scores;

-- Running totals
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) as running_total
FROM orders;

-- Partitioned window
SELECT
    category,
    product,
    price,
    AVG(price) OVER (PARTITION BY category) as avg_category_price
FROM products;
```

### Common Table Expressions (CTE)
```sql
-- Recursive CTE for hierarchical data
WITH RECURSIVE org_chart AS (
    -- Anchor member
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;

-- Non-recursive CTE
WITH active_users AS (
    SELECT id, email, created_at
    FROM users
    WHERE status = 'active'
),
recent_posts AS (
    SELECT user_id, COUNT(*) as post_count
    FROM posts
    WHERE created_at > DATE_SUB(NOW(), INTERVAL 30 DAY)
    GROUP BY user_id
)
SELECT u.email, COALESCE(p.post_count, 0) as recent_posts
FROM active_users u
LEFT JOIN recent_posts p ON u.id = p.user_id;
```

## JSON Support

### JSON Operations
```sql
-- Create table with JSON column
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    attributes JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- Generated columns from JSON
    price DECIMAL(10,2) AS (attributes->>'$.price') STORED,
    category VARCHAR(50) AS (attributes->>'$.category') STORED,

    INDEX idx_category (category),
    INDEX idx_price (price)
);

-- Insert JSON data
INSERT INTO products (name, attributes) VALUES
('Laptop', '{"price": 999.99, "category": "electronics", "specs": {"cpu": "i7", "ram": "16GB"}}'),
('Book', '{"price": 29.99, "category": "books", "author": "John Doe"}');

-- Query JSON data
SELECT name, attributes->>'$.price' as price
FROM products
WHERE attributes->>'$.category' = 'electronics';

-- JSON array operations
SELECT name, JSON_LENGTH(attributes->'$.specs') as spec_count
FROM products;

-- Update JSON field
UPDATE products
SET attributes = JSON_SET(attributes, '$.price', 899.99)
WHERE id = 1;

-- Add to JSON array
UPDATE products
SET attributes = JSON_ARRAY_APPEND(attributes, '$.tags', 'bestseller')
WHERE id = 2;
```

## Transactions and Locking

### Transaction Management
```sql
-- ACID transaction
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Verify balances
SELECT SUM(balance) FROM accounts WHERE id IN (1, 2);

COMMIT;
-- or ROLLBACK if something went wrong

-- Transaction isolation levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Savepoints
START TRANSACTION;
INSERT INTO logs (message) VALUES ('Step 1');
SAVEPOINT sp1;

INSERT INTO logs (message) VALUES ('Step 2');
SAVEPOINT sp2;

-- Rollback to savepoint
ROLLBACK TO sp1;
COMMIT;
```

### Locking Strategies
```sql
-- Pessimistic locking
SELECT * FROM inventory
WHERE product_id = 123
FOR UPDATE;  -- Exclusive lock

-- Shared lock
SELECT * FROM inventory
WHERE product_id = 123
FOR SHARE;  -- Other transactions can read but not modify

-- Skip locked rows (queue processing)
SELECT * FROM jobs
WHERE status = 'pending'
ORDER BY created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;

-- Nowait (fail immediately if locked)
SELECT * FROM inventory
WHERE product_id = 123
FOR UPDATE NOWAIT;
```

## Replication

### Master-Replica Setup
```sql
-- On Master
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

SHOW MASTER STATUS;
-- Note the File and Position

-- On Replica
CHANGE MASTER TO
    MASTER_HOST='master_host',
    MASTER_USER='repl',
    MASTER_PASSWORD='password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=12345;

START SLAVE;
SHOW SLAVE STATUS\G

-- GTID-based replication (recommended)
CHANGE MASTER TO
    MASTER_HOST='master_host',
    MASTER_USER='repl',
    MASTER_PASSWORD='password',
    MASTER_AUTO_POSITION=1;
```

## Performance Tuning

### Server Configuration
```ini
# my.cnf / my.ini

[mysqld]
# InnoDB settings
innodb_buffer_pool_size = 8G  # 70-80% of available RAM
innodb_log_file_size = 512M
innodb_flush_log_at_trx_commit = 2  # 1 for ACID, 2 for performance
innodb_flush_method = O_DIRECT

# Connection settings
max_connections = 200
max_connect_errors = 100
connect_timeout = 10

# Query cache (deprecated in 8.0, removed)
# Use application-level caching instead

# Slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow-query.log
long_query_time = 2

# Binary logging
log_bin = mysql-bin
binlog_format = ROW
expire_logs_days = 7
max_binlog_size = 100M

# Character set
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci
```

### Query Performance
```sql
-- Analyze table statistics
ANALYZE TABLE users;

-- Optimize table (defragment)
OPTIMIZE TABLE users;

-- Show index usage
SELECT * FROM sys.schema_unused_indexes;

-- Show expensive queries
SELECT * FROM sys.statements_with_runtimes_in_95th_percentile;

-- Show table statistics
SELECT * FROM sys.schema_table_statistics
WHERE table_schema = 'mydb'
ORDER BY total_latency DESC;
```

## Security Best Practices

### User Management
```sql
-- Create user with strong password
CREATE USER 'app_user'@'localhost'
IDENTIFIED BY 'StrongP@ssw0rd!'
PASSWORD EXPIRE INTERVAL 90 DAY
FAILED_LOGIN_ATTEMPTS 3
PASSWORD_LOCK_TIME 2;

-- Grant minimal privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_user'@'localhost';

-- Read-only user
CREATE USER 'readonly'@'%' IDENTIFIED BY 'password';
GRANT SELECT ON mydb.* TO 'readonly'@'%';

-- Application user (no DDL)
CREATE USER 'app'@'10.0.0.%' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app'@'10.0.0.%';

-- Admin user (with caution)
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;

-- Show privileges
SHOW GRANTS FOR 'app_user'@'localhost';

-- Revoke privileges
REVOKE INSERT, UPDATE, DELETE ON mydb.* FROM 'app_user'@'localhost';
```

### SSL/TLS Encryption
```sql
-- Require SSL for user
CREATE USER 'secure_user'@'%'
IDENTIFIED BY 'password'
REQUIRE SSL;

-- Check SSL status
SHOW VARIABLES LIKE '%ssl%';
\s  -- Show SSL cipher in use
```

## Backup and Recovery

### Logical Backups (mysqldump)
```bash
# Full database backup
mysqldump -u root -p --all-databases > full_backup.sql

# Single database backup
mysqldump -u root -p mydb > mydb_backup.sql

# Backup with routines and triggers
mysqldump -u root -p --routines --triggers mydb > mydb_full.sql

# Restore
mysql -u root -p mydb < mydb_backup.sql

# Compressed backup
mysqldump -u root -p mydb | gzip > mydb_backup.sql.gz
gunzip < mydb_backup.sql.gz | mysql -u root -p mydb
```

### Physical Backups (MySQL Enterprise Backup / Percona XtraBackup)
```bash
# Percona XtraBackup - hot backup
xtrabackup --backup --target-dir=/backup/full

# Prepare backup
xtrabackup --prepare --target-dir=/backup/full

# Restore
xtrabackup --copy-back --target-dir=/backup/full

# Incremental backup
xtrabackup --backup --target-dir=/backup/inc1 \
    --incremental-basedir=/backup/full
```

### Point-in-Time Recovery
```bash
# Binary log replay
mysqlbinlog mysql-bin.000001 mysql-bin.000002 | mysql -u root -p

# Restore to specific time
mysqlbinlog --stop-datetime="2025-11-17 10:00:00" \
    mysql-bin.000001 | mysql -u root -p
```

## Monitoring

### Key Metrics
```sql
-- Connection statistics
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
SHOW PROCESSLIST;

-- Query performance
SHOW STATUS LIKE 'Slow_queries';
SHOW STATUS LIKE 'Questions';

-- InnoDB status
SHOW ENGINE INNODB STATUS\G

-- Table locks
SHOW STATUS LIKE 'Table_locks%';

-- Buffer pool usage
SELECT * FROM information_schema.INNODB_BUFFER_POOL_STATS;
```

## Best Practices Summary

1. **Use MySQL 8.4 LTS for production** - Long-term support and stability
2. **Explore MySQL 9.x for AI/ML** - VECTOR type for embeddings and similarity search
3. **Always use prepared statements** - Prevent SQL injection
4. **Index strategically** - Cover common queries, avoid over-indexing
5. **Use utf8mb4 character set** - Full Unicode support including emojis
6. **Enable binary logging** - Essential for replication and point-in-time recovery
7. **Monitor slow queries** - Optimize bottlenecks regularly
8. **Implement proper backup strategy** - Test restores regularly
9. **Use InnoDB engine** - ACID compliance and better performance
10. **Keep MySQL updated** - Security patches and performance improvements

## When Helping Users

1. **Ask about MySQL version** - Features vary significantly between versions
2. **Recommend MySQL 8.4 LTS** - For production systems requiring long-term support
3. **Suggest VECTOR type for AI/ML** - If working with embeddings (MySQL 9.0+)
4. **Emphasize proper indexing** - Critical for performance
5. **Validate data types** - Use appropriate types for data
6. **Implement transactions** - For data integrity
7. **Security first** - Principle of least privilege
8. **Plan for scale** - Replication, partitioning, sharding strategies

Your goal is to help users build robust, performant, and secure MySQL databases using the latest features and industry best practices.
