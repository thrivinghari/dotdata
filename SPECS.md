# DotData File Format - Complete Technical Specification

## Document Purpose

This specification provides comprehensive technical documentation for the `.data` file format - a MongoDB operations language designed for database testing and data management. This document is intended for AI systems, parsers, and developers who need to understand, implement, or work with the format.

## Format Overview

The `.data` format is a domain-specific language (DSL) that:
- Uses **markdown-inspired syntax** for readability
- Supports **flexible single-line/multi-line** operations based on complexity
- Provides **100% MongoDB operation coverage** for testing scenarios
- Includes **variable substitution** and **built-in functions**
- Uses **string-based IDs** throughout (never MongoDB ObjectId)
- Supports **hybrid raw MongoDB syntax** for complex scenarios
- Includes **performance tuning** and **mathematical operations**

## File Structure and Parsing Rules

### File Organization
```
1. Database connection (required, first non-comment line)
2. Variable declarations (optional)
3. Sectioned operations (INSERT, UPDATE, DELETE, FIND, etc.)
4. Cleanup section (optional, typically at end)
```

### Section Format
**Syntax**: `=== Section Name === // Optional comments`

**Examples**:
```data
=== Foundation Data === // Core accounts and users
INSERT accounts { "_id": "test-account", "name": "Test" }
INSERT users { "_id": "test-user", "accountId": "test-account" }

=== Business Logic Tests === // Main application logic
UPDATE users WHERE _id = "test-user" SET loginCount += 1
FIND users WHERE accountId = "test-account" SELECT name, loginCount

=== Cleanup === // Remove test data
DELETE users WHERE accountId = "test-account"
DELETE accounts WHERE _id = "test-account"
```

### Line Types
- **Comments**: Lines starting with `###` (documentation) or `#` (inline)
- **Database Connection**: `MONGO <connection_string>`
- **Variables**: `@variableName = value`
- **Sections**: `=== Section Name === // optional comments`
- **Operations**: Various MongoDB operations within sections
- **Empty Lines**: Ignored for parsing

### Character Encoding
- UTF-8 encoding required
- Line endings: `\n`, `\r\n`, or `\r` supported
- Indentation: 2 spaces for nested structures

## Core Language Elements

### 1. Database Connection (Required)

**Syntax**: `MONGO <hostname>:<port>/<database>`

**Example**: `MONGO localhost:27017/roadwarrior`

**Rules**:
- Must be first non-comment line
- Only one MONGO statement per file
- Database name becomes context for all operations

### 2. Variable System

**Declaration Syntax**: `@<variableName> = <value>`

**Usage Syntax**: `{{<variableName>}}`

**Examples**:
```data
@accountId = test-account-123
@userId = user-{{$randomUUID}}

INSERT users { "_id": "{{userId}}", "AccountId": "{{accountId}}" }
```

**Rules**:
- Variables must be declared before use
- Variable names: alphanumeric, underscore, hyphen
- Values can be literals or function calls
- Case-sensitive

### 3. Built-in Functions

| Function | Return Type | Description | Example |
|----------|-------------|-------------|---------|
| `{{$now}}` | ISO String | Current timestamp | `"2024-01-15T10:30:00Z"` |
| `{{$timestamp}}` | Number | Unix timestamp | `1642248600` |
| `{{$randomGUID}}` | String | Random Microsoft GUID | `"A1B2C3D4-E5F6-7890-ABCD-123456789012"` |
| `{{$randomUUID}}` | String | Random RFC UUID | `"a1b2c3d4-e5f6-7890-abcd-123456789012"` |
| `{{$randomObjectId}}` | String | Random ObjectId format | `"507f1f77bcf86cd799439020"` |
| `{{$futureDate:Xd}}` | ISO String | X days in future | `{{$futureDate:30d}}` |
| `{{$pastDate:Xd}}` | ISO String | X days in past | `{{$pastDate:7d}}` |
| `{{$futureDate:Xh}}` | ISO String | X hours in future | `{{$futureDate:24h}}` |
| `{{$pastDate:Xm}}` | ISO String | X minutes in past | `{{$pastDate:30m}}` |
| `{{$randomInt:min:max}}` | Number | Random integer | `{{$randomInt:1:100}}` |
| `{{$randomFloat:min:max}}` | Number | Random float | `{{$randomFloat:1.0:100.0}}` |
| `{{$randomDate:start:end}}` | ISO String | Random date | `{{$randomDate:2023-01-01:2024-01-01}}` |
| `{{$randomChoice:val1,val2}}` | Any | Random choice | `{{$randomChoice:red,blue,green}}` |
| `{{$randomBool}}` | Boolean | Random boolean | `true` or `false` |
| `{{$index}}` | Number | Current index (load tests) | `1`, `2`, `3`, ... |
| `{{$dateFormat:date:format}}` | String | Format date | `{{$dateFormat:{{$now}}:YYYY-MM-DD}}` |

**Function Parsing Rules**:
- Functions always wrapped in `{{` and `}}`
- Parameters separated by `:`
- Date parameters support `d` (days), `h` (hours), `m` (minutes), `s` (seconds)

## Flexible Syntax System

### Single-Line vs Multi-Line Rules

**Use Single-Line When**:
- Single condition in WHERE clause
- Single field in SET/SELECT clause
- Simple operations

**Use Multi-Line When**:
- Multiple conditions in WHERE clause
- Multiple fields in SET/SELECT clause
- Complex operations

### Single-Line Examples
```data
DELETE users WHERE _id = "{{userId}}"
UPDATE users WHERE _id = "{{userId}}" SET LastLogin = "{{$now}}"
FIND users WHERE UserType = 1 SELECT Name, Email
CREATE_INDEX ON users FIELDS Email ASC UNIQUE
```

### Multi-Line Examples
```data
DELETE users  
WHERE:
  - IsActive = false
  - Created < "{{$pastDate:30d}}"

UPDATE users
WHERE:
  - UserType = 1
  - IsActive = true
SET:
  - LoginCount += 1
  - LastSeen = "{{$now}}"

FIND users
WHERE:
  - UserType IN [1, 2]
  - IsActive = true
SELECT:
  - Name
  - Email
  - Created
```

## Complete Operation Reference

### INSERT Operations

**Single Document**:
```data
INSERT <collection>
{
  <json_document>
}
```

**Multiple Documents**:
```data
INSERT_MANY <collection>
[
  { <json_document> },
  { <json_document> }
]
```

**Rules**:
- JSON must be valid
- Use string IDs (never ObjectId)
- Variable substitution in values

### UPDATE Operations

**Single-Line Syntax** (simple case):
```data
UPDATE <collection> WHERE <condition> SET <assignment>
```

**Multi-Line Syntax** (complex case):
```data
UPDATE <collection>
WHERE:
  - <condition>
  - <condition>
SET:
  - <assignment>
  - <assignment>
```

**Assignment Operators**:
- `=` - Set value
- `+=` - Increment (MongoDB `$inc`)
- `-=` - Decrement
- `PUSH` - Add to array (MongoDB `$push`)
- `REMOVE` - Remove from array (MongoDB `$pull`)
- `ADD_UNIQUE` - Add unique to array (MongoDB `$addToSet`)
- `REMOVE_ALL` - Remove multiple from array (MongoDB `$pullAll`)
- `UNSET` - Remove field (MongoDB `$unset`)
- `NOW` - Set current date (MongoDB `$currentDate`)

### UPSERT Operations

**Syntax**: Same as UPDATE but with `UPSERT` keyword

**Additional Clause**:
```data
SET_ON_INSERT:
  - <assignment>
  - <assignment>
```

Maps to MongoDB `$setOnInsert` operator.

### DELETE Operations

**Single-Line Syntax**:
```data
DELETE <collection> WHERE <condition>
```

**Multi-Line Syntax**:
```data
DELETE <collection>
WHERE:
  - <condition>
  - <condition>
```

### FIND Operations

**Single-Line Syntax**:
```data
FIND <collection> [WHERE <condition>] [SELECT <fields>] [SORT <field>] [LIMIT <n>] [EXPECT <result>]
```

**Multi-Line Syntax**:
```data
FIND <collection>
[WHERE:
  - <condition>
  - <condition>]
[SELECT:
  - <field>
  - <field>]
[SORT:
  - <field> <direction>
  - <field> <direction>]
[LIMIT <number>]
[SKIP <number>]
[EXPECT <expected_result>]
```

**Field Selection**:
- Simple field: `fieldName`
- Nested field: `profile.address.city`
- Alias: `fieldName AS alias`

**Sort Directions**: `ASC`, `DESC`

**Expect Values**:
- Count: `3 results` or `1 result`
- Data: `[{...}]` (JSON array)

### Query Operators

