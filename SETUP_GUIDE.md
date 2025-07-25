# .data File Setup and Execution Guide

## Prerequisites

### 1. MongoDB Installation
```bash
# Install MongoDB Community Edition
# For macOS (using Homebrew):
brew tap mongodb/brew
brew install mongodb-community

# Start MongoDB service
brew services start mongodb/brew/mongodb-community

# Verify MongoDB is running
mongosh --eval "db.adminCommand('ismaster')"
```

### 2. Project Dependencies
```bash
# Ensure Node.js and npm are installed
node --version
npm --version

# Install project dependencies
npm install
```

## File Structure

Your `.data` file should be structured as follows:

```data
### Example API Test Data Setup
MONGO localhost:27017/roadwarrior

### Variables for consistency
@accountId = test-account-123
@userId = test-user-456

### Insert test account
INSERT accounts
{
  "_id": "{{accountId}}",
  "name": "Test Account",
  "created": "{{$now}}"
}

### Simple user insert
INSERT users WHERE _id = "{{userId}}" SET name = "Test User"

### Complex user setup
INSERT users
{
  "_id": "{{userId}}",
  "accountId": "{{accountId}}",
  "email": "test@example.com",
  "userType": 1,
  "isActive": true,
  "created": "{{$now}}"
}

### Complex query example  
FIND users
WHERE:
  - accountId = "{{accountId}}"
  - userType = 1
  - isActive = true
SELECT:
  - name
  - email
  - created
SORT created DESC
LIMIT 10
EXPECT 1 result

### === CLEANUP ===
DELETE users WHERE accountId = "{{accountId}}"
DELETE accounts WHERE _id = "{{accountId}}"
```

## Execution Steps

### Step 1: Prepare Your .data File

1. Create your `.data` file with the database connection
2. Define variables for reusable values
3. Add your data operations (INSERT, UPDATE, DELETE, FIND)
4. Include cleanup operations at the end

### Step 2: Manual Execution (Current Method)

**Option A: Using MongoDB Compass or mongosh**
```bash
# Connect to your database
mongosh "mongodb://localhost:27017/roadwarrior"

# Manually execute each operation, substituting variables:
db.accounts.insertOne({
  "_id": "test-account-123", 
  "name": "Test Account",
  "created": new Date()
})

db.users.insertOne({
  "_id": "test-user-456",
  "accountId": "test-account-123", 
  "email": "test@example.com",
  "userType": 1,
  "isActive": true,
  "created": new Date()
})

# Verify data
db.users.find({"accountId": "test-account-123", "userType": 1}).pretty()
```

**Option B: Generate MongoDB Script**
```javascript
// Convert your .data operations to MongoDB JavaScript
use roadwarrior

// Variables
const accountId = "test-account-123"
const userId = "test-user-456"
const now = new Date()

// Insert operations
db.accounts.insertOne({
  "_id": accountId,
  "name": "Test Account", 
  "created": now
})

db.users.insertOne({
  "_id": userId,
  "accountId": accountId,
  "email": "test@example.com",
  "userType": 1,
  "isActive": true,
  "created": now
})

// Query verification
const result = db.users.find({
  "accountId": accountId,
  "userType": 1,
  "isActive": true
}).toArray()

print(`Found ${result.length} users`)
print(JSON.stringify(result, null, 2))

// Cleanup
db.users.deleteMany({"accountId": accountId})
db.accounts.deleteMany({"_id": accountId})
```

### Step 3: Start Your API Server

```bash
# In your project directory
npm start
# or
node app/core-api/server.ts
```

The API should be available at `http://localhost:3000` (or your configured port).

### Step 4: Execute HTTP Tests

Create your corresponding `.http` file:

```http
### Test GetDrivers API with inserted data
GET http://localhost:3000/GetDrivers?startEpoc=1640995200&endEpoc=1672531199
AuthToken: {{sessionToken}}

### Expected Response:
# HTTP/1.1 200 OK
# Content-Type: application/json
# 
# [
#   {
#     "_id": "test-user-456",
#     "name": "Test User",
#     "email": "test@example.com", 
#     "userType": 1,
#     "isActive": true
#   }
# ]
```

## Future: Automated .data Runner

### Planned Implementation

A future `.data` runner tool will:

