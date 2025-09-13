---
layout: default
title: MongoDB Shell Commands
---
MongoDB **shell** (`mongosh`) is an interactive JavaScript interface to MongoDB.  
You can use it to **connect, query, and manage databases** directly.

---

## 🟢 1. Starting MongoDB Shell

### Starting MongoDB

```bash
mongod
```

Starts the MongoDB server (daemon).  
Default port: **27017**.

### Connecting to Shell
```bash
mongosh
```
Connects to the MongoDB server.  
If running on another host or port:
```bash
mongosh "mongodb://localhost:27017"
```

You can also connect with authentication:
```bash
mongosh "mongodb://username:password@localhost:27017/admin"
```

---

## 🟢 2. Database Commands

### Show all Databases

```javascript
show dbs
```
Displays all databases on the server.

### Create/Select a Database
```javascript
use myDatabase
```
- Switches to `myDatabase`.  
- If it doesn’t exist, MongoDB will create it **when data is inserted**.

### Check Current Database
```javascript
db
```
Shows the name of the current database.

### Drop/Delete a Database
```javascript
db.dropDatabase()
```
Deletes the current database.

---

## 🟢 3. Collection Commands

Collections are similar to **tables** in RDBMS. Documents are like rows.

### Show Collections
```javascript
show collections
```
Lists all collections in the current database.

### Create a Collection
```javascript
db.createCollection("students")
```
Creates a new collection.

Options:
```javascript
db.createCollection("students", { capped: true, size: 10000 })
```
- `capped: true`: Creates a fixed-size collection.
- `size`: Specifies the size in bytes.

### Drop a Collection
```javascript
db.students.drop()
```
Deletes the `students` collection.

---

## 🟢 4. CRUD Operations

### ➕ Insert Documents

#### Insert One
```javascript
db.students.insertOne({
  name: "Abbas",
  age: 25,
  course: "Spring Boot"
})
```

#### Insert Many
```javascript
db.students.insertMany([
  { name: "Ali", age: 22, course: "Java" },
  { name: "Sara", age: 24, course: "Angular" }
])
```
You can also specify `ordered: false` to continue inserting if errors occur.

---

### 🔍 Read Documents

#### Find All
```javascript
db.students.find()
```

#### Pretty Print
```javascript
db.students.find().pretty()
```

#### Find with Condition
```javascript
db.students.find({ age: 22 })
```

#### Projection (Show Specific Fields)
```javascript
db.students.find({}, { name: 1, age: 1, _id: 0 })
```
- `1` = include field
- `0` = exclude field

#### Sorting
```javascript
db.students.find().sort({ age: 1 })  // Ascending
db.students.find().sort({ age: -1 }) // Descending
```

#### Limit & Skip
```javascript
db.students.find().limit(5)
db.students.find().skip(5)
db.students.find().skip(5).limit(5)
```

#### Count Documents
```javascript
db.students.countDocuments({ course: "Java" })
```

---

### ✏️ Update Documents

#### Update One
```javascript
db.students.updateOne(
  { name: "Ali" },
  { $set: { course: "Node.js" } }
)
```

#### Update Many
```javascript
db.students.updateMany(
  { course: "Java" },
  { $set: { course: "Spring Boot" } }
)
```

#### Replace Document
```javascript
db.students.replaceOne(
  { name: "Sara" },
  { name: "Sara", age: 25, course: "React" }
)
```

##### Common Update Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `$set` | Set a field value | `{ $set: { age: 30 } }` |
| `$unset` | Remove a field | `{ $unset: { course: "" } }` |
| `$inc` | Increment a value | `{ $inc: { age: 1 } }` |
| `$mul` | Multiply a value | `{ $mul: { age: 2 } }` |
| `$rename` | Rename a field | `{ $rename: { course: "subject" } }` |
| `$push` | Add to an array | `{ $push: { skills: "MongoDB" } }` |
| `$addToSet` | Add unique value to an array | `{ $addToSet: { skills: "Java" } }` |
| `$pop` | Remove first/last array element | `{ $pop: { skills: 1 } }` |
| `$pull` | Remove specific value from array | `{ $pull: { skills: "Java" } }` |

---

### ❌ Delete Documents

#### Delete One
```javascript
db.students.deleteOne({ name: "Ali" })
```

#### Delete Many
```javascript
db.students.deleteMany({ course: "Angular" })
```

#### Delete All
```javascript
db.students.deleteMany({})
```

---

## 🟢 5. Indexing

Indexes improve query performance.

### Create Index
```javascript
db.students.createIndex({ name: 1 }) // Ascending
db.students.createIndex({ age: -1 }) // Descending
```

### Compound Index
```javascript
db.students.createIndex({ name: 1, age: -1 })
```

### View Indexes
```javascript
db.students.getIndexes()
```

### Drop Index
```javascript
db.students.dropIndex("name_1")
```

---

## 🟢 6. Aggregation Framework

The aggregation framework allows complex data transformations.

### Example: Group & Count
```javascript
db.students.aggregate([
  { $group: { _id: "$course", total: { $sum: 1 } } }
])
```

### Example: Match & Project
```javascript
db.students.aggregate([
  { $match: { age: { $gt: 20 } } },
  { $project: { name: 1, course: 1, _id: 0 } }
])
```

### Common Stages
| Stage | Purpose |
|------|---------|
| `$match` | Filter documents |
| `$group` | Group and aggregate data |
| `$project` | Reshape documents (select fields) |
| `$sort` | Sort results |
| `$limit` | Limit number of documents |
| `$skip` | Skip documents |
| `$lookup` | Join with another collection |
| `$unwind` | Deconstruct arrays |

---

## 🟢 7. User & Security Commands

### Create User
```javascript
db.createUser({
  user: "admin",
  pwd: "password123",
  roles: ["readWrite", "dbAdmin"]
})
```

### Show Users
```javascript
show users
```

### Drop User
```javascript
db.dropUser("admin")
```

---

## 🟢 8. Miscellaneous Commands

| Command | Purpose |
|--------|---------|
| `db.stats()` | Show database statistics |
| `db.serverStatus()` | Show server info |
| `db.getCollectionNames()` | List collections |
| `db.collection.stats()` | Show collection stats |
| `db.collection.countDocuments()` | Count documents in a collection |
| `db.collection.distinct("field")` | Get distinct values of a field |

---

## ⚡ Quick Tips

- All commands are **JavaScript-based**.
- MongoDB is **schema-less**, but documents should be structured consistently.
- Query by ObjectId:
```javascript
db.students.find({ _id: ObjectId("6512bd43d9caa6e02c990b0a") })
```

- Use `upsert: true` in update operations to insert if the document does not exist:
```javascript
db.students.updateOne(
  { name: "New User" },
  { $set: { course: "Python" } },
  { upsert: true }
)
```

---

## ✅ Summary Table

| Operation      | Command Example |
|----------------|------------------|
| Create DB      | `use myDB` |
| Create Coll.   | `db.createCollection("users")` |
| Insert         | `db.users.insertOne({...})` |
| Read           | `db.users.find({...})` |
| Update         | `db.users.updateOne({...})` |
| Delete         | `db.users.deleteOne({...})` |
| Drop DB        | `db.dropDatabase()` |
| Drop Coll.     | `db.users.drop()` |

---

{% include_relative mongodb_shell_commands_detailed.md %}
