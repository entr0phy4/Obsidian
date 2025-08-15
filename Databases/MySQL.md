# MySQL

## Overview
MySQL is an open-source relational database management system (RDBMS) based on Structured Query Language (SQL). It's one of the most popular databases for web applications and is a central component of the LAMP stack.

## Architecture

### Storage Engines
- **InnoDB**: Default engine, supports transactions, foreign keys, row-level locking
- **MyISAM**: Fast but no transactions, table-level locking
- **Memory**: Data stored in RAM, temporary tables
- **Archive**: Compressed storage for archival data

### Components
- **MySQL Server** (mysqld): Core database engine
- **MySQL Client** (mysql): Command-line interface
- **MySQL Utilities**: Administration tools (mysqldump, mysqlcheck, etc.)

## Installation

### Ubuntu/Debian
```bash
# Update package index
sudo apt update

# Install MySQL server
sudo apt install mysql-server

# Secure installation
sudo mysql_secure_installation

# Start MySQL service
sudo systemctl start mysql
sudo systemctl enable mysql
```

### CentOS/RHEL
```bash
# Install MySQL repository
sudo yum install mysql-server

# Or for newer versions
sudo dnf install mysql-server

# Start and enable MySQL
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Get temporary root password
sudo grep 'temporary password' /var/log/mysqld.log
```

### Docker Installation
```bash
# Run MySQL in Docker
docker run -d \
  --name mysql-server \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppass \
  -v mysql_data:/var/lib/mysql \
  mysql:8.0
```

## Basic Operations

### Connection
```bash
# Connect to MySQL
mysql -u username -p
mysql -u username -p -h hostname -P port database_name

# Connect with SSL
mysql -u username -p --ssl-mode=REQUIRED

# Connect to remote server
mysql -u username -p -h remote-server.com -P 3306
```

### Database Management
```sql
-- Show databases
SHOW DATABASES;

-- Create database
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Use database
USE myapp;

-- Drop database
DROP DATABASE myapp;

-- Show current database
SELECT DATABASE();
```

### Table Operations
```sql
-- Show tables
SHOW TABLES;

-- Create table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB;

-- Show table structure
DESCRIBE users;
SHOW CREATE TABLE users;

-- Drop table
DROP TABLE users;

-- Rename table
RENAME TABLE old_name TO new_name;
```

### Data Types

### Numeric Types
```sql
TINYINT     -- 1 byte, -128 to 127
SMALLINT    -- 2 bytes, -32,768 to 32,767
MEDIUMINT   -- 3 bytes
INT         -- 4 bytes, -2,147,483,648 to 2,147,483,647
BIGINT      -- 8 bytes
DECIMAL(M,D)-- Fixed-point number
FLOAT       -- Single precision floating point
DOUBLE      -- Double precision floating point
```

### String Types
```sql
CHAR(n)     -- Fixed length string
VARCHAR(n)  -- Variable length string
TEXT        -- Large text data
TINYTEXT    -- Small text (255 characters)
MEDIUMTEXT  -- Medium text (16MB)
LONGTEXT    -- Large text (4GB)
BINARY      -- Fixed-length binary string
VARBINARY   -- Variable-length binary string
BLOB        -- Binary large object
```

### Date and Time Types
```sql
DATE        -- YYYY-MM-DD
TIME        -- HH:MM:SS
DATETIME    -- YYYY-MM-DD HH:MM:SS
TIMESTAMP   -- YYYY-MM-DD HH:MM:SS (with timezone)
YEAR        -- YYYY
```

## CRUD Operations

### INSERT
```sql
-- Insert single row
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john@example.com', 'hashed_password');

-- Insert multiple rows
INSERT INTO users (username, email, password_hash) VALUES
('alice', 'alice@example.com', 'hash1'),
('bob', 'bob@example.com', 'hash2'),
('carol', 'carol@example.com', 'hash3');

-- Insert from another table
INSERT INTO user_backup (username, email)
SELECT username, email FROM users WHERE created_at < '2023-01-01';

-- Insert with ON DUPLICATE KEY UPDATE
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'newemail@example.com', 'newhash')
ON DUPLICATE KEY UPDATE email = VALUES(email);
```