1. **Parse .data files** - Read and validate syntax
2. **Variable substitution** - Replace `{{variable}}` and function calls
3. **Execute operations** - Run MongoDB operations in sequence
4. **Transaction support** - Handle BEGIN/COMMIT/ROLLBACK
5. **Error handling** - Capture and report execution errors
6. **Performance monitoring** - Track operation timing

### Expected Usage

```bash
# Execute .data file
data-runner execute get-drivers.data

# Execute with cleanup
data-runner execute get-drivers.data --cleanup

# Dry run (validate only)
data-runner validate get-drivers.data

# Benchmark mode
data-runner benchmark get-drivers.data --iterations 100
```

## Expected Data Structure After Execution

After running the `/GetDrivers` endpoint data setup, you should have:

### Accounts Collection
```javascript
{
  "_id": "test-account-123",
  "name": "Test Account",
  "email": "test@example.com", 
  "isActive": true,
  "created": ISODate("2024-01-15T10:00:00Z")
}
```

### Users Collection (Drivers)
```javascript
[
  {
    "_id": "driver-active-1",
    "accountId": "test-account-123",
    "name": "John Driver",
    "email": "john@example.com",
    "userType": 1,  // RwRoleTypes.Driver
    "isActive": true,
    "isDeleted": false,
    "created": ISODate("2024-01-15T10:00:00Z")
  },
  {
    "_id": "driver-active-2", 
    "accountId": "test-account-123",
    "name": "Jane Driver",
    "email": "jane@example.com",
    "userType": 1,
    "isActive": true,
    "isDeleted": false,
    "created": ISODate("2024-01-15T10:00:00Z")
  }
]
```

### Sessions Collection
```javascript
{
  "_id": "test-session-123",
  "userId": "admin-user-123",
  "accountId": "test-account-123", 
  "orgId": "test-org-123",
  "token": "test-auth-token-123",
  "expires": ISODate("2024-01-22T10:00:00Z"),
  "isActive": true,
  "created": ISODate("2024-01-15T10:00:00Z")
}
```

## Expected API Response

After data setup, calling the GetDrivers API should return:

```json
{
  "code": 200,
  "data": [
    {
      "_id": "driver-active-1",
      "name": "John Driver", 
      "email": "john@example.com",
      "userType": 1,
      "isActive": true,
      "created": "2024-01-15T10:00:00.000Z"
    },
    {
      "_id": "driver-active-2",
      "name": "Jane Driver",
      "email": "jane@example.com", 
      "userType": 1,
      "isActive": true,
      "created": "2024-01-15T10:00:00.000Z"
    }
  ]
}
```

## Troubleshooting

### Common Issues

1. **MongoDB Connection Failed**
   ```bash
   # Check if MongoDB is running
   brew services list | grep mongodb
   
   # Start MongoDB if stopped
   brew services start mongodb/brew/mongodb-community
   ```

2. **Database/Collection Not Found**
   - MongoDB creates databases and collections automatically on first insert
   - Ensure your connection string database name matches your API configuration

3. **Variable Substitution Errors**
   - Ensure all variables are declared before use: `@variable = value`
   - Check variable names match exactly: `{{variable}}`

4. **API Authentication Errors**
   - Verify session data exists and matches expected structure
   - Check AuthToken header in your HTTP requests
   - Ensure session hasn't expired

5. **Data Type Mismatches**
   - Use string IDs, not ObjectId: `"_id": "string-id"`
   - Ensure date formats are ISO strings: `"{{$now}}"`
   - Check numeric fields are proper numbers, not strings

### Validation Commands

```bash
# Check MongoDB connection
mongosh "mongodb://localhost:27017/roadwarrior" --eval "db.adminCommand('ping')"

# Verify collections exist
mongosh "mongodb://localhost:27017/roadwarrior" --eval "db.getCollectionNames()"

# Check user data
mongosh "mongodb://localhost:27017/roadwarrior" --eval "db.users.find({userType: 1}).pretty()"

# Verify session data  
mongosh "mongodb://localhost:27017/roadwarrior" --eval "db.sessions.find({isActive: true}).pretty()"
```

This guide provides the complete workflow for setting up test data using the `.data` format and executing API tests manually until an automated runner is implemented. 