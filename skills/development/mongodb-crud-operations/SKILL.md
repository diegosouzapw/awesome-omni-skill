---
name: mongodb-crud-operations
version: "2.1.0"
description: Master MongoDB CRUD operations, document insertion, querying, updating, and deletion. Learn BSON format, ObjectId, data types, and basic operations. Use when working with documents, collections, and fundamental MongoDB operations.
sasmp_version: "1.3.0"
bonded_agent: 01-mongodb-fundamentals
bond_type: PRIMARY_BOND

# Production-Grade Skill Configuration
capabilities:
  - document-insertion
  - document-querying
  - document-updating
  - document-deletion
  - bulk-operations
  - atomic-operations

input_validation:
  required_context:
    - operation_type
    - collection_name
  optional_context:
    - document_schema
    - write_concern
    - read_preference

output_format:
  code_example: string
  explanation: string
  error_handling: string
  best_practices: array

error_handling:
  common_errors:
    - code: CRUD001
      condition: "Duplicate key error (11000)"
      recovery: "Check unique indexes, use upsert if appropriate"
    - code: CRUD002
      condition: "Document validation failed"
      recovery: "Verify document matches schema validation rules"
    - code: CRUD003
      condition: "Write concern timeout"
      recovery: "Increase wtimeout or reduce write concern level"

prerequisites:
  mongodb_version: "4.0+"
  required_knowledge:
    - mongodb-connection
    - bson-types
  driver_requirements:
    - "MongoDB native driver for your language"

testing:
  unit_test_template: |
    // Test CRUD operation
    const insertResult = await collection.insertOne(doc)
    expect(insertResult.insertedId).toBeDefined()
    const found = await collection.findOne({_id: insertResult.insertedId})
    expect(found).toMatchObject(doc)
---

# MongoDB CRUD Operations

Master fundamental MongoDB Create, Read, Update, Delete operations.

## Quick Start

### Connect to MongoDB
```javascript
const { MongoClient } = require('mongodb');

const client = new MongoClient('mongodb://localhost:27017');
await client.connect();

const db = client.db('myapp');
const users = db.collection('users');
```

### Create Documents
```javascript
// Insert one document
const result = await users.insertOne({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
  createdAt: new Date()
});
console.log('Inserted ID:', result.insertedId);

// Insert multiple documents
await users.insertMany([
  { name: 'Alice', email: 'alice@example.com' },
  { name: 'Bob', email: 'bob@example.com' }
]);
```

### Read Documents
```javascript
// Find one document
const user = await users.findOne({ email: 'john@example.com' });

// Find all documents
const allUsers = await users.find({}).toArray();

// Find with filter
const activeUsers = await users.find({ status: 'active' }).toArray();

// Find by ObjectId
const { ObjectId } = require('mongodb');
const user = await users.findOne({ _id: new ObjectId('...') });
```

### Update Documents
```javascript
// Update one document
const result = await users.updateOne(
  { email: 'john@example.com' },
  { $set: { age: 31, updatedAt: new Date() } }
);

// Update multiple documents
await users.updateMany(
  { status: 'inactive' },
  { $set: { lastNotified: new Date() } }
);

// Replace entire document
await users.replaceOne(
  { _id: userId },
  { name: 'New Name', email: 'new@example.com' }
);
```

### Delete Documents
```javascript
// Delete one document
await users.deleteOne({ email: 'john@example.com' });

// Delete multiple documents
await users.deleteMany({ status: 'deleted' });

// Delete all documents (careful!)
await users.deleteMany({});
```

## BSON Data Types

```javascript
// String
{ name: 'John' }

// Number (int32, int64, double)
{ age: 30, price: 19.99 }

// Boolean
{ isActive: true }

// Date
{ createdAt: new Date() }

// Array
{ tags: ['mongodb', 'database', 'nosql'] }

// Object (embedded document)
{ address: { city: 'New York', zip: '10001' } }

// ObjectId (default _id field)
{ _id: ObjectId('507f1f77bcf86cd799439011') }

// Null
{ description: null }

// Binary Data
{ image: Buffer.from('data') }

// Regular Expression
{ email: /.*@example\.com/ }
```

## Key Concepts

- **_id Field**: Automatically generated ObjectId, unique identifier
- **Collections**: Tables equivalent in SQL
- **Documents**: JSON-like records (up to 16MB)
- **Field Names**: Case-sensitive, cannot start with $
- **Operators**: $set, $inc, $push, $pull, $unset, etc.

## Python Example (PyMongo)

```python
from pymongo import MongoClient
from datetime import datetime

client = MongoClient('mongodb://localhost:27017')
db = client['myapp']
users = db['users']

# Insert
result = users.insert_one({
    'name': 'John',
    'email': 'john@example.com',
    'createdAt': datetime.now()
})

# Read
user = users.find_one({'email': 'john@example.com'})

# Update
users.update_one(
    {'_id': result.inserted_id},
    {'$set': {'age': 30}}
)

# Delete
users.delete_one({'_id': result.inserted_id})
```

## Best Practices

✅ Always handle errors with try-catch
✅ Use connection pooling
✅ Close connections properly
✅ Use ObjectId for _id fields
✅ Validate data before insertion
✅ Use appropriate write concerns
✅ Index frequently queried fields
✅ Plan for schema evolution
