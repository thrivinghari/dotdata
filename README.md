# .data File Format - Complete Documentation

## 🎯 **Key Features**

- **Complete MongoDB Coverage**: 100% support for all MongoDB operations and features
- **DBSoup-Inspired Schema Syntax**: Readable, structured schema definitions with field prefixes
- **Flexible Syntax**: Both single-line and multi-line formats for optimal readability
- **Advanced Analytics**: Statistical functions, time-series analysis, and forecasting
- **Machine Learning Pipeline**: Complete ML workflow from preprocessing to prediction
- **Real-time Capabilities**: Change streams and live data monitoring
- **GridFS Support**: File storage and retrieval operations
- **Enterprise Features**: Replica sets, sharding, and high availability
- **Performance Optimization**: Query analysis, profiling, and optimization tools
- **Hybrid Raw Syntax**: Escape hatch for complex MongoDB operations
- **Built-in Functions**: Dynamic value generation and variable substitution
- **Error Handling**: Comprehensive try/catch and validation support

## 🆕 **DBSoup-Inspired Schema Syntax**

The `.data` format now includes readable, structured schema definitions inspired by the DBSoup format:

```data
# String-based ID (manual assignment)
CREATE_COLLECTION users
SCHEMA:
  * _id           : String                    [PK,REQUIRED]
  * username      : String                    [REQUIRED,UNIQUE,MINLENGTH:3]
  * email         : String                    [REQUIRED,UNIQUE,FORMAT:email]

# ObjectId-based ID (MongoDB default, auto-generated)
CREATE_COLLECTION products
SCHEMA:
  * _id           : ObjectId                  [PK,AUTO]
  * name          : String                    [REQUIRED,MINLENGTH:3]
  * price         : Number                    [REQUIRED,MIN:0]

# UUID-based ID (for distributed systems)
CREATE_COLLECTION orders
SCHEMA:
  * _id           : UUID                      [PK,AUTO]
  * orderNumber   : String                    [REQUIRED,UNIQUE]
  * customerId    : String                    [REQUIRED,FK:customers._id]

# Integer-based ID (for legacy compatibility)
CREATE_COLLECTION categories
SCHEMA:
  * _id           : Number                    [PK,AUTO_INCREMENT]
  * name          : String                    [REQUIRED,UNIQUE]
  * parentId      : Number                    [FK:categories._id]
```

## Primary Key Data Types

The `.data` format supports multiple data types for `_id` primary key fields:

### **Supported `_id` Types**

| Type | Example | Use Case |
|------|---------|----------|
| **String** | `"user_001"` | Simple, readable IDs |
| **ObjectId** | `"507f1f77bcf86cd799439020"` | MongoDB's default, 24-char hex |
| **GUID** | `"A1B2C3D4-E5F6-7890-ABCD-EF1234567890"` | Microsoft/Windows systems (most common) |
| **UUID** | `"a1b2c3d4-e5f6-7890-abcd-ef1234567890"` | RFC standard format |
| **Number** | `12345` | Sequential/auto-increment IDs |

### **Examples by Type**

```data
# String IDs (most flexible)
INSERT users { "_id": "user_001", "name": "John" }
INSERT users { "_id": "admin-user", "name": "Admin" }

# ObjectId (MongoDB standard)  
INSERT orders { "_id": "507f1f77bcf86cd799439020", "total": 99.99 }
INSERT orders { "_id": "507f191e810c19729de860ea", "total": 149.50 }

# GUID (Microsoft/Windows - very common)
INSERT sessions { "_id": "A1B2C3D4-E5F6-7890-ABCD-EF1234567890", "userId": "user_001" }
INSERT sessions { "_id": "F47AC10B-58CC-4372-A567-0E02B2C3D479", "userId": "user_002" }

# UUID (RFC standard)
INSERT tokens { "_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890", "expires": "{{$futureDate:1h}}" }
INSERT tokens { "_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479", "expires": "{{$futureDate:2h}}" }

# Number IDs (sequential)
INSERT products { "_id": 1001, "name": "Laptop" }
INSERT products { "_id": 1002, "name": "Mouse" }
```

## Easy GUID and ObjectId Syntax

### **Easy GUID Syntax (Most Common)**

You can use GUID strings directly without explicit casting - they're automatically detected and converted:

```data
# GUID strings are automatically converted to GUID objects for _id fields
INSERT users
{
  "_id": "A1B2C3D4-E5F6-7890-ABCD-EF1234567890",  # Auto-converts to GUID
  "name": "John Doe",
  "email": "john@example.com"
}

INSERT sessions  
{
  "_id": "F47AC10B-58CC-4372-A567-0E02B2C3D479",  # Auto-converts to GUID
  "userId": "A1B2C3D4-E5F6-7890-ABCD-EF1234567890", # References user above
  "createdAt": "{{$now}}"
}

# Generate random GUIDs dynamically
INSERT tokens
{
  "_id": "{{$randomGUID}}",                       # Generates new GUID
  "sessionId": "F47AC10B-58CC-4372-A567-0E02B2C3D479", # References session
  "expiresAt": "{{$futureDate:1h}}"
}
```

### **Easy ObjectId Syntax**

You can use 24-character hex strings directly without explicit casting - they're automatically detected and converted:

```data
# ObjectId strings are automatically converted to ObjectId objects for _id fields
INSERT products
{
  "_id": "507f1f77bcf86cd799439020",              # Auto-converts to ObjectId
  "name": "Wireless Headphones",
  "price": 99.99
}

INSERT orders
{
  "_id": "507f191e810c19729de860ea",              # Auto-converts to ObjectId  
  "productId": "507f1f77bcf86cd799439020",        # References product above
  "quantity": 2
}

# Generate random ObjectIds dynamically
INSERT reviews
{
  "_id": "{{$randomObjectId}}",                   # Generates new ObjectId
  "productId": "507f1f77bcf86cd799439020",        # References product
  "rating": 5,
  "createdAt": "{{$now}}"
}
```

**Format Recognition**:
- **GUID**: `8-4-4-4-12` hex digits with hyphens, typically uppercase
- **ObjectId**: Exactly 24 hex characters, no hyphens
- **UUID**: `8-4-4-4-12` hex digits with hyphens, typically lowercase

**Field Prefixes**:
- `*` - Required field (cannot be null)
- `-` - Optional field (can be null)
- `!` - Indexed field (has database index)
- `@` - Sensitive/encrypted field
- `~` - Masked field (data masking)
- `>` - Partitioned field (sharding key)
- `$` - Audit field (audit trail)

## Comments

The `.data` format supports inline and standalone comments using two syntaxes:

### Comment Syntax

**Hash Comments (`#`)**:
```data
# This is a full-line comment
INSERT users
{
  "_id": "user_001",           # Inline comment after field
  "name": "John Doe",          # Another inline comment
  "email": "john@example.com"  # Email field comment
}

# Multiple line comments
# can be written like this
# for longer explanations
```

**Double-slash Comments (`//`)**:
```data
// This is a full-line comment
INSERT products
{
  "_id": "product_001",         // Inline comment after field
  "name": "Laptop",             // Product name
  "price": 999.99               // Price in USD
}

// You can also mix comment styles
INSERT orders
{
  "_id": "order_001",           # Hash-style comment
  "productId": "product_001",   // Double-slash comment
  "quantity": 2                 # Both styles work
}
```

### Comment Rules

1. **Characters after `#` or `//` are ignored** until the end of the line
2. **Both syntaxes can be used interchangeably** in the same file
3. **Inline comments** can appear at the end of any line
4. **Full-line comments** can appear on their own lines
5. **Comments are ignored during execution** - they're for documentation only

### Comment Examples