### SELECT
```sql
-- Basic select
SELECT * FROM users;
SELECT username, email FROM users;

-- WHERE clause
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE username LIKE 'john%';
SELECT * FROM users WHERE created_at >= '2023-01-01';

-- ORDER BY and LIMIT
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- GROUP BY and aggregation
SELECT COUNT(*) as user_count FROM users;
SELECT DATE(created_at) as date, COUNT(*) as daily_signups
FROM users
GROUP BY DATE(created_at)
ORDER BY date DESC;

-- JOIN operations
SELECT u.username, p.title
FROM users u
JOIN posts p ON u.id = p.user_id
WHERE u.active = 1;
```

### UPDATE
```sql
-- Update single row
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Update multiple rows
UPDATE users SET active = 0 WHERE last_login < '2023-01-01';

-- Update with JOIN
UPDATE users u
JOIN user_stats s ON u.id = s.user_id
SET u.status = 'premium'
WHERE s.total_purchases > 1000;
```

### DELETE
```sql
-- Delete specific rows
DELETE FROM users WHERE id = 1;
DELETE FROM users WHERE active = 0 AND created_at < '2022-01-01';

-- Delete with JOIN
DELETE u FROM users u
JOIN inactive_users i ON u.id = i.user_id;

-- Truncate table (faster for all rows)
TRUNCATE TABLE temp_table;
```

## Indexes and Performance

### Index Types
```sql
-- Primary key (automatically indexed)
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY
);

-- Unique index
CREATE UNIQUE INDEX idx_username ON users(username);

-- Regular index
CREATE INDEX idx_email ON users(email);

-- Composite index
CREATE INDEX idx_name_email ON users(last_name, first_name, email);

-- Full-text index
CREATE FULLTEXT INDEX idx_content ON articles(title, content);

-- Show indexes
SHOW INDEX FROM users;

-- Drop index
DROP INDEX idx_email ON users;
```

### Query Optimization
```sql
-- Explain query execution plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Analyze table
ANALYZE TABLE users;

-- Optimize table
OPTIMIZE TABLE users;

-- Show query performance
SHOW PROFILES;
SET profiling = 1;
-- Run your query
SHOW PROFILE FOR QUERY 1;
```

## User Management

### Create Users
```sql
-- Create user
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'secure_password';
CREATE USER 'appuser'@'%' IDENTIFIED BY 'secure_password';

-- Create user with authentication plugin
CREATE USER 'appuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

### Grant Permissions
```sql
-- Grant database-specific permissions
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'appuser'@'localhost';

-- Grant all privileges on a database
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';

-- Grant specific table permissions
GRANT SELECT ON myapp.users TO 'readonly'@'localhost';

-- Grant global permissions
GRANT PROCESS, REPLICATION CLIENT ON *.* TO 'monitor'@'localhost';

-- Refresh privileges
FLUSH PRIVILEGES;
```

### Show and Revoke Permissions
```sql
-- Show user privileges
SHOW GRANTS FOR 'appuser'@'localhost';

-- Revoke permissions
REVOKE DELETE ON myapp.* FROM 'appuser'@'localhost';

-- Drop user
DROP USER 'appuser'@'localhost';
```

## Backup and Restore

### mysqldump
```bash
# Backup single database
mysqldump -u username -p database_name > backup.sql

# Backup with specific options
mysqldump -u username -p \
  --single-transaction \
  --routines \
  --triggers \
  database_name > backup.sql

# Backup all databases
mysqldump -u username -p --all-databases > all_databases.sql

# Backup specific tables
mysqldump -u username -p database_name table1 table2 > tables.sql

# Backup with compression
mysqldump -u username -p database_name | gzip > backup.sql.gz
```

### Restore
```bash
# Restore database
mysql -u username -p database_name < backup.sql

# Restore compressed backup
gunzip < backup.sql.gz | mysql -u username -p database_name

# Restore with progress
pv backup.sql | mysql -u username -p database_name
```

### Binary Log Backups
```bash
# Enable binary logging in my.cnf
log-bin=mysql-bin
binlog-format=ROW