| Operator | MongoDB Equivalent | Example |
|----------|-------------------|---------|
| `=` | `$eq` | `status = "active"` |
| `!=` | `$ne` | `status != "deleted"` |
| `>` | `$gt` | `age > 18` |
| `>=` | `$gte` | `score >= 90` |
| `<` | `$lt` | `price < 100` |
| `<=` | `$lte` | `rating <= 5` |
| `IN` | `$in` | `category IN ["tech", "science"]` |
| `NOT IN` | `$nin` | `status NOT IN ["banned", "suspended"]` |
| `CONTAINS` | `$regex` | `name CONTAINS "John"` |
| `STARTS_WITH` | `$regex` | `email STARTS_WITH "admin"` |
| `ENDS_WITH` | `$regex` | `domain ENDS_WITH ".com"` |
| `MATCHES` | `$regex` | `phone MATCHES "^\\+1"` |
| `LIKE` | `$regex` | `email LIKE "admin@%"` |
| `BETWEEN` | `$gte` + `$lte` | `created BETWEEN "2023-01-01" AND "2023-12-31"` |
| `EXISTS` | `$exists: true` | `phoneNumber EXISTS` |
| `IS_NUMBER` | `$type: "number"` | `age IS_NUMBER` |
| `IS_NOT_NULL` | `$ne: null` | `profile IS_NOT_NULL` |
| `HAS_ALL` | `$all` | `permissions HAS_ALL ["read", "write"]` |
| `SIZE` | `$size` | `tags SIZE > 3` |
| `ELEM_MATCH` | `$elemMatch` | `items ELEM_MATCH { price > 100 }` |

### Logical Operators

**Implicit AND**: Multiple conditions in bullet list
```data
WHERE:
  - status = "active"
  - age > 18
```

**Explicit Grouping**:
```data
WHERE:
  - status = "active"
  - AND (
      - age > 18
      - OR verified = true
    )
```

## Complex Mathematical Operations

### Date Math Functions

| Function | MongoDB Equivalent | Description | Example |
|----------|-------------------|-------------|---------|
| `YEAR(field)` | `{ $year: "$field" }` | Extract year | `YEAR(created) = 2024` |
| `MONTH(field)` | `{ $month: "$field" }` | Extract month | `MONTH(created) = 12` |
| `DAY(field)` | `{ $dayOfMonth: "$field" }` | Extract day | `DAY(created) > 15` |
| `HOUR(field)` | `{ $hour: "$field" }` | Extract hour | `HOUR(timestamp) BETWEEN 9 AND 17` |
| `MINUTE(field)` | `{ $minute: "$field" }` | Extract minute | `MINUTE(recorded) = 0` |
| `SECOND(field)` | `{ $second: "$field" }` | Extract second | `SECOND(event) = 0` |
| `DAY_OF_WEEK(field)` | `{ $dayOfWeek: "$field" }` | Day of week (1-7) | `DAY_OF_WEEK(created) NOT IN [1, 7]` |
| `DAY_OF_YEAR(field)` | `{ $dayOfYear: "$field" }` | Day of year (1-366) | `DAY_OF_YEAR(created) > 100` |
| `WEEK(field)` | `{ $week: "$field" }` | Week of year | `WEEK(created) = 25` |

**Date Arithmetic Functions**:
```data
# Date arithmetic
FIND events WHERE created > DATE_ADD("{{$now}}", -30, "days")
FIND sessions WHERE expires < DATE_SUB("{{$now}}", 1, "hour")

# Complex date conditions
FIND logs
WHERE:
  - YEAR(timestamp) = 2024
  - MONTH(timestamp) IN [1, 2, 3]
  - DAY_OF_WEEK(timestamp) NOT IN [1, 7]  # Exclude weekends
```

### Mathematical Expression Functions

| Function | MongoDB Equivalent | Description | Example |
|----------|-------------------|-------------|---------|
| `ABS(field)` | `{ $abs: "$field" }` | Absolute value | `ABS(score - 50) < 10` |
| `SQRT(field)` | `{ $sqrt: "$field" }` | Square root | `SQRT(value) > 5` |
| `POWER(base, exp)` | `{ $pow: ["$base", exp] }` | Power operation | `POWER(base, 2) < 100` |
| `MOD(field, divisor)` | `{ $mod: ["$field", divisor] }` | Modulo operation | `price MOD 10 = 0` |
| `CEIL(field)` | `{ $ceil: "$field" }` | Ceiling function | `CEIL(rating) = 5` |
| `FLOOR(field)` | `{ $floor: "$field" }` | Floor function | `FLOOR(score) >= 80` |
| `ROUND(field, places)` | `{ $round: ["$field", places] }` | Round to places | `ROUND(average, 2)` |
| `TRUNC(field)` | `{ $trunc: "$field" }` | Truncate decimals | `TRUNC(price) = 100` |

**Arithmetic Operations**:
```data
# Mathematical expressions
FIND products WHERE price MOD 10 = 0
FIND metrics WHERE SQRT(variance) > threshold
FIND orders WHERE ROUND(total, 2) = 99.99

# Complex expressions with EXPR
FIND analytics
WHERE:
  - type = "performance"
  - EXPR "this.responseTime < this.averageTime * 1.5"
```

### String Mathematical Functions

| Function | MongoDB Equivalent | Description | Example |
|----------|-------------------|-------------|---------|
| `STRLEN(field)` | `{ $strLenCP: "$field" }` | String length | `STRLEN(name) > 5` |
| `SUBSTR(field, start, len)` | `{ $substr: ["$field", start, len] }` | Substring | `SUBSTR(email, 0, 5) = "admin"` |
| `CONCAT(str1, str2, ...)` | `{ $concat: ["$str1", "$str2"] }` | Concatenate | `CONCAT(firstName, " ", lastName)` |
| `UPPER(field)` | `{ $toUpper: "$field" }` | Uppercase | `UPPER(status) = "ACTIVE"` |
| `LOWER(field)` | `{ $toLower: "$field" }` | Lowercase | `LOWER(email) CONTAINS "@gmail.com"` |

### Array Mathematical Functions

| Function | MongoDB Equivalent | Description | Example |
|----------|-------------------|-------------|---------|
| `SIZE(field)` | `{ $size: "$field" }` | Array size | `SIZE(items) > 2` |
| `SUM(field)` | `{ $sum: "$field" }` | Sum array elements | `SUM(items.quantity) < 10` |
| `AVG(field)` | `{ $avg: "$field" }` | Average array elements | `AVG(items.price) > 50` |
| `MIN(field)` | `{ $min: "$field" }` | Minimum value | `MIN(scores) >= 60` |
| `MAX(field)` | `{ $max: "$field" }` | Maximum value | `MAX(ratings) <= 5` |

### Complex Expression Operations

```data
# Conditional expressions
UPDATE products
WHERE category = "electronics"
SET:
  - discountRate = COND(price > 1000, 0.1, 0.05)
  - finalPrice = MULTIPLY(price, SUBTRACT(1, discountRate))

# Complex aggregation with mathematical operations
AGGREGATE sales
PIPELINE:
  - MATCH created >= "{{$pastDate:30d}}"
  - ADD_FIELDS:
      - profit = SUBTRACT(revenue, cost)
      - margin = DIVIDE(profit, revenue)
      - roundedMargin = ROUND(margin, 2)
      - isHighMargin = COND(GT(margin, 0.3), true, false)
  - MATCH profit > 0
  - SORT margin DESC
```

## Performance Tuning

### Query Analysis with EXPLAIN

**Simple Explain**:
```data
EXPLAIN FIND users WHERE email = "test@example.com"
```

**Complex Explain with Options**:
```data
EXPLAIN
OPERATION:
  FIND products
  WHERE:
    - category = "electronics"
    - price BETWEEN 100 AND 500
    - inStock = true
OPTIONS:
  - Mode = "executionStats"
  - Verbose = true
EXPECT executionTimeMillis < 50
```

**Explain Modes**:
- `queryPlanner` - Query plan only
- `executionStats` - Execution statistics
- `allPlansExecution` - All considered plans

### Performance Profiling

**Enable Profiling**:
```data
PROFILE
LEVEL: 2
SLOW_MS: 100
OPERATIONS: ["find", "update", "aggregate"]
```

**Profile Specific Operation**:
```data
PROFILE OPERATION:
  AGGREGATE orders
  PIPELINE:
    - MATCH status = "completed"
    - GROUP BY customerId:
        - totalSpent = SUM(amount)
        - orderCount = COUNT(*)
EXPECT avgTimeMs < 200
```

**Disable Profiling**:
```data
PROFILE LEVEL: 0
```

**Profiling Levels**:
- `0` - Disabled
- `1` - Profile slow operations only
- `2` - Profile all operations

### Advanced Index Strategies

**TTL Index (Time To Live)**:
```data
CREATE_INDEX ON sessions
FIELDS:
  - expires ASC TTL
OPTIONS:
  - Name = "session_ttl_idx"
  - ExpireAfterSeconds = 3600
```

**Partial Index**:
```data
CREATE_INDEX ON users
FIELDS:
  - email ASC UNIQUE
OPTIONS:
  - Name = "active_user_email_idx"
  - PartialFilter:
      - isActive = true
      - isDeleted = false
```