```data
# =============================================================================
# User Management Data Setup
# =============================================================================

// Set up test organization
INSERT organizations
{
  "_id": "org_001",                    # Primary organization ID
  "name": "Test Corp",                 // Organization name
  "domain": "testcorp.com",            # Email domain for users
  "isActive": true,                    // Organization is active
  "createdAt": "{{$now}}"              # Current timestamp
}

// Create test users for the organization
INSERT_MANY users [
  {
    "_id": "user_001",                 # First test user
    "orgId": "org_001",               // Link to organization
    "email": "alice@testcorp.com",    # User email
    "role": "admin"                   // Administrator role
  },
  {
    "_id": "user_002",                 # Second test user  
    "orgId": "org_001",               // Same organization
    "email": "bob@testcorp.com",      # Different email
    "role": "user"                    // Regular user role
  }
]

# Query to verify the data was inserted correctly
FIND users
WHERE orgId = "org_001"              // Filter by organization
SELECT:
  - _id                              # User ID
  - email                            # Email address
  - role                             # User role
# Results should show both users
```

## Date and Time Values

The `.data` format provides multiple ways to work with dates and times, from natural string literals to explicit casting and built-in functions.

### Implicit Date/Time Strings (Natural)

Most date/time values can be specified as natural string literals:

```data
# ISO 8601 strings (recommended format)
INSERT events
{
  "_id": "event_001",
  "startTime": "2024-01-15T10:30:00Z",        # UTC timestamp
  "endTime": "2024-01-15T14:30:00-05:00",     # With timezone offset
  "publishDate": "2024-01-15",                # Date only
  "deadline": "2024-01-15T23:59:59.999Z"     # With milliseconds
}

# Human-readable formats (parsed automatically)
INSERT appointments
{
  "_id": "appt_001",
  "scheduledFor": "January 15, 2024 10:30 AM",  # Natural language
  "reminderAt": "2024/01/15 09:30",            # Alternative format
  "createdAt": "Mon, 15 Jan 2024 10:30:00 GMT" # RFC format
}
```

### Explicit Date Casting

Use `Date()` casting when you need to ensure proper date type or for clarity:

```data
# Explicit date casting for MongoDB Date objects
INSERT logs
{
  "_id": "log_001",
  "timestamp": Date("2024-01-15T10:30:00Z"),    # Force Date object
  "eventDate": Date("2024-01-15"),              # Date-only as Date object
  "expiresAt": Date("2024-12-31T23:59:59Z")     # Explicit future date
}

# Force date parsing for ambiguous strings
INSERT records
{
  "_id": "record_001",
  "dateField": Date("01/15/2024"),              # US format → Date object
  "timeField": Date("10:30:00"),                # Time-only → Date object
  "textDate": "01/15/2024"                      # Remains as String
}
```

### Built-in Date Functions

Use dynamic date functions for relative or generated dates:

```data
# Current date/time functions
INSERT sessions
{
  "_id": "{{$randomUUID}}",
  "createdAt": "{{$now}}",                      # Current ISO timestamp
  "timestamp": "{{$timestamp}}",                # Unix timestamp (number)
  "sessionId": "session_{{$timestamp}}"         # Embedded in string
}

# Relative date functions
INSERT notifications
{
  "_id": "notif_001",
  "scheduledFor": "{{$futureDate:7d}}",         # 7 days from now
  "validUntil": "{{$futureDate:30d}}",          # 30 days from now
  "lastReminderSent": "{{$pastDate:1d}}",       # 1 day ago
  "createdThisWeek": "{{$pastDate:7d}}"         # 1 week ago
}

# Date formatting functions
INSERT reports
{
  "_id": "report_001",
  "reportDate": "{{$dateFormat:{{$now}}:YYYY-MM-DD}}",           # 2024-01-15
  "fileName": "report_{{$dateFormat:{{$now}}:YYYY_MM_DD}}.pdf",  # report_2024_01_15.pdf
  "displayDate": "{{$dateFormat:{{$now}}:MMM DD, YYYY}}"        # Jan 15, 2024
}
```

### Time-only Values

For time-only fields, you can use various formats:

```data
# Time-only specifications
INSERT schedules
{
  "_id": "schedule_001",
  "startTime": "09:30:00",                      # HH:MM:SS format
  "endTime": "17:30",                           # HH:MM format
  "breakTime": "12:00:00.000",                  # With milliseconds
  "reminder": "08:45"                           # Simple format
}

# Time with explicit casting
INSERT business_hours
{
  "_id": "hours_001",
  "openTime": Date("09:00:00"),                 # Time as Date object
  "closeTime": Date("18:00:00"),                # Time as Date object
  "lunchStart": "12:00",                        # Time as String
  "lunchEnd": "13:00"                           # Time as String
}
```

### Unix Timestamps

Handle Unix timestamps (seconds/milliseconds since epoch):

```data
# Unix timestamp values
INSERT metrics
{
  "_id": "metric_001",
  "epochSeconds": 1705320600,                   # Unix timestamp (seconds)
  "epochMillis": 1705320600000,                 # Unix timestamp (milliseconds)
  "timestampString": "1705320600",              # Unix timestamp as string
  "convertedDate": Date(1705320600000)          # Convert milliseconds to Date
}

# Generate Unix timestamps
INSERT analytics
{
  "_id": "analytics_001",
  "recordedAt": "{{$timestamp}}",               # Current Unix timestamp
  "isoTime": "{{$now}}",                       # Current ISO string
  "dayStartEpoch": "{{$timestamp:startOfDay}}", # Start of current day
  "weekStartEpoch": "{{$timestamp:startOfWeek}}" # Start of current week
}
```

### Date Comparisons and Queries

Use dates in query operations:

```data
# Query with date ranges
FIND orders
WHERE:
  - createdAt >= "2024-01-01T00:00:00Z"        # String comparison
  - createdAt < Date("2024-02-01T00:00:00Z")   # Date object comparison
  - updatedAt > "{{$pastDate:30d}}"            # Relative date

# Query with date functions
FIND events
WHERE:
  - eventDate >= "{{$now}}"                    # Future events
  - createdAt BETWEEN ["{{$pastDate:7d}}", "{{$now}}"] # Last week
  - YEAR(eventDate) = 2024                     # Specific year
  - MONTH(eventDate) IN [1, 2, 3]             # Q1 events
```

### Date Arithmetic and Manipulation

Perform date calculations:

```data
# Date math in updates
UPDATE subscriptions
WHERE _id = "sub_001"
SET:
  - renewalDate = DATE_ADD(startDate, "1y")    # Add 1 year
  - trialEndDate = DATE_ADD(createdAt, "14d")  # Add 14 days
  - lastBillingDate = DATE_SUB("{{$now}}", "1m") # Subtract 1 month

# Complex date calculations
UPDATE user_stats
WHERE lastLogin < "{{$pastDate:90d}}"
SET:
  - inactiveDays = DATE_DIFF("{{$now}}", lastLogin, "days")
  - status = "inactive"
  - suspensionDate = DATE_ADD("{{$now}}", "7d")
```

### Timezone Handling

Work with different timezones:

```data
# Timezone-aware dates
INSERT global_events
{
  "_id": "event_global_001",
  "utcTime": "2024-01-15T15:30:00Z",           # UTC
  "eastCoastTime": "2024-01-15T10:30:00-05:00", # EST
  "westCoastTime": "2024-01-15T07:30:00-08:00", # PST
  "londonTime": "2024-01-15T15:30:00+00:00",   # GMT
  "tokyoTime": "2024-01-16T00:30:00+09:00"     # JST
}

# Convert between timezones
UPDATE meetings
WHERE _id = "meeting_001"
SET:
  - localTime = CONVERT_TIMEZONE(utcTime, "America/New_York")
  - displayTime = FORMAT_TIMEZONE(utcTime, "user_timezone")
```

### Special Date Values

Handle special date cases:

```data
# Special date values
INSERT special_dates
{
  "_id": "special_001",
  "nullDate": null,                            # No date set
  "minDate": Date("1970-01-01T00:00:00Z"),     # Unix epoch start
  "maxDate": Date("2038-01-19T03:14:07Z"),     # 32-bit timestamp limit
  "indefinite": "never",                       # String indicator
  "tbd": "TBD"                                # To be determined
}

# Date validation
INSERT user_profiles
{
  "_id": "user_001",
  "birthDate": Date("1990-01-15"),             # Must be Date object
  "registrationDate": "{{$now}}",              # Auto-generated
  "lastLoginDate": null,                       # Initially null
  "accountExpiryDate": "{{$futureDate:365d}}"  # 1 year from now
}
```

