---
layout: default
title: Mongoose Notes
---

Mongoose is an **Object Data Modeling (ODM) library for MongoDB and Node.js**.  
It provides a **schema-based solution** to model application data, offering **validation**, **type casting**, **query building**, and **middleware**.

---

## 🚀 1. Installation & Setup

### Install Mongoose
```bash
npm install mongoose
```
This adds Mongoose to your Node.js project.

### Connect to MongoDB
```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://127.0.0.1:27017/myDatabase', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})
.then(() => console.log('✅ MongoDB connected'))
.catch(err => console.error('❌ Connection error:', err));
```

- `useNewUrlParser` and `useUnifiedTopology` help handle deprecations.
- Replace `myDatabase` with your DB name.

### Connection Events
```javascript
mongoose.connection.on('connected', () => console.log('Mongoose connected'));
mongoose.connection.on('error', (err) => console.log('Mongoose connection error:', err));
mongoose.connection.on('disconnected', () => console.log('Mongoose disconnected'));
```

### Disconnect
```javascript
mongoose.disconnect();
```

---

## 🏗️ 2. Schema

A **Schema** defines the structure of documents inside a collection.

### Create Schema
```javascript
const { Schema } = mongoose;

const studentSchema = new Schema({
  name: { type: String, required: true },
  age: { type: Number, min: 18 },
  email: { type: String, unique: true, lowercase: true },
  courses: [String],
  createdAt: { type: Date, default: Date.now }
});
```

### Schema Types
| Type         | Description                         |
|--------------|--------------------------------------|
| String       | Text data                           |
| Number       | Numeric values                       |
| Date         | Date/time                            |
| Boolean      | true/false                           |
| Array        | List of values                        |
| Buffer       | Binary data                           |
| ObjectId     | MongoDB ObjectId reference            |
| Mixed        | Any type (`Schema.Types.Mixed`)       |
| Decimal128   | High-precision decimal               |
| Map          | Key-value pairs                       |

### Validation
```javascript
const userSchema = new Schema({
  username: { type: String, required: [true, 'Username is required'], minlength: 3 },
  age: { type: Number, min: [18, 'Must be at least 18'], max: 60 }
});
```

- **Custom Validator**
```javascript
email: {
  type: String,
  validate: {
    validator: v => /^\S+@\S+\.\S+$/.test(v),
    message: props => `${props.value} is not a valid email!`
  }
}
```

### Default Values
```javascript
createdAt: { type: Date, default: Date.now }
```

### Indexes
```javascript
userSchema.index({ username: 1, email: 1 }, { unique: true });
```

---

## 🏷️ 3. Models

A **Model** is a constructor compiled from a schema.  
It represents a MongoDB **collection** and provides methods to interact with it.

### Create Model
```javascript
const Student = mongoose.model('Student', studentSchema);
```

- Collection name will be pluralized (`students`).

---

## ➕ 4. CRUD Operations

### Insert Documents

#### Create & Save
```javascript
const newStudent = new Student({ name: 'Abbas', age: 25, email: 'abbas@example.com' });
await newStudent.save();
```

#### Create Multiple
```javascript
await Student.insertMany([
  { name: 'Ali', age: 22, email: 'ali@example.com' },
  { name: 'Sara', age: 24, email: 'sara@example.com' }
]);
```

---

### 🔍 Read Documents

#### Find All
```javascript
const students = await Student.find();
```

#### Find by Condition
```javascript
const student = await Student.findOne({ age: { $gte: 20 } });
```

#### Find by ID
```javascript
const student = await Student.findById('6512bd43d9caa6e02c990b0a');
```

#### Projection
```javascript
const names = await Student.find({}, 'name age -_id');
```

#### Sorting
```javascript
const sorted = await Student.find().sort({ age: -1 });
```

#### Limit & Skip
```javascript
const page = await Student.find().skip(10).limit(5);
```

#### Count Documents
```javascript
const count = await Student.countDocuments({ age: { $gte: 18 } });
```

#### Distinct Values
```javascript
const courses = await Student.distinct('courses');
```

---

### ✏️ Update Documents

#### Update One
```javascript
await Student.updateOne({ name: 'Ali' }, { $set: { age: 23 } });
```

#### Update Many
```javascript
await Student.updateMany({ age: { $lt: 20 } }, { $inc: { age: 1 } });
```

#### Find and Update
```javascript
const updated = await Student.findOneAndUpdate(
  { name: 'Ali' },
  { age: 30 },
  { new: true } // returns updated doc
);
```

