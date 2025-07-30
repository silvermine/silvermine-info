# Silvermine Coding Standards Specific to SQL

## Formatting

SQL statements should be easy to read, understand, and maintain.

### Indentation and Alignment

   * Vertically align sibling elements of queries for improved readability
   * Indent new lines in statements and queries
   * Align related keywords and clauses at the same indentation level
   * When splitting comma-delimited text across multiple lines, start the new line with a
     comma
   * Break lines at keywords (JOIN, WHERE, AND, OR)
   * Terminate statements with a semicolon on a new line

```sql
-- Good
SELECT u.userID
     , u.firstName
     , u.lastName
     , p.profileImageUrl
  FROM Users u
  JOIN UserProfiles p
    ON u.userID = p.userID
 WHERE u.isActive = 1
   AND u.createdDate >= '2023-01-01'
 ORDER BY u.lastName
        , u.firstName
;
-- Bad
SELECT u.userID, u.firstName, u.lastName, p.profileImageUrl FROM Users u
   JOIN UserProfiles p ON u.userID = p.userID WHERE u.isActive = 1
   AND u.createdDate >= '2023-01-01' ORDER BY u.lastName, u.firstName;
```

### Keyword Capitalization

   * Use UPPERCASE for all SQL keywords and functions

```sql
-- Good
SELECT COUNT(*) AS userCount
  FROM Users
 WHERE createdDate >= CURRENT_DATE - INTERVAL 30 DAY
;

-- Bad
select count(*) as userCount
   from Users
   where createdDate >= current_date - interval 30 day
;
```

## Naming Conventions