### Date Formats Summary

| Format | Example | Type | Use Case |
|--------|---------|------|----------|
| ISO 8601 | `"2024-01-15T10:30:00Z"` | String/Date | Standard format |
| Date only | `"2024-01-15"` | String/Date | Date without time |
| Time only | `"10:30:00"` | String/Date | Time without date |
| Unix timestamp | `1705320600` | Number | Epoch seconds |
| Unix millis | `1705320600000` | Number | Epoch milliseconds |
| Natural language | `"January 15, 2024"` | String | Human readable |
| Built-in function | `"{{$now}}"` | Dynamic | Generated values |
| Explicit casting | `Date("2024-01-15")` | Date | Force Date object |

## System Directives

The `.data` format includes special **system directives** that configure behavior. These are predefined keywords, not user variables.

### @COLLECTION_ID_TYPE (Predefined Directive)

**Purpose**: Set the default `_id` field type for an entire collection.

**Syntax**: `@COLLECTION_ID_TYPE collection_name = TypeName`

**Supported Types**:
- `ObjectId` - MongoDB ObjectId type
- `UUID` - Standard UUID format
- `String` - String type (default)
- `Number` - Numeric type

**Examples**:
```data
# System directive - sets collection ID type globally
@COLLECTION_ID_TYPE products = ObjectId
@COLLECTION_ID_TYPE users = String
@COLLECTION_ID_TYPE orders = UUID
@COLLECTION_ID_TYPE categories = Number

# Now all operations on these collections use the specified type automatically
INSERT products { "_id": "507f1f77bcf86cd799439020", "name": "Product" }  # → ObjectId
INSERT users { "_id": "user_123", "name": "John" }                        # → String
INSERT orders { "_id": "550e8400-e29b-41d4-a716-446655440000" }           # → UUID
INSERT categories { "_id": "42", "name": "Electronics" }                  # → Number
```

**Scope**: Once set, the directive applies to ALL subsequent operations on that collection within the file.

## User Variables

User-defined variables for reusable values:

```data
# User variables (defined by you)
@userId = "user_12345"
@productId = "{{$randomObjectId}}"
@timestamp = "{{$now}}"

# Use in operations
INSERT users { "_id": "{{userId}}", "createdAt": "{{timestamp}}" }
```

## Type Casting and ID Specification

When working with collections that don't have explicit schemas, you can specify field types using **implicit type casting** (automatic) or **explicit type casting functions** (when needed).

### Implicit Type Casting (Automatic)

The format automatically infers types from literal values:

```data
# Automatic type inference - no casting functions needed
INSERT products
{
  "_id": "507f1f77bcf86cd799439020",  # String literal → String (or ObjectId if 24-char hex)
  "name": "Wireless Headphones",      # String literal → String
  "price": 99.99,                     # Number literal → Number  
  "inStock": true,                    # Boolean literal → Boolean
  "quantity": 42                      # Integer literal → Number
}
```

**Implicit Rules:**
- `"text"` → `String("text")` (automatically)
- `42` → `Number(42)` (automatically)  
- `99.99` → `Number(99.99)` (automatically)
- `true`/`false` → `Boolean(true/false)` (automatically)

### Explicit Type Casting (When Needed)

Use explicit casting functions only for special types or when you need to override the natural type:

```data
# Explicit casting for special MongoDB types
INSERT products
{
  "_id": ObjectId("507f1f77bcf86cd799439020"),  # Force ObjectId
  "categoryId": ObjectId("507f1f77bcf86cd799439021"),
  "createdAt": Date("2024-01-15T10:30:00Z"),   # Force Date
  "sessionId": UUID("550e8400-e29b-41d4-a716-446655440000")  # Force UUID
}

# Override natural types when needed
INSERT mixed_data
{
  "_id": String("42"),        # Force "42" to be String, not Number
  "numericId": Number("123"), # Force "123" to be Number, not String
  "price": String("99.99")    # Force "99.99" to be String, not Number
}
```

### Collection Type Hints (System Directive)

The `@COLLECTION_ID_TYPE` directive is a **predefined system feature** that sets the `_id` field type for an entire collection:

```data
# System directive - not a user variable
@COLLECTION_ID_TYPE products = ObjectId

INSERT products
{
  "_id": "507f1f77bcf86cd799439020",  # String literal auto-converted to ObjectId
  "name": "Wireless Headphones",      # String literal → String (natural)
  "price": 99.99                      # Number literal → Number (natural)
}

FIND products WHERE _id = "507f1f77bcf86cd799439020"  # Also converted to ObjectId
```

### Smart Type Detection (for `_id` fields only)

For `_id` fields specifically, the format uses **pattern recognition** to automatically infer the correct type:

```data
INSERT mixed_ids
{
  # Smart detection based on _id value patterns:
  "_id": "507f1f77bcf86cd799439020",           # → ObjectId (24-char hex)
  # "_id": "550E8400-E29B-41D4-A716-446655440000", # → GUID (Microsoft format)
  # "_id": "550e8400-e29b-41d4-a716-446655440000", # → UUID (lowercase format)
  # "_id": 12345,                                # → Number (numeric)
  # "_id": "custom-string-id",                   # → String (fallback)
}
```

**Smart Detection Patterns for `_id`**:

| Pattern | Example | Detected Type | Priority |
|---------|---------|---------------|----------|
| **24-char hex** | `"507f1f77bcf86cd799439020"` | `ObjectId` | 1 (highest) |
| **GUID format** | `"550E8400-E29B-41D4-A716-446655440000"` | `GUID` | 2 |
| **UUID format** | `"550e8400-e29b-41d4-a716-446655440000"` | `UUID` | 3 |
| **Numeric** | `12345` or `"12345"` | `Number` | 4 |
| **Other strings** | `"custom-id"` | `String` | 5 (lowest) |

**GUID vs UUID Detection**:
- **GUID**: Microsoft format, typically uppercase: `"A1B2C3D4-E5F6-7890-ABCD-EF1234567890"`
- **UUID**: Standard format, typically lowercase: `"a1b2c3d4-e5f6-7890-abcd-ef1234567890"`
- Both use the same `8-4-4-4-12` hex digit pattern with hyphens

```data
# Examples of smart _id detection:
INSERT products_objectid { "_id": "507f1f77bcf86cd799439020" }        # → ObjectId
INSERT products_guid { "_id": "A1B2C3D4-E5F6-7890-ABCD-EF1234567890" } # → GUID  
INSERT products_uuid { "_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890" } # → UUID
INSERT products_number { "_id": 12345 }                                # → Number
INSERT products_string { "_id": "custom-product-001" }                 # → String
```

### Available Explicit Type Casting Functions

| Function | When to Use | Example |
|----------|-------------|---------|
| `ObjectId("hex")` | Force ObjectId type | `ObjectId("507f1f77bcf86cd799439020")` |
| `UUID("uuid")` | Force UUID type | `UUID("550e8400-e29b-41d4-a716-446655440000")` |
| `Date("iso")` | Force Date type | `Date("2024-01-15T10:30:00Z")` |
| `String("value")` | Override to String | `String("42")` # "42" as String, not Number |
| `Number("value")` | Override to Number | `Number("42")` # "42" as Number, not String |

**Key Point**: You only need explicit casting for ObjectId, UUID, Date, or when overriding the natural type!

## Database Connection

```data
### Database connection (required)
MONGO localhost:27017/database_name

### Variables (optional)
@variable = value

=== Setup Operations === // Create foundational data
INSERT accounts { "_id": "test-account", "name": "Test Account" }
INSERT users { "_id": "test-user", "accountId": "test-account" }

=== Main Test Operations === // Core testing logic
UPDATE users WHERE _id = "test-user" SET lastLogin = "{{$now}}"
FIND users WHERE accountId = "test-account" EXPECT 1 result

=== Cleanup Operations === // Remove test data
DELETE users WHERE accountId = "test-account"
DELETE accounts WHERE _id = "test-account"
```