#### Save Method
```javascript
const student = await Student.findById(id);
student.age = 28;
await student.save();
```

---

### ❌ Delete Documents

#### Delete One
```javascript
await Student.deleteOne({ name: 'Sara' });
```

#### Delete Many
```javascript
await Student.deleteMany({ age: { $lt: 18 } });
```

#### Find and Delete
```javascript
const removed = await Student.findOneAndDelete({ name: 'Ali' });
```

---

## 🔗 5. Relationships & Population

Mongoose supports references between documents.

### Example Schema
```javascript
const courseSchema = new Schema({ name: String, duration: Number });
const Course = mongoose.model('Course', courseSchema);

const studentSchema = new Schema({
  name: String,
  enrolledCourse: { type: Schema.Types.ObjectId, ref: 'Course' }
});
```

### Populate Example
```javascript
const result = await Student.find().populate('enrolledCourse');
```

- `populate()` replaces the ObjectId with the actual document.

---

## ⚡ 6. Middleware (Hooks)

Mongoose supports middleware to run **before** or **after** certain operations.

### Types of Middleware
| Type      | Example                  | Runs On                           |
|-----------|--------------------------|------------------------------------|
| Document  | `pre('save')`, `post('save')` | Before/after saving a doc       |
| Query     | `pre('find')`, `post('find')` | Before/after queries            |
| Aggregate | `pre('aggregate')`       | Before aggregation               |

### Example Pre-Save Hook
```javascript
studentSchema.pre('save', function(next) {
  console.log('Document is about to be saved');
  next();
});
```

### Post Hook Example
```javascript
studentSchema.post('save', function(doc) {
  console.log('Document saved:', doc);
});
```

---

## 🧩 7. Virtuals

Virtuals are document properties not stored in MongoDB.  
They are computed dynamically.

### Example
```javascript
studentSchema.virtual('fullInfo').get(function() {
  return `${this.name} (${this.age} years)`;
});
```

Usage:
```javascript
const s = await Student.findOne();
console.log(s.fullInfo);
```

---

## 🔎 8. Aggregation

Mongoose supports MongoDB’s aggregation pipeline.

### Example
```javascript
const result = await Student.aggregate([
  { $match: { age: { $gte: 20 } } },
  { $group: { _id: null, avgAge: { $avg: '$age' } } }
]);
```

You can also use `.allowDiskUse()` for large datasets.

---

## 🛠️ 9. Schema Options

When defining a schema, you can pass global options:
```javascript
const opts = { timestamps: true, versionKey: false };
const userSchema = new Schema({ name: String }, opts);
```
| Option      | Description                             |
|-------------|------------------------------------------|
| `timestamps` | Adds `createdAt` and `updatedAt` fields |
| `versionKey` | Disable `__v` field if set to false     |
| `strict`     | Enforce only defined fields             |

---

## ⚙️ 10. Lean Queries

Use `.lean()` to return plain JavaScript objects instead of Mongoose documents (faster, no getters/setters).

```javascript
const plainDocs = await Student.find().lean();
```

---

## 🧵 11. Transactions

For atomic operations across multiple documents/collections (MongoDB Replica Set required).

```javascript
const session = await mongoose.startSession();
session.startTransaction();
try {
  await Student.create([{ name: 'New Student' }], { session });
  await Course.create([{ name: 'Node.js' }], { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
}
session.endSession();
```

---

## 📈 12. Plugins

Plugins add reusable functionality to schemas.

### Example
```javascript
function myPlugin(schema, options) {
  schema.add({ createdBy: String });
}

studentSchema.plugin(myPlugin);
```

---

## 🔑 13. Best Practices

- ✅ Always define a **schema** for consistency.
- ✅ Use **validation** to enforce data integrity.
- ✅ Use **indexes** for frequently queried fields.
- ✅ Use **lean()** for read-only queries to improve performance.
- ✅ Handle **connection errors** with proper logging.

---

## ✅ Summary Table

| Operation      | Mongoose Method                     |
|----------------|---------------------------------------|
| Create         | `Model.create()`, `doc.save()`       |
| Read           | `Model.find()`, `Model.findOne()`    |
| Update         | `Model.updateOne()`, `findOneAndUpdate()` |
| Delete         | `Model.deleteOne()`, `findOneAndDelete()` |
| Count          | `Model.countDocuments()`             |
| Aggregate      | `Model.aggregate()`                  |
| Populate       | `Model.populate()`                   |

---