**Sparse Index**:
```data
CREATE_INDEX ON users
FIELDS:
  - phoneNumber ASC SPARSE
OPTIONS:
  - Name = "phone_sparse_idx"
```

**Compound Index with Field Order**:
```data
CREATE_INDEX ON orders
FIELDS:
  - customerId ASC
  - status ASC  
  - created DESC
OPTIONS:
  - Name = "customer_order_lookup_idx"
  - Background = true
```

**Multikey Index for Arrays**:
```data
CREATE_INDEX ON products
FIELDS:
  - tags ASC MULTIKEY
OPTIONS:
  - Name = "product_tags_idx"
```

### Index Optimization

**Index Usage Hints**:
```data
FIND orders
WHERE customerId = "{{customerId}}"
USE_INDEX "customer_order_lookup_idx"
SORT created DESC
```

**Index Statistics**:
```data
INDEX_STATS ON orders
```

**Index Rebuild**:
```data
REINDEX COLLECTION orders
```

### Query Optimization

**Query with Hints and Timeouts**:
```data
FIND products
WHERE:
  - category = "electronics"
  - price > 100
USE_INDEX "category_price_idx"
COMMENT "Product search optimization test"
MAX_TIME_MS 5000
```

**Aggregation Optimization**:
```data
AGGREGATE orders
PIPELINE:
  - MATCH:
      - created >= "{{$pastDate:30d}}"
      - status = "completed"
  - GROUP BY customerId:
      - totalSpent = SUM(amount)
OPTIONS:
  - AllowDiskUse = true
  - MaxTimeMS = 10000
  - Comment = "Monthly customer analytics"
```

## Hybrid Raw MongoDB Syntax

For complex scenarios requiring native MongoDB syntax, use the `RAW` keyword:

### Raw Query Conditions

```data
# Complex query with raw MongoDB operators
FIND orders
WHERE:
  - status = "completed"
  - RAW: { "items": { "$elemMatch": { "price": { "$gt": 100 }, "category": "electronics" } } }
  - created >= "{{$pastDate:30d}}"
```

### Raw Update Operations

```data
# Raw update with complex array operations
UPDATE products
WHERE _id = "{{productId}}"
SET:
  - lastUpdated = "{{$now}}"
  - RAW: { 
      "$inc": { "viewCount": 1 },
      "$push": { 
        "viewHistory": { 
          "$each": [{ "timestamp": "{{$now}}", "user": "{{userId}}" }],
          "$slice": -100 
        }
      }
    }
```

### Raw Aggregation Stages

```data
# Hybrid aggregation with readable and raw syntax
AGGREGATE users
PIPELINE:
  - MATCH isActive = true
  - RAW: { 
      "$lookup": {
        "from": "orders",
        "let": { "userId": "$_id" },
        "pipeline": [
          { "$match": { "$expr": { "$eq": ["$customerId", "$$userId"] } } },
          { "$group": { "_id": null, "totalSpent": { "$sum": "$amount" } } }
        ],
        "as": "orderStats"
      }
    }
  - UNWIND orderStats
  - MATCH orderStats.totalSpent > 1000
```

### Raw Complex Expressions

```data
# Raw expressions for complex business logic
FIND analytics
WHERE:
  - type = "user_engagement"
  - RAW: { 
      "$expr": {
        "$and": [
          { "$gt": ["$pageViews", { "$multiply": ["$sessions", 3] }] },
          { "$lt": ["$bounceRate", 0.3] }
        ]
      }
    }
```

**Raw Syntax Rules**:
- Use `RAW:` followed by native MongoDB JSON syntax
- Can be mixed with readable syntax in same operation
- Maintains variable substitution within raw blocks
- Useful for advanced operations not yet supported

## AGGREGATE Operations

**Simple Pipeline**:
```data
AGGREGATE <collection>
PIPELINE:
  - MATCH <condition>
  - GROUP BY <field> COUNT(*) AS <alias>
  - SORT <field> <direction>
```

**Complex Pipeline**:
```data
AGGREGATE <collection>
PIPELINE:
  - MATCH:
      - <condition>
      - <condition>
  - GROUP BY <field>:
      - <alias> = COUNT(*)
      - <alias> = SUM(<field>)
      - <alias> = AVG(<field>)
  - SORT:
      - <field> <direction>
  - LIMIT <number>
```

**Pipeline Stages**:
- `MATCH` - Filter documents
- `GROUP BY` - Group and aggregate
- `SORT` - Sort results
- `LIMIT` - Limit results
- `SKIP` - Skip results
- `PROJECT` - Select fields
- `LOOKUP` - Join collections
- `UNWIND` - Deconstruct arrays
- `COUNT` - Count documents
- `ADD_FIELDS` - Add computed fields
- `FACET` - Multiple parallel pipelines
- `BUCKET` - Categorize documents

**Aggregate Functions**:
- `COUNT(*)` - Count documents
- `SUM(field)` - Sum field values
- `AVG(field)` - Average field values
- `MIN(field)` - Minimum field value
- `MAX(field)` - Maximum field value
- `FIRST(field)` - First value
- `LAST(field)` - Last value
- `PUSH(field)` - Collect values into array
- `ADD_TO_SET(field)` - Collect unique values

### INDEX Operations

**Simple Index**:
```data
CREATE_INDEX ON <collection> FIELDS <field> <direction> [<modifier>]
```

**Complex Index**:
```data
CREATE_INDEX ON <collection>
FIELDS:
  - <field> <direction>
  - <field> <direction>
[OPTIONS:
  - <option> = <value>
  - <option> = <value>]
```

**Index Directions**: `ASC`, `DESC`, `TEXT`, `GEO_2DSPHERE`, `GEO_2D`, `HASHED`, `TTL`, `SPARSE`, `MULTIKEY`

**Index Modifiers**: `UNIQUE`, `SPARSE`, `BACKGROUND`, `TTL`

**Index Options**:
- `Name = "index_name"`
- `Background = true`
- `Unique = true`
- `Sparse = true`
- `Language = "english"`
- `ExpireAfterSeconds = 3600` (TTL)
- `PartialFilter: { conditions }`

**Drop Index**:
```data
DROP_INDEX ON <collection> NAME "<index_name>"
```

### TRANSACTION Operations

**Basic Transaction**:
```data
BEGIN_TRANSACTION

<operation>
<operation>
<operation>

COMMIT_TRANSACTION
```

**Rollback Transaction**:
```data
BEGIN_TRANSACTION

<operation>
<operation>

ROLLBACK_TRANSACTION
```

**Rules**:
- All operations between BEGIN and COMMIT/ROLLBACK are atomic
- Nested transactions not supported
- Any operation failure causes automatic rollback

### CONDITIONAL Operations

**Simple Conditional**:
```data
IF_EXISTS <collection> WHERE <condition>
THEN <operation>
[ELSE <operation>]
```

**Multi-Line Conditional**:
```data
IF_EXISTS <collection>
WHERE <condition>
THEN:
  <operation>
  <operation>
[ELSE:
  <operation>
  <operation>]
```

**Count Conditional**:
```data
IF COUNT <collection> WHERE <condition> > <number>
THEN:
  <operation>
ELSE:
  <operation>
```

### SCHEMA Operations

**Simple Schema**:
```data
CREATE_COLLECTION <collection>
WITH_SCHEMA:
  Required: <field> (<type>), <field> (<type>)
```

**Detailed Schema**:
```data
CREATE_COLLECTION <collection>
WITH_SCHEMA:
  Required Fields:
    - <field> (<type>, <constraints>)
    - <field> (<type>, <constraints>)
  [Optional Fields:
    - <field> (<type>, <constraints>)
    - <field> (<type>, <constraints>)]
```

**Field Types**: `String`, `Integer`, `Number`, `Boolean`, `Array`, `Object`, `Date`

**Field Constraints**:
- `Range: <min>-<max>` - Numeric range
- `Format: Email` - Email format validation
- `MinLength: <n>` - Minimum string length
- `MaxLength: <n>` - Maximum string length

**Schema Validation**:
```data
VALIDATE_SCHEMA <collection> EXPECT <Valid|Invalid>
```

#### Schema Operations

##### CREATE_COLLECTION with DBSoup Schema

**Purpose**: Create MongoDB collections with readable, structured schema definitions inspired by DBSoup format.

**DBSoup-Inspired Syntax**:
```data
CREATE_COLLECTION collection_name
SCHEMA:
  * required_field  : DataType                [constraints]
  - optional_field  : DataType                [constraints]
  ! indexed_field   : DataType                [IX,constraints]
  @ sensitive_field : DataType                [ENCRYPTED,constraints]
  ~ masked_field    : DataType                [MASK:pattern,constraints]
  > partition_field : DataType                [PARTITION:method,constraints]
  $ audit_field     : DataType                [AUDIT:level,constraints]
```

**Field Prefixes** (inspired by DBSoup):
- `*` - Required field (cannot be null)
- `-` - Optional field (can be null)
- `!` - Indexed field (has database index)
- `@` - Sensitive/encrypted field (contains PII or encrypted data)
- `~` - Masked field (has data masking patterns)
- `>` - Partitioned field (partitioning or sharding keys)
- `$` - Audit field (audit trails and security logging)