## Section Format

Sections group related operations that run sequentially:

**Format**: `=== Section Name === // Optional comments`

**Examples**:
```data
=== Account Setup === // Create test accounts and users
=== Data Validation === // Verify data integrity
=== Performance Tests === // Load and benchmark operations
=== Cleanup === // Remove all test data
```

**Rules**:
- Sections are executed in order from top to bottom
- Operations within a section run sequentially
- Comments after `//` are optional but recommended for clarity
- Section names should be descriptive and unique

## Built-in Functions

**Dynamic Value Generation:**
- `{{$now}}` - Current timestamp (ISO string)
- `{{$timestamp}}` - Current Unix timestamp (seconds)
- `{{$randomGUID}}` - Generate Microsoft GUID format (most common)
- `{{$randomUUID}}` - Generate UUID v4 (RFC standard)
- `{{$randomObjectId}}` - Generate 24-character ObjectId hex string
- `{{$randomInt:1:100}}` - Random integer between 1-100
- `{{$randomFloat:1.0:10.0}}` - Random decimal between 1.0-10.0
- `{{$randomDate:2024-01-01:2024-12-31}}` - Random date in range
- `{{$randomChoice:red,green,blue}}` - Random selection from list
- `{{$randomBool}}` - Random true/false
- `{{$index}}` - Current iteration index (in loops)

**Date and Time Functions:**
- `{{$futureDate:7d}}` - Date 7 days from now
- `{{$futureDate:24h}}` - Date 24 hours from now
- `{{$futureDate:30m}}` - Date 30 minutes from now
- `{{$pastDate:30d}}` - Date 30 days ago
- `{{$pastDate:1h}}` - Date 1 hour ago
- `{{$pastDate:15m}}` - Date 15 minutes ago
- `{{$dateFormat:{{$now}}:YYYY-MM-DD}}` - Format date with pattern
- `{{$dateFormat:{{$now}}:MMM DD, YYYY}}` - Format as "Jan 15, 2024"
- `{{$dateFormat:{{$now}}:HH:mm:ss}}` - Format as "14:30:45"

**ObjectId Examples:**
```data
# Generate new ObjectId automatically
INSERT products
{
  "_id": "{{$randomObjectId}}",  # Generates: "507f1f77bcf86cd799439020"
  "name": "Product {{$index}}",
  "createdAt": "{{$now}}"
}

# Use in queries
FIND orders WHERE customerId = "{{$randomObjectId}}"

# Array of ObjectIds
@productIds = ["{{$randomObjectId}}", "{{$randomObjectId}}", "{{$randomObjectId}}"]
FIND products WHERE _id IN {{productIds}}
```

### Date and Time Parameters
- **d**: days (`7d` = 7 days)
- **h**: hours (`24h` = 24 hours) 
- **m**: minutes (`30m` = 30 minutes)
- **s**: seconds (`45s` = 45 seconds)

## Syntax Rules

### Single-Line vs Multi-Line

**Use Single-Line When:**
- Single condition in WHERE clause
- Single field in SET/SELECT clause
- Simple operations

**Use Multi-Line When:**
- Multiple conditions in WHERE clause
- Multiple fields in SET/SELECT clause
- Complex operations

### Examples

```data
# Single-line (simple)
DELETE users WHERE _id = "{{userId}}"
UPDATE users WHERE _id = "{{userId}}" SET lastLogin = "{{$now}}"
FIND users WHERE userType = 1 SELECT name, email

# Multi-line (complex)
DELETE users
WHERE:
  - isActive = false
  - created < "{{$pastDate:30d}}"

UPDATE users
WHERE:
  - userType = 1
  - isActive = true
SET:
  - loginCount += 1
  - lastSeen = "{{$now}}"
  - tags PUSH "verified"
```

## Core Operations

### INSERT Operations

**Single Document:**
```data
INSERT collection
{
  "_id": "{{id}}",
  "field": "value",
  "created": "{{$now}}"
}
```

**Multiple Documents:**
```data
INSERT_MANY collection
[
  { "_id": "id1", "name": "Item 1" },
  { "_id": "id2", "name": "Item 2" }
]
```

### UPDATE Operations

**Single-Line:**
```data
UPDATE users WHERE _id = "{{userId}}" SET lastLogin = "{{$now}}"
```

**Multi-Line:**
```data
UPDATE users
WHERE:
  - _id = "{{userId}}"
  - isActive = true
SET:
  - lastLogin = "{{$now}}"
  - loginCount += 1
  - tags PUSH "verified"
```

**Assignment Operators:**
- `=` - Set value
- `+=` - Increment 
- `-=` - Decrement
- `PUSH` - Add to array
- `REMOVE` - Remove from array
- `ADD_UNIQUE` - Add unique to array
- `REMOVE_ALL` - Remove multiple from array
- `UNSET` - Remove field
- `NOW` - Set current date

### UPSERT Operations

```data
UPSERT sessions
WHERE userId = "{{userId}}"
SET:
  - lastAccess = "{{$now}}"
  - isActive = true
SET_ON_INSERT:
  - _id = "{{$randomUUID}}"
  - created = "{{$now}}"
```

### DELETE Operations

```data
# Simple
DELETE logs WHERE level = "DEBUG"

# Complex
DELETE users
WHERE:
  - isActive = false
  - lastLogin < "{{$pastDate:90d}}"
  - loginCount = 0
```

### FIND Operations

```data
# Simple
FIND users WHERE userType = 1

# Complex
FIND users
WHERE:
  - userType IN [1, 2]
  - isActive = true
  - created >= "{{$pastDate:30d}}"
SELECT:
  - name
  - email
  - created AS joinDate
SORT:
  - created DESC
  - name ASC
LIMIT 50
SKIP 10
EXPECT 25 results
```

## Query Operators

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

## Complex Mathematical Operations

### Date Math Functions

```data
# Date extraction functions
FIND events WHERE YEAR(created) = 2024
FIND logs WHERE MONTH(timestamp) = 12
FIND sessions WHERE DAY(created) > 15
FIND activity WHERE HOUR(timestamp) BETWEEN 9 AND 17
FIND metrics WHERE MINUTE(recorded) = 0

# Date arithmetic
FIND orders WHERE created > DATE_ADD("{{$now}}", -30, "days")
FIND events WHERE expires < DATE_SUB("{{$now}}", 1, "hour")

# Complex date conditions
FIND users
WHERE:
  - YEAR(created) = 2024
  - MONTH(created) IN [1, 2, 3]
  - DAY_OF_WEEK(created) NOT IN [1, 7]  # Exclude weekends
```

### Mathematical Expressions

```data
# Mathematical operators
FIND products WHERE price MOD 10 = 0
FIND users WHERE ABS(score - 50) < 10
FIND metrics WHERE SQRT(value) > 5
FIND data WHERE POWER(base, 2) < 100

# Complex expressions with EXPR
FIND orders
WHERE:
  - total > 100
  - EXPR "this.discountAmount > this.total * 0.1"

# Aggregation with complex math
AGGREGATE sales
PIPELINE:
  - MATCH created >= "{{$pastDate:30d}}"
  - ADD_FIELDS:
      - profit = SUBTRACT(revenue, cost)
      - margin = DIVIDE(profit, revenue)
      - roundedMargin = ROUND(margin, 2)
  - MATCH profit > 0
```

### Advanced Expression Operations

```data
# Conditional expressions
UPDATE products
WHERE category = "electronics"
SET:
  - discountRate = COND(price > 1000, 0.1, 0.05)
  - finalPrice = MULTIPLY(price, SUBTRACT(1, discountRate))

# String operations with math
FIND users
WHERE:
  - STRLEN(name) > 5
  - SUBSTR(email, 0, 5) = "admin"
  - CONCAT(firstName, " ", lastName) CONTAINS "John"

# Array math operations
FIND orders
WHERE:
  - SIZE(items) > 2
  - AVG(items.price) > 50
  - SUM(items.quantity) < 10
```