# Show binary logs
SHOW BINARY LOGS;

# Backup binary logs
mysqlbinlog mysql-bin.000001 > binlog.sql

# Point-in-time recovery
mysqlbinlog --stop-datetime="2023-12-01 10:00:00" mysql-bin.000001 | mysql -u root -p
```

## Replication

### Master Configuration
```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
binlog-do-db = myapp
```

### Slave Configuration
```ini
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
log-slave-updates = 1
read-only = 1
```

### Setup Replication
```sql
-- On master: create replication user
CREATE USER 'replica'@'%' IDENTIFIED BY 'replica_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;

-- Get master status
SHOW MASTER STATUS;

-- On slave: configure replication
CHANGE MASTER TO
    MASTER_HOST='master_ip',
    MASTER_USER='replica',
    MASTER_PASSWORD='replica_password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;

-- Start slave
START SLAVE;

-- Check slave status
SHOW SLAVE STATUS\G
```

## Configuration

### Important Configuration Options
```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
# Basic settings
datadir = /var/lib/mysql
socket = /var/run/mysqld/mysqld.sock
port = 3306
bind-address = 127.0.0.1

# Character set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# InnoDB settings
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
innodb_log_buffer_size = 64M
innodb_flush_log_at_trx_commit = 1

# Query cache (deprecated in MySQL 8.0)
query_cache_type = 1
query_cache_size = 64M

# Connection settings
max_connections = 200
wait_timeout = 300
interactive_timeout = 300

# Logging
log-error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

## Security

### Security Best Practices
```sql
-- Remove test database
DROP DATABASE IF EXISTS test;

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Disable root remote login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- Set strong passwords
ALTER USER 'root'@'localhost' IDENTIFIED BY 'strong_password';

-- Enable SSL
-- Add to my.cnf:
-- ssl-ca=ca-cert.pem
-- ssl-cert=server-cert.pem
-- ssl-key=server-key.pem
```

### Encryption
```sql
-- Enable encryption at rest
-- In my.cnf:
-- innodb_encrypt_tables=ON
-- innodb_encrypt_log=ON

-- SSL connection enforcement
CREATE USER 'secure_user'@'%' IDENTIFIED BY 'password' REQUIRE SSL;

-- Check SSL status
SHOW STATUS LIKE 'Ssl_cipher';
```

## Monitoring

### Performance Monitoring
```sql
-- Show processlist
SHOW PROCESSLIST;

-- Show engine status
SHOW ENGINE INNODB STATUS;

-- Performance schema queries
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY sum_timer_wait DESC LIMIT 10;

-- Show variables
SHOW VARIABLES LIKE 'innodb%';
SHOW STATUS LIKE 'Connections';
```

### Log Analysis
```bash
# Monitor error log
tail -f /var/log/mysql/error.log

# Analyze slow query log
mysqldumpslow /var/log/mysql/slow.log

# Use pt-query-digest (Percona Toolkit)
pt-query-digest /var/log/mysql/slow.log
```

## Troubleshooting

### Common Issues
```bash
# Can't connect to MySQL server
# Check if MySQL is running
sudo systemctl status mysql

# Check port is listening
netstat -tulpn | grep :3306

# Check MySQL error log
sudo tail -f /var/log/mysql/error.log

# Reset root password
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables &
mysql -u root
UPDATE mysql.user SET authentication_string=PASSWORD('newpassword') WHERE User='root';
FLUSH PRIVILEGES;
```

### Performance Issues
```sql
-- Check for locks
SHOW ENGINE INNODB STATUS;

-- Check table status
SHOW TABLE STATUS;

-- Identify slow queries
SELECT * FROM performance_schema.events_statements_summary_by_digest
WHERE sum_timer_wait > 1000000000000
ORDER BY sum_timer_wait DESC;
```

## Related Tools
- [[mysqldump]] - Database backup utility
- [[Percona Toolkit]] - Database administration tools
- [[phpMyAdmin]] - Web-based administration
- [[MySQL Workbench]] - GUI administration tool

## Related Topics
- [[Database design]]
- [[SQL optimization]]
- [[Database replication]]
- [[Database security]]
- [[ACID properties]]