### Column Names

   * Use camel case for all column names
   * Use aliases when necessary to ensure camel case is maintained in result sets
   * Be descriptive and avoid abbreviations unless they improve readability
   * Capitalize "ID" suffix in column names [Special Cases](https://github.com/silvermine/silvermine-info/blob/master/coding-standards.md#special-cases)

```sql
-- Good
SELECT u.userID
     , u.firstName
     , u.lastName
     , u.emailAddress
     , p.profileImageUrl
     , p.lastLoginDate
  FROM Users u
  JOIN UserProfiles p 
    ON u.userID = p.userID
;

-- Bad
SELECT u.user_id
     , u.first_name
     , u.last_name
     , u.email_addr
     , p.profile_img_url
     , p.last_login_dt
  FROM Users u
  JOIN UserProfiles p 
    ON u.user_id = p.userId;
```

### Table and View Names

   * Use pascal case for table and view names
   * Use descriptive names that clearly indicate the entity
   * Avoid unnecessary prefixes or suffixes
   * Use the plural form of the entity, i.e. use "Users" not "User"

```sql
-- Good
CREATE TABLE UserProfiles (
   profileID INTEGER PRIMARY KEY
 , userID INTEGER NOT NULL
 , profileImageUrl TEXT
 , bio TEXT
 , createdDate DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Bad
CREATE TABLE tbl_user_profiles (
   profile_ID INTEGER PRIMARY KEY
 , user_ID INTEGER NOT NULL
 , profile_img_url TEXT
 , bio TEXT
 , created_dt DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Aliases

   * Use meaningful aliases that are short but clear
   * Prefer one-letter or two-letter aliases for simple cases, descriptive abbreviations
     for complex queries
   * Be consistent with alias usage throughout a query

```sql
-- Good
SELECT u.firstName
     , u.lastName
     , addr.streetAddress
     , addr.city
     , addr.postalCode
  FROM Users u
  JOIN UserAddresses addr 
    ON u.userID = addr.userID
;

-- Also acceptable for simple cases
SELECT u.firstName
     , u.lastName
     , a.streetAddress
  FROM Users u
  JOIN UserAddresses a 
    ON u.userID = a.userID
;

-- Acceptable two-letter case
SELECT u1.firstName
     , u1.lastName
     , u2.firstName
     , u2.lastName
     , addr.streetAddress
     , addr.city
     , addr.postalCode
  FROM Users u1
  JOIN UserAddresses addr
    ON u1.userID = addr.userID
  JOIN UserAddresses addrDuplicate
    ON addr.streetAddress = addrDuplicate.streetAddress
   AND addr.city = addrDuplicate.city
   AND addr.postalCode = addrDuplicate.postalCode
  JOIN Users u2
    ON addrDuplicate.userID = u2.userID
;
```

## Query Structure

### SELECT Statements

   * List columns in a logical order (typically: IDs first, then descriptive fields, then
     dates)
   * Use explicit column names rather than SELECT *
   * Group related columns together

```sql
-- Good
SELECT u.userID
     , u.firstName
     , u.lastName
     , u.emailAddress
     , u.isActive
     , u.createdDate
     , u.lastModifiedDate
  FROM Users u
 WHERE u.isActive = 1
;
```

### JOIN Clauses

   * Use explicit JOIN syntax (INNER JOIN, LEFT JOIN, etc.)
   * Align ON conditions with the JOIN keyword
   * Use parentheses for complex join conditions

```sql
-- Good
SELECT u.firstName
     , u.lastName
     , o.orderTotal
  FROM Users u
 INNER JOIN Orders o
    ON u.userID = o.userID
   AND o.orderStatus = 'completed'
  LEFT JOIN UserProfiles p
    ON u.userID = p.userID
   AND p.isPublic = 1
;
```

### WHERE Clauses

   * Place each condition on its own line for complex queries
      * Follow our principles of readability and vertical alignment for long or nested
        conditions
   * Use parentheses to clarify the order of operations
   * Group related conditions together

```sql
-- Good
SELECT *
  FROM Orders
 WHERE orderStatus IN ('pending', 'processing')
   AND orderDate >= '2023-01-01'
   AND (orderTotal > 100.00 OR customerType = 'premium')
   AND shippingAddress IS NOT NULL
;

-- Good: Long and nested conditions with proper vertical alignment
SELECT o.orderID
     , o.orderDate
     , c.customerName
     , p.productName
  FROM Orders o
  JOIN Customers c
    ON o.customerID = c.customerID
  JOIN OrderItems oi 
    ON o.orderID = oi.orderID
  JOIN Products p
    ON oi.productID = p.productID
  JOIN Categories pc
    ON p.categoryID = pc.categoryID
   AND pc.categoryName IN ('Electronics', 'Books', 'Clothing')
   AND pc.isActive = 1
 WHERE (
             o.orderStatus IN ('pending', 'processing', 'shipped')
         AND o.orderDate BETWEEN '2023-01-01' AND '2023-12-31'
         AND o.orderTotal > 50.00
       )
   AND (
             c.customerType = 'premium'
          OR (
                   c.customerType = 'standard'
               AND c.loyaltyPoints >= 1000
               AND c.accountCreatedDate <= '2022-12-31'
             )
       )
;
```

## Performance Guidelines

### Indexing Strategy

   * Create indexes for columns that are frequently used in WHERE clauses
   * Create indexes if necessary for columns used in JOIN conditions
   * Create composite indexes for multi-column queries
   * Be mindful of the cost of maintaining indexes on frequently-updated tables

```sql
-- Good indexing examples
CREATE INDEX idxUsersEmail ON users(emailAddress);
CREATE INDEX idxOrdersUserIDOrderDate ON orders(userID, orderDate);
CREATE INDEX idxUsersActiveCreated ON users(isActive, createdDate);
```

### SQLite-Specific Performance

   * Use `EXPLAIN QUERY PLAN` to verify that queries use indexes effectively
   * Avoid table scans unless you're confident the table will remain small
   * Use appropriate data types to minimize storage and improve performance

```sql
-- Check query performance
EXPLAIN QUERY PLAN
SELECT u.firstName
     , u.lastName
  FROM Users u
 WHERE u.emailAddress = 'user@example.com'
;

-- You should see "USING INDEX" rather than "SCAN TABLE"
```

### Query Optimization

   * Use EXISTS instead of IN for subqueries when checking for existence
   * Limit result sets with appropriate WHERE conditions
   * Use LIMIT when you only need a subset of results

```sql
-- Good - using EXISTS
SELECT u.firstName
     , u.lastName
  FROM Users u
 WHERE EXISTS (
    SELECT 1
      FROM Orders o
     WHERE o.userID = u.userID
       AND o.orderStatus = 'completed'
    );

-- Less efficient - using IN
SELECT u.firstName
     , u.lastName
  FROM Users u
 WHERE u.userID IN (
    SELECT o.userID
      FROM Orders o
     WHERE o.orderStatus = 'completed'
    );
```

## Data Definition Language (DDL)

### Table Creation

   * Define primary keys explicitly
   * Use appropriate data types
   * Include NOT NULL constraints where appropriate
   * Add meaningful default values

```sql
CREATE TABLE Users (
   userID INTEGER PRIMARY KEY AUTOINCREMENT
 , firstName TEXT NOT NULL
 , lastName TEXT NOT NULL
 , emailAddress TEXT NOT NULL UNIQUE
 , isActive INTEGER NOT NULL DEFAULT 1
 , createdDate DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
 , lastModifiedDate DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Foreign Key Constraints

   * Define foreign key relationships explicitly
   * Use meaningful constraint names
   * Consider cascading actions carefully

```sql
CREATE TABLE Orders (
   orderID INTEGER PRIMARY KEY AUTOINCREMENT
 , userID INTEGER NOT NULL
 , orderTotal DECIMAL(10,2) NOT NULL
 , orderStatus TEXT NOT NULL DEFAULT 'pending'
 , createdDate DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
 , FOREIGN KEY (userID) REFERENCES Users(userID)
   ON DELETE CASCADE
   ON UPDATE CASCADE
);
```

## Comments and Documentation

### Inline Comments

   * Use comments to explain complex business logic
   * Document non-obvious performance considerations
   * Explain the purpose of complex subqueries

```sql
-- Calculate the average order value for active users
-- who have placed at least 3 orders in the last year
SELECT u.userID
     , u.firstName
     , u.lastName
     , AVG(o.orderTotal) AS averageOrderValue
  FROM Users u
 INNER JOIN Orders o
    ON u.userID = o.userID
   AND o.orderDate >= DATE('now', '-1 year')  -- Last 12 months only
 WHERE u.isActive = 1
 GROUP BY u.userID
        , u.firstName
        , u.lastName
HAVING COUNT(o.orderID) >= 3 -- Minimum 3 orders required
;
```

### Header Comments

   * Include purpose and context for complex queries
   * Document any assumptions or business rules
   * Note performance characteristics for critical queries

```sql
/**
 * User Activity Report Query
 * 
 * Purpose: Generate monthly activity report showing user engagement metrics
 * Performance: Uses composite index on (userID, activityDate) - typically <100ms
 * Business Rules: Only includes activities from verified users
 */
SELECT u.userID
     , u.firstName
     , u.lastName
     , COUNT(a.activityID) AS totalActivities
     , MAX(a.activityDate) AS lastActivityDate
  FROM Users u
  LEFT JOIN UserActivities a
    ON u.userID = a.userID
   AND a.activityDate >= DATE('now', 'start of month')
 WHERE u.isVerified = 1
 GROUP BY u.userID
        , u.firstName
        , u.lastName
 ORDER BY totalActivities DESC
;
```

## Best Practices

### Security

   * Use parameterized queries to prevent SQL injection
   * Avoid dynamic SQL construction with string concatenation
   * Limit database permissions to the minimum required

### Maintainability

   * Keep queries focused on a single purpose
   * Break complex queries into views or Common Table Expressions (CTE) when appropriate
      * Note that SQLite supports both ordinary and recursive expressions
   * Test queries with realistic data volumes

### Error Handling

   * Consider NULL values in your logic
   * Use appropriate data types to prevent type conversion errors
   * Test edge cases (empty results, large datasets, etc.)

```sql
-- Good - handles NULL values explicitly
SELECT u.firstName
     , u.lastName
     , COALESCE(p.profileImageUrl, '/images/default-avatar.png') AS profileImageUrl
  FROM Users u
  LEFT JOIN UserProfiles p 
    ON u.userID = p.userID
;
```

## Common Patterns

### Pagination

```sql
-- Efficient pagination using LIMIT and OFFSET
SELECT u.userID
     , u.firstName
     , u.lastName
  FROM Users u
 WHERE u.isActive = 1
 ORDER BY u.lastName
        , u.firstName
 LIMIT 20 OFFSET 40 -- Page 3, 20 items per page
;
```

### Conditional Logic

```sql
-- Use CASE statements for conditional logic
SELECT u.firstName
     , u.lastName
     , CASE 
       WHEN u.lastLoginDate >= DATE('now', '-7 days') THEN 'Active'
       WHEN u.lastLoginDate >= DATE('now', '-30 days') THEN 'Inactive'
       ELSE 'Dormant'
       END AS userStatus
  FROM Users u
;
```

### Aggregation with Grouping

```sql
-- Clear grouping and aggregation
SELECT DATE(o.orderDate) AS orderDay
     , COUNT(*) AS totalOrders
     , SUM(o.orderTotal) AS dailyRevenue
     , AVG(o.orderTotal) AS averageOrderValue
  FROM Orders o
 WHERE o.orderStatus = 'completed'
   AND o.orderDate >= DATE('now', '-30 days')
 GROUP BY DATE(o.orderDate)
 ORDER BY orderDay DESC
;
```