## Performance Tuning

### Query Analysis with EXPLAIN

```data
# Simple explain
EXPLAIN FIND users WHERE email = "test@example.com"

# Complex explain with options
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

### Performance Profiling

```data
# Enable profiling for slow operations
PROFILE
LEVEL: 2
SLOW_MS: 100
OPERATIONS: ["find", "update", "aggregate"]

# Profile specific operation
PROFILE OPERATION:
  AGGREGATE orders
  PIPELINE:
    - MATCH status = "completed"
    - GROUP BY customerId:
        - totalSpent = SUM(amount)
        - orderCount = COUNT(*)
EXPECT avgTimeMs < 200

# Disable profiling
PROFILE LEVEL: 0
```

### Advanced Index Strategies

```data
# TTL Index (Time To Live)
CREATE_INDEX ON sessions
FIELDS:
  - expires ASC TTL
OPTIONS:
  - Name = "session_ttl_idx"
  - ExpireAfterSeconds = 3600

# Partial Index with filter expression
CREATE_INDEX ON users
FIELDS:
  - email ASC UNIQUE
OPTIONS:
  - Name = "active_user_email_idx"
  - PartialFilter:
      - isActive = true
      - isDeleted = false

# Compound Index with specific field order
CREATE_INDEX ON orders
FIELDS:
  - customerId ASC
  - status ASC  
  - created DESC
OPTIONS:
  - Name = "customer_order_lookup_idx"
  - Background = true

# Sparse Index (only indexes documents with the field)
CREATE_INDEX ON users
FIELDS:
  - phoneNumber ASC SPARSE
OPTIONS:
  - Name = "phone_sparse_idx"

# Multikey Index for arrays
CREATE_INDEX ON products
FIELDS:
  - tags ASC MULTIKEY
OPTIONS:
  - Name = "product_tags_idx"
```

### Index Optimization

```data
# Index usage hints
FIND orders
WHERE customerId = "{{customerId}}"
USE_INDEX "customer_order_lookup_idx"
SORT created DESC

# Index statistics
INDEX_STATS ON orders

# Index rebuild for optimization
REINDEX COLLECTION orders
```

### Query Optimization

```data
# Query with hint and comment for profiling
FIND products
WHERE:
  - category = "electronics"
  - price > 100
USE_INDEX "category_price_idx"
COMMENT "Product search optimization test"
MAX_TIME_MS 5000

# Aggregation with optimization hints
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

For complex scenarios that require native MongoDB syntax, use the `RAW` keyword as an escape hatch:

### Raw Query Syntax

```data
# Complex aggregation with raw MongoDB operators
FIND orders
WHERE:
  - status = "completed"
  - RAW: { "items": { "$elemMatch": { "price": { "$gt": 100 }, "category": "electronics" } } }
  - created >= "{{$pastDate:30d}}"

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

### Raw Aggregation Pipelines

```data
# Hybrid aggregation with both readable and raw syntax
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

## Advanced Aggregation

### Complex Pipeline Operations

```data
# Advanced aggregation with multiple stages
AGGREGATE ecommerce_events
PIPELINE:
  - MATCH:
      - eventType IN ["purchase", "add_to_cart", "view_product"]
      - timestamp >= "{{$pastDate:30d}}"
  - LOOKUP orders ON userId = customerId AS userOrders
  - UNWIND userOrders
  - GROUP BY userId:
      - totalEvents = COUNT(*)
      - totalPurchases = SUM(COND(eventType = "purchase", 1, 0))
      - avgOrderValue = AVG(userOrders.amount)
      - conversionRate = DIVIDE(totalPurchases, totalEvents)
  - MATCH:
      - totalEvents > 10
      - conversionRate > 0.05
  - SORT conversionRate DESC
  - LIMIT 100
```

### Advanced Lookup Operations

```data
# Complex lookup with pipeline
AGGREGATE customers
PIPELINE:
  - LOOKUP:
      - FROM orders
      - LET customerId = _id
      - PIPELINE:
          - MATCH:
              - EXPR customerId = customerRef
              - status = "completed"
              - created >= "{{$pastDate:90d}}"
          - GROUP BY null:
              - totalSpent = SUM(amount)
              - orderCount = COUNT(*)
      - AS customerStats
  - UNWIND customerStats
  - MATCH customerStats.totalSpent > 500
```

## Advanced Features

### Transactions

```data
BEGIN_TRANSACTION

# Transfer money between accounts
UPDATE accounts WHERE _id = "{{fromAccount}}" SET balance -= 100
UPDATE accounts WHERE _id = "{{toAccount}}" SET balance += 100

# Record transaction
INSERT transactions
{
  "_id": "{{$randomUUID}}",
  "from": "{{fromAccount}}",
  "to": "{{toAccount}}",
  "amount": 100,
  "timestamp": "{{$now}}"
}

COMMIT_TRANSACTION
```

### Conditional Logic

```data
# Complex conditional with count
IF COUNT orders WHERE customerId = "{{customerId}}" AND status = "pending" > 5
THEN:
  UPDATE customers WHERE _id = "{{customerId}}" SET riskLevel = "high"
  INSERT alerts { "_id": "{{$randomUUID}}", "type": "high_pending_orders", "customerId": "{{customerId}}" }
ELSE:
  UPDATE customers WHERE _id = "{{customerId}}" SET riskLevel = "normal"
```

### Schema Validation

**Collection Creation with DBSoup-inspired Schema:**
```data
# Create collection with structured schema definition
CREATE_COLLECTION user_profiles
SCHEMA:
  * _id           : String                    [PK,REQUIRED]
  * userId        : String                    [REQUIRED,FK:users._id]
  * email         : String                    [REQUIRED,UNIQUE,FORMAT:email]
  - firstName     : String                    [MAXLENGTH:50]
  - lastName      : String                    [MAXLENGTH:50]
  - age           : Number                    [RANGE:18:120]
  - status        : String                    [ENUM:active,inactive,suspended,DEFAULT:active]
  - preferences   : Object                    [OPTIONAL]
  - tags          : Array<String>             [OPTIONAL]
  - metadata      : Object                    [OPTIONAL]
  - createdAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]
  - updatedAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]

# Create collection with validation rules
CREATE_COLLECTION orders
SCHEMA:
  * _id           : String                    [PK,REQUIRED]
  * customerId    : String                    [REQUIRED,FK:customers._id]
  * orderNumber   : String                    [REQUIRED,UNIQUE,PATTERN:^ORD-\d{8}$]
  * status        : String                    [REQUIRED,ENUM:pending,processing,shipped,delivered,cancelled]
  * items         : Array<Object>             [REQUIRED,MIN_ITEMS:1]
  * total         : Number                    [REQUIRED,MIN:0]
  - discountCode  : String                    [MAXLENGTH:20]
  - shippingAddr  : Object                    [REQUIRED_IF:status=shipped]
  - notes         : String                    [MAXLENGTH:500]
  * createdAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]
  - updatedAt     : Date                      [SYSTEM,DEFAULT:CURRENT_TIMESTAMP]

# Alternative: Raw JSON Schema for complex cases
CREATE_COLLECTION complex_analytics
SCHEMA_JSON: {
  "type": "object",
  "properties": {
    "_id": { "type": "string" },
    "complexNestedStructure": {
      "type": "object",
      "properties": {
        "deeply": {
          "type": "object",
          "properties": {
            "nested": { "type": "array" }
          }
        }
      }
    }
  }
}
```

**Schema Validation:**
```data
# Validate existing documents against schema
VALIDATE_SCHEMA user_profiles
EXPECT_VALID > 0.95

# Validate specific documents
VALIDATE_SCHEMA orders
WHERE createdAt >= "{{$pastDate:7d}}"
EXPECT_VALID = 1.0
```