**Data Types**:
**Primitive Types**:
- `String` - Text values
- `Number` - Numeric values (integers and decimals)  
- `Boolean` - `true` or `false`
- `Date` - Date/time objects
- `ObjectId` - MongoDB ObjectId (24-character hex)
- `GUID` - Microsoft GUID format (8-4-4-4-12 hex with hyphens, case-insensitive)
- `UUID` - RFC 4122 UUID format (8-4-4-4-12 hex with hyphens, lowercase)
- `null` - Null/undefined values

**Field Constraints**:
- `Range: <min>-<max>` - Numeric range
- `Format: Email` - Email format validation
- `MinLength: <n>` - Minimum string length
- `MaxLength: <n>` - Maximum string length

**Schema Validation**:
```data
VALIDATE_SCHEMA <collection> EXPECT <Valid|Invalid>
```

**Examples**:

```data
# Basic user collection
CREATE_COLLECTION users
SCHEMA:
  * _id           : String                    [PK,REQUIRED]
  * username      : String                    [REQUIRED,UNIQUE,MINLENGTH:3,MAXLENGTH:30]
  * email         : String                    [REQUIRED,UNIQUE,FORMAT:email]
  @ password      : String                    [ENCRYPTED,REQUIRED,MINLENGTH:8]
  - firstName     : String                    [MAXLENGTH:50]
  - lastName      : String                    [MAXLENGTH:50]
  - age           : Number                    [RANGE:13:120]
  - status        : String                    [ENUM:active,inactive,suspended,DEFAULT:active]
  ! createdAt     : Date                      [IX,SYSTEM,DEFAULT:CURRENT_TIMESTAMP]
  - updatedAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]

# E-commerce order collection with complex schema
CREATE_COLLECTION orders
SCHEMA:
  * _id           : String                    [PK,REQUIRED]
  * orderNumber   : String                    [REQUIRED,UNIQUE,PATTERN:^ORD-\d{8}$]
  * customerId    : String                    [REQUIRED,FK:customers._id]
  * status        : String                    [REQUIRED,ENUM:pending,processing,shipped,delivered,cancelled]
  * items         : Array<Object>             [REQUIRED,MIN_ITEMS:1,MAX_ITEMS:100]
  * total         : Number                    [REQUIRED,MIN:0]
  @ paymentInfo   : Object                    [ENCRYPTED,REQUIRED]
  ~ cardNumber    : String                    [MASK:****-****-****-####]
  > customerId    : String                    [PARTITION:hash,REQUIRED]
  $ orderEvents   : Array<Object>             [AUDIT:full,SYSTEM]
  - notes         : String                    [MAXLENGTH:500]
  ! customerEmail : String                    [IX:text,FORMAT:email]
  * createdAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]
  - updatedAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]
```

##### DEFINE_EMBEDDED_TYPE

**Purpose**: Define reusable embedded type schemas for complex nested objects.

**Syntax**:
```data
DEFINE_EMBEDDED_TYPE TypeName
SCHEMA:
  * field1        : DataType                  [constraints]
  - field2        : DataType                  [constraints]
```

**Examples**:
```data
DEFINE_EMBEDDED_TYPE Address
SCHEMA:
  * street        : String                    [REQUIRED,MAXLENGTH:100]
  * city          : String                    [REQUIRED,MAXLENGTH:50]
  * state         : String                    [REQUIRED,MAXLENGTH:50]
  * zipCode       : String                    [REQUIRED,PATTERN:^\d{5}(-\d{4})?$]
  * country       : String                    [REQUIRED,ENUM:US,CA,MX,DEFAULT:US]

DEFINE_EMBEDDED_TYPE OrderItem
SCHEMA:
  * productId     : String                    [REQUIRED,FK:products._id]
  * quantity      : Number                    [REQUIRED,MIN:1,MAX:999]
  * unitPrice     : Number                    [REQUIRED,MIN:0]
  * totalPrice    : Number                    [COMPUTED:quantity*unitPrice]
```

##### CREATE_COLLECTION with Raw JSON Schema

**Purpose**: For complex cases that require native MongoDB JSON Schema syntax.

**Syntax**:
```data
CREATE_COLLECTION collection_name
SCHEMA_JSON: {
  "type": "object",
  "properties": {
    "field": { "type": "string" }
  },
  "required": ["field"]
}
```

##### VALIDATE_SCHEMA

**Purpose**: Validate existing documents against collection schema.

**Syntax**:
```data
VALIDATE_SCHEMA collection_name
[WHERE conditions]
EXPECT_VALID operator threshold
```

**Examples**:
```data
# Validate all documents in collection
VALIDATE_SCHEMA users
EXPECT_VALID > 0.95

# Validate recent documents
VALIDATE_SCHEMA orders
WHERE createdAt >= "{{$pastDate:7d}}"
EXPECT_VALID = 1.0

# Validate with specific conditions
VALIDATE_SCHEMA products
WHERE status = "active"
EXPECT_VALID >= 0.99
```

### PERFORMANCE Operations

**Load Testing**:
```data
LOAD_TEST <collection> COUNT <number> TEMPLATE { <json_template> }
```

**Complex Load Test**:
```data
LOAD_TEST <collection>
COUNT <number>
TEMPLATE:
  - <field> = <template_value>
  - <field> = <template_value>
```

**Template Functions**:
- `RANDOM_DATE("<start>", "<end>")`
- `RANDOM_INT(<min>, <max>)`
- `RANDOM_CHOICE("<opt1>", "<opt2>", ...)`
- `{{$index}}` - Current iteration index

**Benchmarking**:
```data
BENCHMARK <operation> ITERATIONS <number> EXPECT_TIME < <time_spec>
```

**Time Specifications**: `<number>ms`, `<number>s`, `<number>m`

### ERROR Handling

**Expected Errors**:
```data
<operation>
EXPECT_ERROR "<error_message>"
```

**Try-Catch Blocks**:
```data
TRY:
  <operation>
  <operation>
CATCH:
  <operation>
  <operation>
```

**Advanced Error Handling**:
```data
TRY:
  <operation>
CATCH ValidationError:
  <operation>
CATCH DuplicateKeyError:
  <operation>
CATCH:
  <operation>
```

## Comments

The `.data` format supports both hash-style and double-slash comments for documentation and inline explanations.

### Comment Syntax

**Hash Comments (`#`)**:
- `# Full line comment` - Entire line is ignored
- `field: value  # Inline comment` - Everything after `#` is ignored

**Double-slash Comments (`//`)**:
- `// Full line comment` - Entire line is ignored  
- `field: value  // Inline comment` - Everything after `//` is ignored

### Comment Rules

1. **Characters after `#` or `//` are ignored** until end of line
2. **Both comment styles can be mixed** in the same file
3. **Comments can appear anywhere** - full lines or inline
4. **Comments are stripped during parsing** - no impact on execution
5. **Comments do not nest** - only single-line comments supported

### Examples

```data
# =============================================================================
# Database Setup Script
# =============================================================================

// Organization setup
INSERT organizations
{
  "_id": "org_001",                    # Primary key
  "name": "Test Company",              // Display name
  "domain": "test.com",                # Email domain
  "settings": {
    "maxUsers": 100,                   # User limit
    "features": ["basic", "reports"]   // Enabled features
  }
}

# User creation with mixed comment styles
INSERT users
{
  "_id": "user_001",                   // User identifier
  "orgId": "org_001",                  # Link to organization
  "profile": {
    "firstName": "John",               # Given name
    "lastName": "Doe",                 // Family name
    "email": "john@test.com"           # Contact email
  },
  "permissions": ["read", "write"],    // Access rights
  "isActive": true                     # Account status
}

// Query with inline documentation
FIND users
WHERE:
  - orgId = "org_001"                  # Filter by organization
  - isActive = true                    // Only active users
SELECT:
  - _id                                # User ID
  - profile.email                      # Email address
  - permissions                        // User permissions
LIMIT 10                               # Restrict results
```

## Date and Time Values

The `.data` format provides comprehensive support for date and time values through multiple approaches.

### Date/Time Input Formats

**Implicit String Formats** (automatically recognized):

| Format | Example | Description |
|--------|---------|-------------|
| ISO 8601 | `"2024-01-15T10:30:00Z"` | Standard UTC timestamp |
| ISO with timezone | `"2024-01-15T10:30:00-05:00"` | With timezone offset |
| Date only | `"2024-01-15"` | Date without time |
| Time only | `"10:30:00"` | Time without date |
| Alternative format | `"2024/01/15 10:30"` | Slash-separated |
| Natural language | `"January 15, 2024 10:30 AM"` | Human readable |
| RFC format | `"Mon, 15 Jan 2024 10:30:00 GMT"` | Email header format |
| Unix timestamp | `1705320600` | Seconds since epoch |
| Unix milliseconds | `1705320600000` | Milliseconds since epoch |

