# DotData Setup Guide

This guide helps you set up your environment to work with **DotData** files for MongoDB operations and API testing.

## Prerequisites

- **MongoDB** server running locally or accessible remotely
- **Node.js** runtime for the API server
- **Code editor** (VS Code, IntelliJ, etc.) with HTTP client support

## MongoDB Setup

### 1. Install MongoDB

**macOS (using Homebrew):**
```bash
brew install mongodb-community
brew services start mongodb-community
```

**Ubuntu/Debian:**
```bash
sudo apt-get install mongodb
sudo systemctl start mongodb
sudo systemctl enable mongodb
```

**Windows:**
Download and install from [MongoDB Download Center](https://www.mongodb.com/try/download/community)

### 2. Verify MongoDB Installation

```bash
mongo --version
# Should show: MongoDB shell version v5.x or higher
```

### 3. Connect to MongoDB

```bash
mongo
# Should connect to MongoDB shell at localhost:27017
```

## DotData File Structure

Create your DotData files with the `.data` extension:

```
project/
├── data-files/
│   ├── reportingapi/
│   │   ├── get-drivers.data       # Test data setup
│   │   └── get-drivers.http       # API test requests
│   └── users/
│       ├── user-management.data   # User test data
│       └── user-tests.http        # User API tests
```

## Running DotData Files

### Option 1: Manual Execution (Current)

Since DotData is a new format, you'll need to manually convert operations to MongoDB commands:

1. **Open your DotData file** (e.g., `get-drivers.data`)
2. **Copy the operations** you want to execute
3. **Translate to MongoDB syntax** and run in MongoDB shell

**Example DotData:**
```data
INSERT users
{
  "_id": "user_001",
  "name": "John Doe", 
  "email": "john@test.com"
}
```

**MongoDB Shell equivalent:**
```javascript
use roadwarrior
db.users.insertOne({
  "_id": "user_001",
  "name": "John Doe",
  "email": "john@test.com"
})
```

### Option 2: Future - DotData Executor (Planned)

A DotData executor tool is planned for direct execution:

```bash
# Future planned syntax
dotdata-cli execute get-drivers.data
```

## API Server Setup

### 1. Install Dependencies

```bash
cd your-project-root
npm install
```

### 2. Configure Database Connection

Update your MongoDB connection in `config/mongo.js`:

```javascript
module.exports = {
  url: 'mongodb://localhost:27017/roadwarrior',
  options: {
    useNewUrlParser: true,
    useUnifiedTopology: true
  }
}
```

### 3. Start the API Server

```bash
npm start
# or
node app/core-api/server.ts
```

The server should start on `http://localhost:3000` (or your configured port).

## Running HTTP Tests

### 1. Open HTTP File

Open your `.http` file (e.g., `get-drivers.http`) in VS Code with the REST Client extension.

### 2. Execute Requests

Click the "Send Request" button above each HTTP request:

```http
### Test 1: Valid request with proper authentication
GET http://localhost:3000/api/reporting/GetDrivers?startEpoc=1609459200&endEpoc=1640995200
AuthToken: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Complete Testing Workflow

### 1. Prepare Test Data (DotData)

```data
# get-drivers.data
=== Test Data Setup ===

@TRACK_CHANGES = true
@CHANGE_TAG = "get_drivers_test"

# Insert test organization
INSERT organizations
{
  "_id": "org_test_001",
  "name": "Test Organization",
  "domain": "testorg.com"
}

# Insert test users (drivers)
INSERT_MANY users [
  {
    "_id": "driver_001",
    "orgId": "org_test_001", 
    "name": "John Driver",
    "email": "john@testorg.com",
    "UserType": 1,
    "IsActive": true,
    "IsDeleted": false
  },
  {
    "_id": "driver_002", 
    "orgId": "org_test_001",
    "name": "Jane Driver", 
    "email": "jane@testorg.com",
    "UserType": 1,
    "IsActive": true,
    "IsDeleted": false
  }
]

# Insert test session
INSERT sessions
{
  "_id": "session_test_001",
  "UserId": "driver_001",
  "OrgId": "org_test_001",
  "Role": 1,
  "UserIsActive": true,
  "SeshIsDeleted": false,
  "AuthToken": "test_token_12345"
}
```

### 2. Execute Test Data Setup

Manually convert and run the DotData operations in MongoDB shell:

```javascript
use roadwarrior

// Insert test data
db.organizations.insertOne({...})
db.users.insertMany([...])  
db.sessions.insertOne({...})
```

### 3. Run API Tests (HTTP)

```http
### Test GetDrivers API
GET http://localhost:3000/api/reporting/GetDrivers?startEpoc=1609459200&endEpoc=1640995200
AuthToken: test_token_12345

### Expected Response:
# HTTP/1.1 200 OK
# Content-Type: application/json
# 
# [
#   {
#     "_id": "driver_001",
#     "name": "John Driver", 
#     "email": "john@testorg.com"
#   },
#   {
#     "_id": "driver_002",
#     "name": "Jane Driver",
#     "email": "jane@testorg.com" 
#   }
# ]
```

### 4. Clean Up Test Data (DotData)

```data
=== Cleanup ===

# Rollback all tracked changes
ROLLBACK_CHANGES WHERE tag = "get_drivers_test"

# Verify cleanup
SHOW_CHANGES  # Should be empty
```

**Manual cleanup** (until DotData executor exists):

```javascript
// Clean up test data
db.sessions.deleteOne({"_id": "session_test_001"})
db.users.deleteMany({"orgId": "org_test_001"})
db.organizations.deleteOne({"_id": "org_test_001"})
```

## Troubleshooting

### MongoDB Connection Issues

```bash
# Check if MongoDB is running
sudo systemctl status mongodb  # Ubuntu/Debian
brew services list | grep mongodb  # macOS

# Check MongoDB logs
tail -f /var/log/mongodb/mongod.log  # Ubuntu/Debian
tail -f /usr/local/var/log/mongodb/mongo.log  # macOS
```

### API Server Issues

```bash
# Check if server is running
curl http://localhost:3000/health

# Check server logs
npm run logs
```

### DotData Syntax Issues

- Ensure proper indentation for multi-line syntax
- Check that all strings are properly quoted
- Verify MongoDB connection string in MONGO directive
- Use `#` or `//` for comments

## Editor Configuration

### VS Code Extensions

Install these extensions for the best DotData experience:

1. **REST Client** - For executing `.http` files
2. **MongoDB for VS Code** - For MongoDB database browsing
3. **YAML** - For syntax highlighting (similar to DotData)

### VS Code Settings

Add to your `settings.json`:

```json
{
  "files.associations": {
    "*.data": "yaml"
  },
  "rest-client.environmentVariables": {
    "local": {
      "baseUrl": "http://localhost:3000",
      "authToken": "your-test-token-here"
    }
  }
}
```

## Next Steps

1. **Create your first DotData file** for your API endpoint
2. **Set up corresponding HTTP test file**
3. **Follow the testing workflow** above
4. **Wait for DotData executor** for automated execution

For more advanced DotData features, see:
- `README.md` - Complete syntax guide
- `SPECS.md` - Technical specifications
- `format-examples/` - Example DotData files 