**DBSoup Field Prefixes in Schema:**
- `*` - Required field (cannot be null)
- `-` - Optional field (can be null)
- `!` - Indexed field (has database index)
- `@` - Sensitive/encrypted field
- `~` - Masked field (data masking)
- `>` - Partitioned field (sharding key)
- `$` - Audit field (audit trail)
```

### Load Testing

```data
# Advanced load test with complex data
LOAD_TEST realistic_users
COUNT 10000
TEMPLATE:
  - _id = "user-{{$index}}"
  - email = "user{{$index}}@test.com"
  - profile = {
      - name = "User {{$index}}"
      - age = RANDOM_INT(18, 80)
      - preferences = [
          RANDOM_CHOICE("notifications", "email", "sms"),
          RANDOM_CHOICE("dark", "light", "auto")
        ]
      - location = {
          - lat = RANDOM_FLOAT(-90.0, 90.0)
          - lng = RANDOM_FLOAT(-180.0, 180.0)
        }
    }
  - created = RANDOM_DATE("2020-01-01", "2024-01-01")
  - metadata = {
      - source = "load_test"
      - batch = "{{$randomGUID}}"
    }
```

### Error Handling

```data
# Advanced error handling with specific error types
TRY:
  UPDATE inventory WHERE productId = "{{productId}}" SET stock -= 1
  INSERT orders { "_id": "{{orderId}}", "productId": "{{productId}}", "quantity": 1 }
CATCH ValidationError:
  INSERT error_log { "_id": "{{$randomUUID}}", "type": "validation", "message": "Invalid product data" }
CATCH DuplicateKeyError:
  UPDATE orders WHERE _id = "{{orderId}}" SET quantity += 1
CATCH:
  INSERT error_log { "_id": "{{$randomUUID}}", "type": "unknown", "error": "{{$error}}" }
```

## Complete Example

```data
### Complete .data File with Advanced Features
MONGO localhost:27017/advanced_ecommerce

### Variables
@customerId = customer-{{$randomGUID}}
@orderId = order-{{$randomObjectId}}
@productId = product-electronics-{{$randomInt:1000:9999}}

### Advanced schema with TTL
CREATE_COLLECTION user_sessions
WITH_SCHEMA:
  Required Fields:
    - _id (String)
    - userId (String)
    - expires (Date)
    - isActive (Boolean)

### TTL index for automatic cleanup
CREATE_INDEX ON user_sessions
FIELDS:
  - expires ASC TTL
OPTIONS:
  - Name = "session_expiry_idx"
  - ExpireAfterSeconds = 0

### Complex transaction with mathematical operations
BEGIN_TRANSACTION

# Create customer with profile
INSERT customers
{
  "_id": "{{customerId}}",
  "email": "customer{{$randomInt:1:1000}}@advanced-test.com",
  "profile": {
    "name": "Advanced Customer {{$index}}",
    "tier": "{{$randomChoice:bronze,silver,gold,platinum}}",
    "joinDate": "{{$now}}",
    "preferences": {
      "notifications": "{{$randomBool}}",
      "marketing": "{{$randomBool}}"
    }
  },
  "stats": {
    "totalOrders": 0,
    "totalSpent": 0,
    "averageOrderValue": 0
  }
}

# Create order with complex pricing
INSERT orders
{
  "_id": "{{orderId}}",
  "_id": "{{orderId}}",
  "items": [
    {
      "productId": "{{productId}}",
      "quantity": "{{$randomInt:1:5}}",
      "unitPrice": "{{$randomFloat:10.0:500.0}}",
      "discount": 0.1
    }
  ],
  "pricing": {
    "subtotal": "{{$randomFloat:50.0:1000.0}}",
    "tax": 0,
    "shipping": "{{$randomFloat:5.0:25.0}}",
    "total": 0
  },
  "status": "processing",
  "created": "{{$now}}"
}

# Update pricing with mathematical operations
UPDATE orders
WHERE _id = "{{orderId}}"
SET:
  - RAW: {
      "$set": {
        "pricing.tax": { "$multiply": ["$pricing.subtotal", 0.08] },
        "pricing.total": { 
          "$add": [
            "$pricing.subtotal",
            { "$multiply": ["$pricing.subtotal", 0.08] },
            "$pricing.shipping"
          ]
        }
      }
    }

COMMIT_TRANSACTION

### Performance analysis
EXPLAIN
OPERATION:
  FIND orders
  WHERE:
    - customerId = "{{customerId}}"
    - status IN ["processing", "shipped"]
    - created >= "{{$pastDate:30d}}"
OPTIONS:
  - Mode = "executionStats"
  - Verbose = true

### Complex aggregation with raw MongoDB for advanced features
AGGREGATE orders
PIPELINE:
  - MATCH created >= "{{$pastDate:90d}}"
  - LOOKUP customers ON customerId = _id AS customer
  - UNWIND customer
  - ADD_FIELDS:
      - profit = SUBTRACT("$pricing.total", "$pricing.subtotal")
      - isHighValue = COND(GT("$pricing.total", 500), true, false)
      - customerTier = "$customer.profile.tier"
  - RAW: {
      "$group": {
        "_id": {
          "tier": "$customerTier",
          "month": { "$month": "$created" }
        },
        "totalOrders": { "$sum": 1 },
        "totalRevenue": { "$sum": "$pricing.total" },
        "avgOrderValue": { "$avg": "$pricing.total" },
        "highValueOrders": { "$sum": { "$cond": ["$isHighValue", 1, 0] } }
      }
    }
  - SORT:
      - _id.tier ASC
      - _id.month ASC

### Profile the aggregation performance
PROFILE OPERATION:
  AGGREGATE orders
  PIPELINE:
    - MATCH status = "completed"
    - GROUP BY customerId:
        - orderCount = COUNT(*)
        - totalSpent = SUM("pricing.total")
EXPECT avgTimeMs < 100

### Cleanup with mathematical conditions
DELETE orders
WHERE:
  - status = "cancelled"
  - YEAR(created) < 2024
  - MONTH(created) < 6

DELETE customers WHERE _id = "{{customerId}}"
```

This comprehensive format now supports:
- **Complex mathematical operations** with date functions and expressions
- **Performance tuning** with explain plans, profiling, and advanced indexing
- **Hybrid raw MongoDB syntax** for complex scenarios
- **Extended built-in functions** including GUID and ObjectId generation
- **Advanced optimization** strategies and query analysis tools 

## Advanced MongoDB Features

### GridFS Operations

**File Storage:**
```data
# Store file in GridFS
GRIDFS_PUT collection
FILE: "/path/to/file.pdf"
METADATA: { "category": "documents", "uploadedBy": "{{userId}}" }
FILENAME: "document-{{$now}}.pdf"
CHUNK_SIZE: 261120

# Store from base64 data
GRIDFS_PUT_DATA collection
DATA: "{{base64FileData}}"
METADATA: { "type": "image", "size": 1024000 }
FILENAME: "image-{{$randomUUID}}.png"
```

**File Retrieval:**
```data
# Get file by ID
GRIDFS_GET collection WHERE _id = "{{fileId}}" SAVE_TO: "/downloads/file.pdf"

# Get file by filename
GRIDFS_GET collection WHERE filename = "document.pdf" SAVE_TO: "/downloads/"

# Get file metadata only
GRIDFS_FIND collection WHERE metadata.category = "documents"
SELECT:
  - filename
  - uploadDate
  - length
  - metadata
```

**File Management:**
```data
# Delete file
GRIDFS_DELETE collection WHERE _id = "{{fileId}}"

# Rename file
GRIDFS_RENAME collection WHERE _id = "{{fileId}}" FILENAME: "new-name.pdf"

# Update metadata
GRIDFS_UPDATE_METADATA collection WHERE _id = "{{fileId}}" 
SET:
  - metadata.category = "archived"
  - metadata.lastModified = "{{$now}}"