**Explicit Date Casting**:
```data
Date("2024-01-15T10:30:00Z")    # Force Date object
Date("2024-01-15")              # Date-only as Date object
Date("10:30:00")                # Time-only as Date object
Date(1705320600000)             # Unix milliseconds to Date
```

### Built-in Date Functions

**Current Date/Time Functions**:
- `{{$now}}` - Current ISO timestamp string
- `{{$timestamp}}` - Current Unix timestamp (seconds)

**Relative Date Functions**:
- `{{$futureDate:Nd}}` - N days in the future
- `{{$futureDate:Nh}}` - N hours in the future  
- `{{$futureDate:Nm}}` - N minutes in the future
- `{{$pastDate:Nd}}` - N days in the past
- `{{$pastDate:Nh}}` - N hours in the past
- `{{$pastDate:Nm}}` - N minutes in the past

**Date Formatting Functions**:
- `{{$dateFormat:date:format}}` - Format date with pattern

**Format Patterns**:
- `YYYY` - 4-digit year (2024)
- `MM` - 2-digit month (01-12)
- `DD` - 2-digit day (01-31)
- `HH` - 2-digit hour (00-23)
- `mm` - 2-digit minute (00-59)
- `ss` - 2-digit second (00-59)
- `MMM` - Short month name (Jan, Feb)
- `MMMM` - Full month name (January, February)

### Date Operations in Queries

**Comparison Operators**:
```data
# Date range queries
FIND orders
WHERE:
  - createdAt >= "2024-01-01T00:00:00Z"
  - createdAt < "2024-02-01T00:00:00Z"
  - updatedAt > "{{$pastDate:30d}}"
  - eventDate BETWEEN ["{{$pastDate:7d}}", "{{$now}}"]
```

**Date Math Functions**:
- `YEAR(date)` - Extract year from date
- `MONTH(date)` - Extract month (1-12)
- `DAY(date)` - Extract day of month (1-31)
- `HOUR(date)` - Extract hour (0-23)
- `MINUTE(date)` - Extract minute (0-59)
- `SECOND(date)` - Extract second (0-59)
- `DAY_OF_WEEK(date)` - Day of week (1=Sunday, 7=Saturday)
- `DAY_OF_YEAR(date)` - Day of year (1-366)
- `WEEK(date)` - Week of year (1-53)

**Date Arithmetic Functions**:
- `DATE_ADD(date, interval)` - Add time interval
- `DATE_SUB(date, interval)` - Subtract time interval
- `DATE_DIFF(date1, date2, unit)` - Difference between dates

**Interval Formats**:
- `"1y"` - 1 year
- `"3m"` - 3 months
- `"14d"` - 14 days
- `"6h"` - 6 hours
- `"30m"` - 30 minutes
- `"45s"` - 45 seconds

### Timezone Support

**Timezone-aware Timestamps**:
```data
INSERT events
{
  "_id": "event_001",
  "utcTime": "2024-01-15T15:30:00Z",           # UTC
  "localTime": "2024-01-15T10:30:00-05:00",    # EST
  "pacificTime": "2024-01-15T07:30:00-08:00"   # PST
}
```

**Timezone Functions**:
- `CONVERT_TIMEZONE(date, timezone)` - Convert to timezone
- `FORMAT_TIMEZONE(date, timezone)` - Format in timezone

### Examples

**Basic Date Operations**:
```data
# Insert with various date formats
INSERT events
{
  "_id": "event_001",
  "startTime": "2024-01-15T10:30:00Z",        # ISO string
  "endTime": Date("2024-01-15T14:30:00Z"),    # Date object
  "registrationDate": "{{$now}}",             # Current time
  "reminderDate": "{{$futureDate:1d}}",       # Tomorrow
  "createdAt": "{{$timestamp}}"               # Unix timestamp
}

# Query with date filters
FIND appointments
WHERE:
  - scheduledFor >= "{{$now}}"                # Future appointments
  - createdAt > "{{$pastDate:7d}}"            # Created in last week
  - YEAR(scheduledFor) = 2024                 # This year only
  - HOUR(scheduledFor) BETWEEN [9, 17]        # Business hours

# Update with date arithmetic
UPDATE subscriptions
WHERE _id = "sub_001"
SET:
  - renewalDate = DATE_ADD(startDate, "1y")   # Renew for 1 year
  - daysUntilExpiry = DATE_DIFF(expiryDate, "{{$now}}", "days")
  - lastBillingDate = "{{$pastDate:30d}}"     # 30 days ago
```

**Complex Date Processing**:
```data
# Aggregation with date grouping
AGGREGATE orders
PIPELINE:
  - MATCH createdAt >= "{{$pastDate:90d}}"
  - ADD_FIELDS:
      - orderYear = YEAR(createdAt)
      - orderMonth = MONTH(createdAt)
      - orderQuarter = CEIL(DIVIDE(MONTH(createdAt), 3))
      - isWeekend = IN(DAY_OF_WEEK(createdAt), [1, 7])
  - GROUP BY:
      - year = orderYear
      - quarter = orderQuarter
    AGGREGATE:
      - totalOrders = COUNT(*)
      - totalRevenue = SUM(total)
      - avgOrderValue = AVG(total)
  - SORT:
      - year ASC
      - quarter ASC
```

## Date vs String Detection

When no schema is defined, the `.data` format uses **content-based pattern recognition** to determine whether string values should be treated as Date objects or remain as Strings.

### Detection Strategy

**Default Behavior**: All string values remain as Strings unless:
1. Explicitly cast using `Date()` or `String()`
2. Built-in date functions are used (`{{$now}}`, `{{$futureDate:7d}}`)
3. String value starts with a digit and matches recognized date patterns

### Content-Based Date Detection

**Detection Algorithm**:
1. **Check if string starts with digit (0-9)**
2. **If YES**: Scan against recognized date patterns
3. **If pattern matches**: Convert to Date object
4. **If no match or doesn't start with digit**: Remain as String

**Recognized Date Patterns** (for strings starting with digits):

| Pattern | Regex | Example | Priority |
|---------|-------|---------|----------|
| **ISO 8601 UTC** | `/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z$/` | `"2024-01-15T10:30:00Z"` | 1 |
| **ISO with timezone** | `/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}[+-]\d{2}:\d{2}$/` | `"2024-01-15T10:30:00-05:00"` | 2 |
| **ISO date only** | `/^\d{4}-\d{2}-\d{2}$/` | `"2024-01-15"` | 3 |
| **Slash date** | `/^\d{4}\/\d{1,2}\/\d{1,2}$/` | `"2024/1/15"` | 4 |
| **US date format** | `/^\d{1,2}\/\d{1,2}\/\d{4}$/` | `"1/15/2024"` | 5 |
| **European date** | `/^\d{1,2}[\/\.]\d{1,2}[\/\.]\d{4}$/` | `"15/01/2024"` | 6 |
| **Unix seconds** | `/^\d{10}$/` | `"1705320600"` | 7 |
| **Unix milliseconds** | `/^\d{13}$/` | `"1705320600000"` | 8 |
| **Natural language** | `/^\d{1,2}\s+(January|February|...|December)\s+\d{4}$/i` | `"15 January 2024"` | 9 |
| **Time only** | `/^\d{1,2}:\d{2}(:\d{2})?$/` | `"10:30:00"` | 10 |
| **Year only** | `/^\d{4}$/` | `"2024"` | 11 |

### Type Resolution Priority

**Priority Order** (highest to lowest):

1. **Explicit Type Casting** - `Date()`, `String()`, `Number()`, etc.
2. **Built-in Date Functions** - `{{$now}}`, `{{$futureDate:7d}}`, etc.
3. **Content-Based Pattern Detection** - Strings starting with digits, matching date patterns
4. **Default String Behavior** - All other strings remain as Strings

### Examples

**Content-Based Detection (Automatic)**:
```data
INSERT auto_detection
{
  "_id": "example_001",
  
  # Strings starting with digits → scanned for date patterns → detected as dates:
  "eventDate": "2024-01-15T10:30:00Z",          # → Date (ISO 8601 detected)
  "scheduledFor": "2024-01-15",                 # → Date (ISO date detected)
  "meetingTime": "01/15/2024",                  # → Date (US format detected)
  "deadline": "15/01/2024",                     # → Date (European detected)
  "timestamp": "1705320600000",                 # → Date (Unix millis detected)
  "startTime": "09:30:00",                      # → Date (time detected)
  "year": "2024",                              # → Date (year detected)
  
  # Strings starting with digits → scanned → no date match → remain strings:
  "phoneNumber": "555-123-4567",                # → String (not date pattern)
  "productCode": "12345-ABC",                   # → String (not date pattern)
  "version": "2024.1.5",                        # → String (not date pattern)
  "quantity": "100",                            # → String (numeric but not date)
  
  # Strings not starting with digits → not scanned → remain strings:
  "title": "Meeting on 2024-01-15",             # → String (starts with 'M')
  "description": "Due January 15th",            # → String (starts with 'D')
  "status": "active"                            # → String (starts with 'a')
}
```