```

### Change Streams

**Real-time Monitoring:**
```data
# Watch collection changes
WATCH collection
PIPELINE:
  - MATCH:
      - operationType IN ["insert", "update", "delete"]
      - fullDocument.userId = "{{userId}}"
RESUME_TOKEN: "{{resumeToken}}"
MAX_AWAIT_TIME: 5000
ON_CHANGE:
  INSERT change_log
  {
    "_id": "{{$randomUUID}}",
    "operationType": "{{change.operationType}}",
    "documentId": "{{change.documentKey._id}}",
    "timestamp": "{{$now}}"
  }

# Watch database changes
WATCH_DATABASE
FILTER:
  - operationType = "insert"
  - ns.coll IN ["users", "orders", "products"]
ON_CHANGE:
  UPDATE metrics WHERE type = "realtime" SET eventCount += 1
```

**Change Stream Management:**
```data
# Get resume tokens
GET_RESUME_TOKEN collection SAVE_AS: "resumeToken"

# Close change stream
CLOSE_WATCH collection

# Watch with custom pipeline
WATCH orders
PIPELINE:
  - MATCH:
      - operationType = "update"
      - updateDescription.updatedFields.status EXISTS
      - fullDocument.total > 1000
ON_CHANGE:
  # Send notification for high-value order updates
  INSERT notifications
  {
    "_id": "{{$randomUUID}}",
    "type": "high_value_order_update",
    "orderId": "{{change.documentKey._id}}",
    "newStatus": "{{change.updateDescription.updatedFields.status}}"
  }
```

### Advanced Geospatial Operations

**Complex Spatial Queries:**
```data
# Geospatial within polygon
FIND locations
WHERE:
  - type = "restaurant"
  - coordinates GEO_WITHIN {
      "type": "Polygon",
      "coordinates": [[[
        [-74.0, 40.7], [-74.0, 40.8], 
        [-73.9, 40.8], [-73.9, 40.7], 
        [-74.0, 40.7]
      ]]]
    }

# Geospatial intersection
FIND zones
WHERE:
  - geometry GEO_INTERSECTS {
      "type": "LineString",
      "coordinates": [[-74.0, 40.7], [-73.9, 40.8]]
    }

# Near with distance
FIND stores
WHERE:
  - location GEO_NEAR {
      "point": [-74.0060, 40.7128],
      "maxDistance": 1000,
      "minDistance": 10
    }
SELECT:
  - name
  - location
  - distance
SORT distance ASC
```

**Geospatial Calculations:**
```data
# Calculate distance between points
FIND users
WHERE:
  - location EXISTS
  - GEO_DISTANCE(location, [-74.0060, 40.7128]) < 5000
SELECT:
  - name
  - location
  - GEO_DISTANCE(location, [-74.0060, 40.7128]) AS distanceMeters

# Area calculations
FIND properties
WHERE:
  - boundary EXISTS
  - GEO_AREA(boundary) > 10000
SELECT:
  - address
  - GEO_AREA(boundary) AS areaSquareMeters
  - GEO_CENTROID(boundary) AS center
```

## Advanced Statistical Functions

### Statistical Analysis

**Basic Statistics:**
```data
# Advanced aggregation with statistics
AGGREGATE sales
PIPELINE:
  - MATCH created >= "{{$pastDate:30d}}"
  - GROUP BY productCategory:
      - count = COUNT(*)
      - mean = AVG(amount)
      - median = MEDIAN(amount)
      - mode = MODE(amount)
      - stdDev = STD_DEV(amount)
      - variance = VARIANCE(amount)
      - percentile25 = PERCENTILE(amount, 25)
      - percentile75 = PERCENTILE(amount, 75)
      - skewness = SKEWNESS(amount)
      - kurtosis = KURTOSIS(amount)
```

**Correlation and Regression:**
```data
# Correlation analysis
AGGREGATE user_metrics
PIPELINE:
  - MATCH isActive = true
  - GROUP BY null:
      - correlation = CORRELATION(loginCount, purchaseAmount)
      - covariance = COVARIANCE(loginCount, purchaseAmount)
      - regression = LINEAR_REGRESSION(loginCount, purchaseAmount)

# Advanced statistical tests
FIND experiments
WHERE experimentType = "ab_test"
SELECT:
  - experimentId
  - TTEST(controlGroup.results, testGroup.results) AS pValue
  - ANOVA(groups.results) AS fStatistic
  - CHI_SQUARE(observed, expected) AS chiSquare
```

### Time-Series Analysis

**Advanced Windowing:**
```data
# Moving averages and trends
AGGREGATE time_series_data
PIPELINE:
  - MATCH timestamp >= "{{$pastDate:90d}}"
  - SORT timestamp ASC
  - ADD_FIELDS:
      - movingAvg7 = MOVING_AVERAGE(value, 7)
      - movingAvg30 = MOVING_AVERAGE(value, 30)
      - exponentialAvg = EXPONENTIAL_MOVING_AVERAGE(value, 0.1)
      - trend = LINEAR_TREND(value, timestamp)
      - seasonality = SEASONAL_DECOMPOSE(value, 7)

# Forecasting
AGGREGATE sales_data
PIPELINE:
  - MATCH date >= "{{$pastDate:365d}}"
  - TIME_SERIES_FORECAST:
      - valueField = "amount"
      - timeField = "date"
      - periods = 30
      - method = "linear"
      - confidence = 0.95
```

### Machine Learning Integration

**Data Preprocessing:**
```data
# Feature engineering
AGGREGATE user_behavior
PIPELINE:
  - MATCH lastActive >= "{{$pastDate:30d}}"
  - ADD_FIELDS:
      - normalizedSpend = NORMALIZE(totalSpent, "minmax")
      - scaledLoginCount = SCALE(loginCount, "zscore")
      - encodedCategory = ONE_HOT_ENCODE(userCategory)
      - binned_age = DISCRETIZE(age, [18, 30, 50, 65, 100])

# Data quality checks
VALIDATE_DATA user_profiles
CHECKS:
  - NULL_CHECK: ["email", "userId"]
  - RANGE_CHECK: { "age": [0, 150], "score": [0, 100] }
  - FORMAT_CHECK: { "email": "email", "phone": "phone" }
  - OUTLIER_CHECK: { "totalSpent": 3 }  # 3 standard deviations
EXPECT_QUALITY > 0.95
```

**Model Operations:**
```data
# Train model (placeholder for ML pipeline integration)
TRAIN_MODEL user_churn_prediction
FEATURES:
  - loginFrequency
  - totalSpent
  - daysSinceLastLogin
  - supportTickets
TARGET: churned
ALGORITHM: "random_forest"
VALIDATION: "cross_validation"
SAVE_MODEL_AS: "churn_model_v1"

# Apply model predictions
PREDICT user_predictions
MODEL: "churn_model_v1"
DATA: users
WHERE:
  - isActive = true
  - lastLogin >= "{{$pastDate:90d}}"
OUTPUT_FIELD: "churnProbability"
```

### Replica Set and Sharding Operations

**Replica Set Management:**
```data
# Check replica set status
REPLICA_SET_STATUS
EXPECT:
  - primary EXISTS
  - secondaries >= 2
  - health = "ok"

# Read preference configuration
SET_READ_PREFERENCE
MODE: "secondaryPreferred"
TAG_SETS: [{ "region": "us-east" }, { "region": "us-west" }]
MAX_STALENESS: 120

# Write concern settings
SET_WRITE_CONCERN
W: "majority"
J: true
W_TIMEOUT: 5000
```

**Sharding Operations:**
```data
# Enable sharding on database
ENABLE_SHARDING database_name

# Shard collection
SHARD_COLLECTION users
SHARD_KEY: { "userId": "hashed" }
UNIQUE: false

# Add shard
ADD_SHARD "shard1/replica1:27017,replica2:27017,replica3:27017"

# Check shard distribution
SHARD_STATS users
EXPECT_BALANCED: true
MAX_CHUNK_IMBALANCE: 2

# Move chunk
MOVE_CHUNK users
CHUNK: { "userId": { "$minKey": 1 } }
TO_SHARD: "shard2"
``` 

### Date vs String Detection (Without Schema)

When no schema is defined, the `.data` format uses **content-based pattern recognition** to determine whether a string should be treated as a Date object or remain as a String:

#### **Default Behavior: Strings Remain Strings**

By default, **all string values remain as strings** unless explicitly cast or automatically detected:

```data
INSERT events
{
  "_id": "event_001",
  "title": "Meeting with client",       # Remains as String (no number start)
  "description": "Important discussion", # Remains as String (no number start)
  "status": "scheduled"                 # Remains as String (no number start)
}
```

#### **Automatic Date Detection (Content-Based)**

Automatic date detection occurs when a **string value starts with a number** and matches recognized date patterns:

```data
INSERT records
{
  "_id": "record_001",
  
  # Values starting with numbers - scanned for date patterns:
  "eventDate": "2024-01-15T10:30:00Z",       # → Date object (ISO 8601)
  "scheduledFor": "2024-01-15",              # → Date object (date only)
  "deadline": "2024/01/15",                  # → Date object (slash format)
  "meetingTime": "01/15/2024",               # → Date object (US format)
  "dueDate": "15/01/2024",                   # → Date object (European format)
  "timestamp": "1705320600000",              # → Date object (Unix millis)
  "createdAt": "1705320600",                 # → Date object (Unix seconds)
  "publishDate": "15 January 2024",          # → Date object (natural language)
  
  # Values NOT starting with numbers - remain strings:
  "title": "Meeting on 2024-01-15",         # → String (starts with letter)
  "description": "Due January 15th",        # → String (starts with letter)
  "status": "active",                       # → String (starts with letter)
  "category": "important"                   # → String (starts with letter)
}
```

#### **Recognized Date Patterns**

When a string **starts with a digit (0-9)**, it's scanned against these patterns:

| Pattern | Example | Format Description |
|---------|---------|-------------------|
| **ISO 8601 UTC** | `"2024-01-15T10:30:00Z"` | Standard UTC timestamp |
| **ISO with timezone** | `"2024-01-15T10:30:00-05:00"` | With timezone offset |
| **Date only (ISO)** | `"2024-01-15"` | ISO date without time |
| **Date slash format** | `"2024/01/15"` | Alternative slash format |
| **US date format** | `"01/15/2024"`, `"1/15/2024"` | Month/Day/Year |
| **European format** | `"15/01/2024"`, `"15.01.2024"` | Day/Month/Year |
| **Unix timestamp** | `"1705320600"` | Seconds since epoch |
| **Unix milliseconds** | `"1705320600000"` | Milliseconds since epoch |
| **Natural language** | `"15 January 2024"`, `"2024 January 15"` | Human readable |
| **Time only** | `"10:30:00"`, `"14:30"` | Time without date |

#### **Content Detection Examples**

```data
INSERT mixed_content
{
  "_id": "content_001",
  
  # These start with numbers → scanned for date patterns → detected as dates:
  "eventDate": "2024-01-15T10:30:00Z",       # → Date object (ISO detected)
  "scheduledFor": "2024-01-15",              # → Date object (date detected)
  "meetingTime": "01/15/2024 10:30 AM",      # → Date object (US format detected)
  "deadline": "15/01/2024",                  # → Date object (European detected)
  "timestamp": "1705320600000",              # → Date object (Unix millis detected)
  "startTime": "09:30:00",                   # → Date object (time detected)
  "year": "2024",                           # → Date object (year detected)
  
  # These start with numbers → scanned → NOT date patterns → remain strings:
  "phoneNumber": "555-123-4567",             # → String (not a date pattern)
  "productCode": "12345-ABC",                # → String (not a date pattern)
  "version": "2024.1.5",                     # → String (not a date pattern)
  "quantity": "100",                         # → String (could be number but not date)
  
  # These don't start with numbers → remain strings (not scanned):
  "title": "Meeting on 2024-01-15",          # → String (starts with 'M')
  "description": "Due January 15th",         # → String (starts with 'D')
  "status": "active"                         # → String (starts with 'a')
}
```

#### **Explicit Control**

**Force Date Object** - Use `Date()` casting:
```data
INSERT appointments
{
  "_id": "appt_001",
  "eventDate": Date("2024-01-15T10:30:00Z"), # Force Date (would auto-detect anyway)
  "title": Date("2024-01-15"),               # Force Date (unusual but possible)
  "description": Date("15 January 2024")     # Force Date (unusual but possible)
}
```

**Force String** - Use `String()` casting:
```data
INSERT metadata
{
  "_id": "meta_001",
  "versionDate": String("2024-01-15"),       # Force String (override auto-detection)
  "buildNumber": String("20240115"),         # Force String (override auto-detection)
  "timestamp": String("1705320600")          # Force String (override auto-detection)
}
```

#### **Detection Algorithm**

The detection process follows these steps:

1. **Check if string starts with digit (0-9)**
   - If NO → remain as String (no scanning)
   - If YES → proceed to pattern matching

2. **Scan against date patterns** (in priority order):
   - ISO 8601 formats
   - Common date formats (US, European)
   - Unix timestamps
   - Natural language dates
   - Time-only formats

3. **If pattern matches** → convert to Date object
4. **If no pattern matches** → remain as String

#### **Best Practices**

**1. Rely on automatic detection** for most cases:
```data
INSERT events
{
  "_id": "event_001",
  "eventDate": "2024-01-15T10:30:00Z",       # Auto-detected as Date
  "scheduledFor": "2024-01-15",              # Auto-detected as Date  
  "meetingTime": "01/15/2024",               # Auto-detected as Date
  "title": "Important Meeting"               # Remains String (no number start)
}
```

**2. Use explicit casting** when you need to override detection:
```data
INSERT metadata
{
  "_id": "meta_001",
  "versionString": String("2024-01-15"),     # Force String (version info)
  "buildDate": Date("January 15, 2024"),     # Force Date (starts with letter)
  "releaseNumber": "2024.1.5"               # Remains String (not date pattern)
}
```

**3. Use built-in functions** for dynamic dates:
```data
INSERT sessions
{
  "_id": "session_001",
  "createdAt": "{{$now}}",                   # Always Date (function)
  "expiresAt": "{{$futureDate:24h}}",        # Always Date (function)
  "customField": "{{$timestamp}}"            # Number (Unix timestamp function)
}
```

#### **Detection Priority Order**

1. **Explicit Casting** (`Date()`, `String()`) - **Highest Priority**
2. **Built-in Date Functions** (`{{$now}}`, `{{$futureDate:7d}}`)
3. **Content-Based Pattern Detection** (strings starting with numbers)
4. **Default String Behavior** - **Lowest Priority**

#### **Examples Summary**

```data
INSERT comprehensive_example
{
  "_id": "comp_001",
  
  # Priority 1: Explicit casting (highest)
  "forceDate": Date("2024-01-15"),           # → Date object (explicit)
  "forceString": String("2024-01-15"),       # → String (explicit override)
  
  # Priority 2: Built-in functions
  "dynamicDate": "{{$now}}",                 # → Date object (function)
  "dynamicTimestamp": "{{$timestamp}}",      # → Number (function)
  
  # Priority 3: Content-based detection (medium priority)
  "eventDate": "2024-01-15T10:30:00Z",       # → Date object (starts with 2, matches ISO)
  "meetingTime": "01/15/2024",               # → Date object (starts with 0, matches US)
  "phoneNumber": "555-123-4567",             # → String (starts with 5, no date match)
  "version": "2024.1.5",                     # → String (starts with 2, no date match)
  
  # Priority 4: Default string behavior (lowest)
  "title": "Meeting on 2024-01-15",          # → String (starts with M, not scanned)
  "description": "Important discussion",      # → String (starts with I, not scanned)
  "status": "active"                         # → String (starts with a, not scanned)
}
```

This **content-based approach** is much more flexible and doesn't rely on naming conventions! 🎯