**Explicit Control (Highest Priority)**:
```data
INSERT explicit_control
{
  "_id": "example_002",
  "eventDate": String("2024-01-15T10:30:00Z"),  # Force String (override detection)
  "buildDate": Date("January 15, 2024"),        # Force Date (starts with letter)
  "versionDate": String("2024-01-15")           # Force String (override detection)
}
```

**Built-in Functions (High Priority)**:
```data
INSERT dynamic_dates
{
  "_id": "example_003", 
  "timestamp": "{{$now}}",                      # Always Date (function)
  "futureDate": "{{$futureDate:7d}}",           # Always Date (function)
  "unixTime": "{{$timestamp}}"                  # Always Number (function)
}
```

### Query Implications

**Date Objects** support date operations:
```data
# These work with Date objects (auto-detected from content):
FIND events
WHERE:
  - eventDate >= "2024-01-01T00:00:00Z"              # Date comparison
  - YEAR(scheduledFor) = 2024                        # Date function
  - meetingTime BETWEEN ["01/01/2024", "12/31/2024"] # Date range
```

**String Values** require string operations:
```data
# These work with String values (not detected as dates):
FIND events  
WHERE:
  - phoneNumber MATCHES "555-.*"                     # Regex pattern
  - productCode STARTS_WITH "12345"                 # String function
  - LENGTH(version) > 5                             # String function
```

### Edge Cases

**Ambiguous Patterns**:
```data
INSERT edge_cases
{
  "_id": "edge_001",
  "possibleDate": "01/02/2024",     # → Date (detected as US format: Jan 2, 2024)
  "possibleEuro": "02/01/2024",     # → Date (detected as US format: Feb 1, 2024)
  "serialNumber": "20240115001",    # → String (too long for Unix timestamp)
  "coordinates": "40.7128,-74.0060" # → String (not a date pattern)
}
```

**Override Detection**:
```data
INSERT overrides
{
  "_id": "override_001",
  "buildNumber": String("20240115"),     # Force String (would detect as date)
  "naturalDate": Date("January 15th"),   # Force Date (starts with letter)
  "timestamp": String("1705320600")      # Force String (would detect as Unix)
}
```

### Best Practices

1. **Rely on automatic detection** - content-based detection works for most cases
2. **Use explicit casting sparingly** - only when detection is wrong
3. **Use consistent date formats** - ISO 8601 recommended for clarity
4. **Test edge cases** - verify detection works as expected for your data
5. **Use built-in functions** for dynamic dates to ensure Date objects

**Recommended Approach**:
```data
INSERT recommended
{
  "_id": "rec_001",
  "eventDate": "2024-01-15T10:30:00Z",    # Auto-detected as Date
  "createdAt": "{{$now}}",                # Function ensures Date
  "title": "Meeting Title",               # Natural String
  "buildId": String("20240115")           # Force String when needed
}
```

## System Directives

The `.data` format includes predefined system directives that control format behavior. These are **built-in keywords**, not user-defined variables.

### @COLLECTION_ID_TYPE (Predefined)

**Purpose**: Configure the default `_id` field type for collections without explicit schemas.

**Type**: System directive (predefined keyword)

**Syntax**:
```data
@COLLECTION_ID_TYPE collection_name = TypeName
```

**Supported Types**:
- `ObjectId` - MongoDB ObjectId type
- `UUID` - Standard UUID format  
- `String` - String type (default behavior)
- `Number` - Numeric type

**Behavior**:
- **Global Scope**: Applies to ALL operations on the specified collection within the file
- **Persistent**: Remains active until file end or overridden
- **Automatic Conversion**: Converts simple string values to the specified type
- **Override**: Can be overridden by explicit type casting functions

**Examples**:
```data
# System directive configuration
@COLLECTION_ID_TYPE products = ObjectId
@COLLECTION_ID_TYPE users = String
@COLLECTION_ID_TYPE orders = UUID

# Automatic type conversion based on directives
INSERT products { "_id": "507f1f77bcf86cd799439020" }  # → ObjectId automatically
INSERT users { "_id": "user_123" }                     # → String (explicit directive)
INSERT orders { "_id": "550e8400-e29b-41d4" }          # → UUID automatically

# Override with explicit casting (takes precedence over directive)
INSERT products { "_id": String("custom_string_id") }  # → String (overrides ObjectId directive)
```

**Relationship to User Variables**:
```data
# System directive (predefined)
@COLLECTION_ID_TYPE products = ObjectId

# User variable (your definition)
@myProductId = "507f1f77bcf86cd799439020"

# Usage - system directive converts the user variable value
INSERT products { "_id": "{{myProductId}}" }  # → ObjectId("507f1f77bcf86cd799439020")
```

## Type Casting and ID Specification

When working with collections without explicit schemas, the format uses **implicit type casting** by default, with explicit casting available when needed.

### Implicit Type Casting (Default Behavior)

**Purpose**: Automatically infer types from literal values without requiring casting functions.

**Rules**:
- String literals (`"text"`) → String type
- Numeric literals (`42`, `99.99`) → Number type  
- Boolean literals (`true`, `false`) → Boolean type
- Array literals (`[1, 2, 3]`) → Array type
- Object literals (`{"key": "value"}`) → Object type

**Examples**:
```data
# No explicit casting needed - types inferred automatically
INSERT products
{
  "_id": "507f1f77bcf86cd799439020",  # String (or ObjectId via smart detection)
  "name": "Wireless Headphones",      # String
  "price": 99.99,                     # Number
  "inStock": true,                    # Boolean
  "quantity": 42,                     # Number
  "tags": ["electronics", "audio"],   # Array<String>
  "metadata": {"brand": "TechCorp"}    # Object
}
```

### Explicit Type Casting Functions

**Purpose**: Override natural types or specify special MongoDB types.

**When to Use**:
- Force ObjectId, UUID, or Date types
- Override natural type inference
- Ensure specific type when ambiguous

**Available Functions**:

| Function | Purpose | Example | Natural vs Override |
|----------|---------|---------|-------------------|
| `ObjectId("hex")` | MongoDB ObjectId | `ObjectId("507f1f77bcf86cd799439020")` | Override String → ObjectId |
| `UUID("uuid")` | Standard UUID | `UUID("550e8400-e29b-41d4-a716-446655440000")` | Override String → UUID |
| `Date("iso")` | Date casting | `Date("2024-01-15T10:30:00Z")` | Override String → Date |
| `String("value")` | Force String | `String("42")` | Override Number → String |
| `Number("value")` | Force Number | `Number("42.5")` | Override String → Number |

**Examples**:
```data
# Explicit casting when natural inference isn't desired
INSERT mixed_types
{
  "_id": ObjectId("507f1f77bcf86cd799439020"),  # Force ObjectId (not String)
  "numericCode": String("001234"),              # Force "001234" as String (not Number)
  "priceString": String("99.99"),               # Force "99.99" as String (not Number)
  "numericString": Number("42"),                # Force "42" as Number (not String)
  "timestamp": Date("2024-01-15T10:30:00Z")     # Force Date (not String)
}
```

### Type Resolution Priority

The format resolves types in this order:

1. **Explicit Type Casting** (highest priority)
   ```data
   "field": ObjectId("507f1f77bcf86cd799439020")  # Explicit override
   ```

2. **Collection Type Hints** (for _id fields)
   ```data
   @COLLECTION_ID_TYPE products = ObjectId
   "_id": "507f1f77bcf86cd799439020"  # Uses collection hint
   ```

3. **Smart Detection** (for _id fields only)
   ```data
   "_id": "507f1f77bcf86cd799439020"  # 24-char hex → ObjectId
   ```

4. **Implicit Type Inference** (lowest priority, natural types)
   ```data
   "field": "507f1f77bcf86cd799439020"  # String literal → String
   "count": 42                          # Number literal → Number
   ```

### Simplified Examples

**Most Common Case (No Casting Needed)**:
```data
# Natural type inference - no functions required
INSERT users
{
  "_id": "user_12345",        # String (natural)
  "age": 25,                  # Number (natural)  
  "isActive": true,           # Boolean (natural)
  "scores": [85, 92, 78]      # Array<Number> (natural)
}
```

**Special Types Only**:
```data
# Only use casting for ObjectId, UUID, Date
INSERT orders
{
  "_id": ObjectId("507f1f77bcf86cd799439020"),  # Special type
  "customerId": "user_12345",                   # String (natural)
  "total": 149.99,                             # Number (natural)
  "orderDate": Date("2024-01-15T10:30:00Z")    # Special type
}
```

### Priority Order

The format applies type specification in this priority order:

1. **Explicit Type Casting** (highest priority)
   ```data
   "_id": ObjectId("507f1f77bcf86cd799439020")
   ```

2. **Collection Type Hints**
   ```data
   @COLLECTION_ID_TYPE products = ObjectId
   "_id": "507f1f77bcf86cd799439020"  # Uses collection hint
   ```

3. **Smart Type Detection** (lowest priority)
   ```data
   "_id": "507f1f77bcf86cd799439020"  # Pattern-based detection
   ```

### Mixed Operations Example

```data
# Set collection defaults
@COLLECTION_ID_TYPE products = ObjectId
@COLLECTION_ID_TYPE users = String

# Insert with collection hints
INSERT products { "_id": "507f1f77bcf86cd799439020", "name": "Product 1" }
INSERT users { "_id": "user_123", "name": "John" }

# Override with explicit casting
INSERT products { "_id": String("custom_product_id"), "name": "Special Product" }

# Query with appropriate types
FIND products WHERE _id = "507f1f77bcf86cd799439020"  # ObjectId (collection hint)
FIND products WHERE _id = String("custom_product_id")  # String (explicit cast)
FIND users WHERE _id = "user_123"  # String (collection hint)
```

## Backup, Restore & Temporary Data Management

### Automatic Change Tracking

**Purpose**: Automatically track every change made by the DotData file for precise rollback.

**System Variables**:
- `@TRACK_CHANGES = true/false` - Enable/disable automatic change tracking (default: true)
- `@ROLLBACK_ON_ERROR = true/false` - Automatically rollback changes on error (default: false)  
- `@CHANGE_TAG = "string"` - Tag changes for selective rollback

**Change Tracking Behavior**:
- **INSERT**: Document is tracked for deletion on rollback
- **UPDATE**: Original field values are automatically backed up before change
- **DELETE**: Complete document is backed up before deletion
- **UPSERT**: Tracks whether operation was insert or update for proper rollback

**Examples**:
```data
# Enable change tracking (default)
@TRACK_CHANGES = true

# Insert tracked for deletion
INSERT users {"_id": "new_user", "name": "Test User"}

# Update tracked with original values backup
UPDATE users WHERE _id = "existing_user" SET email = "new@test.com"

# Delete tracked with full document backup  
DELETE users WHERE _id = "temp_user"
```

### Granular Rollback Operations

**Purpose**: Rollback specific changes made by current DotData file execution.

**Syntax**:
```data
ROLLBACK_CHANGES [WHERE conditions]
ROLLBACK_INSERTS [WHERE conditions]
ROLLBACK_UPDATES [WHERE conditions] 
ROLLBACK_DELETES [WHERE conditions]
ROLLBACK_DOCUMENT <collection> <id_field> <id_value>
ROLLBACK_OPERATION <operation_index>
```

**Examples**:
```data
# Rollback all tracked changes
ROLLBACK_CHANGES

# Rollback only insertions (delete inserted documents)
ROLLBACK_INSERTS

# Rollback only updates (restore original values)
ROLLBACK_UPDATES

# Rollback only deletions (re-insert deleted documents)
ROLLBACK_DELETES

# Rollback specific document
ROLLBACK_DOCUMENT users "_id" "new_user_001"

# Rollback by operation order
ROLLBACK_OPERATION 1        # Rollback first operation
ROLLBACK_OPERATION 3        # Rollback third operation

# Rollback with conditions
ROLLBACK_CHANGES WHERE tag = "user_test"
```

**Rollback Behavior**:
- **Rollback INSERT**: Executes `DELETE` for the inserted document
- **Rollback UPDATE**: Executes `UPDATE` with original backed-up values
- **Rollback DELETE**: Executes `INSERT` with the backed-up document
- **Rollback UPSERT**: Either `DELETE` (was insert) or `UPDATE` (was update)

### Change History Operations

**Purpose**: View and verify tracked changes before rollback.

**Syntax**:
```data
SHOW_CHANGES [collection] [WHERE conditions]
VERIFY_ROLLBACK [WHERE conditions]
EXPORT_CHANGES TO "<filename>"
IMPORT_CHANGES FROM "<filename>"
```

**Examples**:
```data
# Show all tracked changes
SHOW_CHANGES

# Show changes for specific collection
SHOW_CHANGES users

# Show changes with conditions
SHOW_CHANGES users WHERE _id = "test_user_001"

# Verify rollback is possible
VERIFY_ROLLBACK

# Export change history
EXPORT_CHANGES TO "test_changes.json"

# Import and replay changes
IMPORT_CHANGES FROM "test_changes.json"
REPLAY_CHANGES
```

**Change History Format**:
```json
{
  "operationIndex": 1,
  "type": "INSERT|UPDATE|DELETE|UPSERT",
  "collection": "users",
  "document": {...},
  "originalValues": {...},  // For UPDATE operations
  "timestamp": "2024-01-01T10:00:00Z",
  "tag": "user_test"
}
```

### Collection-Level Backup Operations

**Purpose**: Traditional backup operations for collection-level safety.

**Syntax**:
```data
BACKUP <collection> [WHERE conditions] TO "<backup_name>"
BACKUP_COLLECTIONS [collection_list] TO "<backup_name>"
```

**Examples**:
```data
# Backup entire collection
BACKUP users TO "users_backup_001"

# Backup with conditions
BACKUP orders 
WHERE status = "active" AND createdAt >= "{{$pastDate:30d}}"
TO "active_orders_backup"

# Backup multiple collections
BACKUP_COLLECTIONS ["users", "orders", "products"] TO "api_test_backup"
```

**Backup Options**:
- `COMPRESSION` - Enable backup compression
- `ENCRYPTION` - Encrypt backup data
- `INCREMENTAL FROM <previous_backup>` - Only backup changes since previous backup

### Collection-Level Restore Operations

**Purpose**: Restore previously backed up collections.

**Syntax**:
```data
RESTORE <collection> FROM "<backup_name>" [MODE <mode>]
RESTORE_COLLECTIONS FROM "<backup_name>"
```

**Restore Modes**:
- `replace` - Completely replace collection with backup (default)
- `merge` - Merge backup data with existing data
- `skip` - Skip conflicts, keep existing data
- `error` - Throw error on conflicts

**Examples**:
```data
# Simple restore (replace mode)
RESTORE users FROM "users_backup_001"

# Restore with merge mode
RESTORE products FROM "products_backup" MODE merge

# Restore multiple collections
RESTORE_COLLECTIONS FROM "api_test_backup"

# Conditional restore
RESTORE users FROM "users_backup_001"
WHERE _id IN ["user_001", "user_002"]
FIELDS: ["email", "profile", "settings"]
```

### Snapshot Operations

**Purpose**: Point-in-time snapshots of entire database state.

**Syntax**:
```data
CREATE_SNAPSHOT "<snapshot_name>" [INCLUDE_COLLECTIONS [collection_list]]
RESTORE_SNAPSHOT "<snapshot_name>"
LIST_SNAPSHOTS [WHERE conditions]
DELETE_SNAPSHOT "<snapshot_name>"
```

**Examples**:
```data
# Create full database snapshot
CREATE_SNAPSHOT "before_api_test_suite"

# Create partial snapshot
CREATE_SNAPSHOT "user_data_snapshot"  
INCLUDE_COLLECTIONS ["users", "profiles", "settings"]
METADATA:
  - description = "Before user management tests"
  - testSuite = "user_management"
  - created = "{{$now}}"

# Restore snapshot (complete rollback)
RESTORE_SNAPSHOT "before_api_test_suite"

# List snapshots
LIST_SNAPSHOTS WHERE created >= "{{$pastDate:7d}}"
```

### Temporary Data Management

**Purpose**: Mark data as temporary with automatic cleanup capabilities.

**Temporary Data Markers**:
- `_temp: true` - Mark as temporary data
- `_ttl: <timestamp>` - Time-to-live (auto-expire)
- `_testData: "<test_id>"` - Test suite identifier
- `_cleanup: true` - Include in cleanup operations

**Examples**:
```data
# Insert temporary test data
INSERT users
{
  "_id": "temp_user_001",
  "name": "Test User",
  "_temp": true,                    # Temporary marker
  "_ttl": "{{$futureDate:1h}}",    # Expires in 1 hour
  "_testData": "api_test_001"       # Test identifier
}

# Bulk mark as temporary
UPDATE users 
WHERE _id STARTS_WITH "test_"
SET:
  - _temp = true
  - _ttl = "{{$futureDate:30m}}"
  - _testData = "user_registration_test"
```

### Cleanup Operations

**Purpose**: Remove temporary test data and restore clean state.

**Syntax**:
```data
CLEANUP_TEMP_DATA [WHERE conditions]
CLEANUP_EXPIRED_DATA [WHERE conditions]  
CLEANUP_COLLECTIONS [collection_list] WHERE conditions
```

**Examples**:
```data
# Clean up all temporary data
CLEANUP_TEMP_DATA WHERE _temp = true

# Clean up by test identifier
CLEANUP_TEMP_DATA WHERE _testData = "api_test_001"

# Clean up expired data
CLEANUP_EXPIRED_DATA WHERE _ttl < "{{$now}}"

# Clean up specific collections
CLEANUP_COLLECTIONS ["users", "orders"] WHERE _temp = true

# Pattern-based cleanup
DELETE * WHERE _id MATCHES "^temp_.*"
```

### Error Handling with Automatic Rollback

**Purpose**: Automatically rollback changes when errors occur.

**Error Handling**:
```data
TRY:
  @ROLLBACK_ON_ERROR = true
  
  INSERT users {"_id": "user_001", "email": "test@test.com"}
  UPDATE users WHERE _id = "existing" SET status = "modified"
  
  # This error triggers automatic rollback
  INSERT users {"_id": "user_001", "email": "duplicate@test.com"}  # Duplicate key
  
CATCH DuplicateKeyError:
  # All changes in TRY block automatically rolled back
  # Handle error with alternative approach
  
FINALLY:
  # Ensure clean state
  SHOW_CHANGES        # Should be empty due to rollback
```

### API Testing Workflow Pattern

**Standard API Testing Pattern**:
```data
# 1. Enable change tracking
@TRACK_CHANGES = true
@ROLLBACK_ON_ERROR = true
@CHANGE_TAG = "{{testName}}"

# 2. Make tracked changes
INSERT test_collection { "_id": "test_001", ... }
UPDATE existing_collection WHERE _id = "existing_001" SET field = "new_value"

# 3. Run API tests (external)
# Your API testing framework executes here

# 4. Precise rollback
TRY:
  ROLLBACK_CHANGES WHERE tag = "{{testName}}"
CATCH:
  # Fallback to snapshot restore if needed
  RESTORE_SNAPSHOT "emergency_backup"

# 5. Verify clean state
VERIFY_ROLLBACK
SHOW_CHANGES        # Should be empty
```

### Performance Considerations

**Change Tracking Performance**:
- Minimal overhead for INSERT operations (only track document ID)
- Moderate overhead for UPDATE operations (backup original values)
- Higher overhead for DELETE operations (backup entire document)
- Use `@TRACK_CHANGES = false` for bulk operations when rollback not needed

**Rollback Performance**:
- Fast rollback for INSERT operations (simple DELETE)
- Moderate rollback for UPDATE operations (restore original values)
- Slower rollback for DELETE operations (re-insert entire document)
- Use operation-specific rollback (`ROLLBACK_INSERTS`) when possible

## System Variables

### Overview

System variables are predefined keywords that control the behavior of the DotData file execution. They are not user-defined variables but built-in system configuration options.

### Collection ID Type Configuration

#### @COLLECTION_ID_TYPE (Predefined)

**Purpose**: Set the default `_id` field type for collections.

**Syntax**:
```data
@COLLECTION_ID_TYPE collection_name = TypeName
```

**Supported Types**:
- `ObjectId` - Standard MongoDB ObjectId
- `UUID` - UUID format  
- `String` - String type (default)
- `Number` - Numeric type (with AUTO_INCREMENT support)

**Examples**:
```data
@COLLECTION_ID_TYPE products = ObjectId
@COLLECTION_ID_TYPE users = String
@COLLECTION_ID_TYPE orders = UUID
@COLLECTION_ID_TYPE categories = Number
```

**Behavior**: Once set, applies to all subsequent operations on that collection within the file.

**Type Detection**: When no @COLLECTION_ID_TYPE is specified, the system uses smart type detection based on the `_id` value format.

### Change Tracking System Variables

#### @TRACK_CHANGES (Predefined)

**Purpose**: Enable/disable automatic change tracking for rollback operations.

**Syntax**:
```data
@TRACK_CHANGES = true|false
```

**Default**: `true`

**Behavior**:
- `true` - All INSERT, UPDATE, DELETE operations are tracked for rollback
- `false` - Operations are not tracked (better performance for bulk operations)

**Change Tracking Details**:
- **INSERT**: Document ID tracked for deletion on rollback
- **UPDATE**: Original field values backed up before modification
- **DELETE**: Complete document backed up before deletion
- **UPSERT**: Tracks whether result was insert or update for proper rollback

**Examples**:
```data
# Default behavior - tracking enabled
@TRACK_CHANGES = true
INSERT users {"_id": "user_001", "name": "John"}     # Tracked
UPDATE users WHERE _id = "user_002" SET name = "Jane"  # Original name backed up

# Disable for bulk operations
@TRACK_CHANGES = false
INSERT_MANY bulk_users [...]                         # Not tracked

# Re-enable tracking
@TRACK_CHANGES = true
INSERT orders {"_id": "order_001", "total": 99.99}   # Tracked again
```

#### @ROLLBACK_ON_ERROR (Predefined)

**Purpose**: Automatically rollback tracked changes when errors occur.

**Syntax**:
```data
@ROLLBACK_ON_ERROR = true|false
```

**Default**: `false`

**Behavior**:
- `true` - Any error triggers automatic rollback of all tracked changes in current execution context
- `false` - Errors do not trigger automatic rollback (manual rollback required)

**Error Context**: Applies to TRY/CATCH blocks and general execution.

**Examples**:
```data
@ROLLBACK_ON_ERROR = true

INSERT users {"_id": "user_001", "email": "test@test.com"}
UPDATE users WHERE _id = "existing" SET status = "modified"

# This duplicate key error will automatically rollback both operations above
INSERT users {"_id": "user_001", "email": "duplicate@test.com"}

# Result: user_001 deleted, existing user restored to original status
```

#### @CHANGE_TAG (Predefined)

**Purpose**: Tag changes for selective rollback operations.

**Syntax**:
```data
@CHANGE_TAG = "string"
```

**Default**: `null` (no tag)

**Behavior**: All subsequent tracked changes are tagged with the specified string until changed.

**Use Cases**:
- Group changes by test suite
- Separate setup vs. test operations
- Enable partial rollback scenarios

**Examples**:
```data
# Tag setup operations
@CHANGE_TAG = "test_setup"
INSERT users {"_id": "setup_user", "name": "Setup User"}
INSERT roles {"_id": "admin_role", "name": "Administrator"}

# Tag test operations
@CHANGE_TAG = "api_test_001"
UPDATE users WHERE _id = "setup_user" SET role = "admin_role"
INSERT sessions {"_id": "test_session", "userId": "setup_user"}

# Clear tag
@CHANGE_TAG = null
INSERT logs {"_id": "log_001", "message": "Test completed"}  # Untagged

# Selective rollback
ROLLBACK_CHANGES WHERE tag = "api_test_001"     # Only rollback test operations
ROLLBACK_CHANGES WHERE tag = "test_setup"       # Then rollback setup
```

### System Variable Scope and Precedence

**File Scope**: System variables apply from declaration point until end of DotData file or redeclaration.

**Override Behavior**: Later declarations override earlier ones.

**Context Isolation**: TRY/CATCH blocks can have isolated variable contexts.

**Examples**:
```data
# Initial settings
@TRACK_CHANGES = true
@ROLLBACK_ON_ERROR = false
@CHANGE_TAG = "phase_1"

INSERT users {"_id": "user_001"}    # Tracked, tagged "phase_1"

# Change settings
@ROLLBACK_ON_ERROR = true
@CHANGE_TAG = "phase_2"

UPDATE users WHERE _id = "user_001" SET status = "active"  # Tracked, tagged "phase_2", auto-rollback enabled

# Temporary disable tracking
@TRACK_CHANGES = false
INSERT_MANY bulk_data [...]         # Not tracked

# Re-enable with new tag
@TRACK_CHANGES = true
@CHANGE_TAG = "cleanup_phase"
DELETE temp_data WHERE temp = true  # Tracked, tagged "cleanup_phase"
```

### Built-in Functions Integration

System variables work seamlessly with built-in functions:

```data
@CHANGE_TAG = "test_{{$timestamp}}"              # Dynamic tag with timestamp
@TRACK_CHANGES = "{{trackingEnabled}}"           # Variable-based setting

INSERT users {
  "_id": "{{$randomUUID}}",
  "createdAt": "{{$now}}",
  "testTag": "{{@CHANGE_TAG}}"                   # Reference current change tag
}
```

### Performance Considerations

**Change Tracking Overhead**:
- **INSERT**: Minimal (only stores document ID)
- **UPDATE**: Moderate (backs up original field values)
- **DELETE**: High (backs up entire document)

**Optimization Strategies**:
```data
# Disable tracking for known bulk operations
@TRACK_CHANGES = false
INSERT_MANY large_dataset [...]

# Use selective rollback instead of full tracking
@TRACK_CHANGES = true
@CHANGE_TAG = "critical_only"
INSERT critical_data {...}          # Only track critical operations
```

### Error Handling with System Variables

System variables integrate with error handling:

```data
TRY:
  @ROLLBACK_ON_ERROR = true
  @CHANGE_TAG = "transaction_001"
  
  INSERT users {...}
  UPDATE orders {...}
  DELETE temp_data {...}
  
CATCH DuplicateKeyError:
  # Changes automatically rolled back due to @ROLLBACK_ON_ERROR = true
  # Handle error recovery
  
CATCH ValidationError:
  # Manual rollback if needed
  ROLLBACK_CHANGES WHERE tag = "transaction_001"
  
FINALLY:
  # Reset variables
  @ROLLBACK_ON_ERROR = false
  @CHANGE_TAG = null
```

## GridFS Operations
