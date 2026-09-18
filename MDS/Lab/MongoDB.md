# MongoDB Shell Quick Syntax Reference

A rapid, syntax-first study guide covering database/collection management, CRUD operations, projections, sorting/limiting, comparison operators, and logical operators based on Lab Sheets 1, 2, and 3.

---

## 1. Database & Collection Management (Lab 1)

### 1.1 Database Operations
```javascript
// Switch to or create a database
use mydb

// Show the currently active database
db

// List all databases on server
show dbs

// Delete currently selected database
db.dropDatabase()
```

### 1.2 Collection Operations
```javascript
// Explicitly create a collection
db.createCollection("mycollection")

// List all collections in current database
show collections

// Drop/delete a collection
db.mycollection.drop()
```

---

## 2. Insert & Save Operations (Lab 2)

### 2.1 Single Insert
```javascript
db.books.insert({
  id: 1,
  title: "Cloud computing",
  tags: ["cloud", "prog"],
  reviews: 5
})
```

### 2.2 Bulk / Multi Insert
Pass an array of document objects:
```javascript
db.books.insert([
  { id: 2, title: "IDS", tags: ["DataScience"], reviews: 3 },
  { id: 3, title: "Data Analytics", reviews: 5 },
  { id: 4, title: "Big Data Systems", reviews: 5 },
  { id: 5, title: "Big Data", reviews: 3 }
])
```

### 2.3 Save Method
Inserts a new document (or updates if an existing `_id` matches):
```javascript
db.books.save({ id: 5, title: "Big Data", reviews: 3 })
```

---

## 3. Update Operations (Lab 2)

Syntax: `db.collection.update(criteria, updateOperation)`

### 3.1 Add a New Field (`$set`)
```javascript
db.books.update(
  { id: 4 },
  { $set: { author: "Celin" } }
)
```

### 3.2 Modify an Existing Field (`$set`)
```javascript
db.books.update(
  { id: 5 },
  { $set: { title: "New Big Data" } }
)
```

### 3.3 Remove a Field (`$unset`)
```javascript
db.books.update(
  { id: 4 },
  { $unset: { author: "" } }
)
```

---

## 4. Delete / Remove Operations (Lab 2)

Syntax: `db.collection.remove(criteria, justOneOption)`

### 4.1 Delete All Matching Documents
```javascript
// Removes all documents matching criteria
db.books.remove({ id: 4 })
```

### 4.2 Delete Only the First Match (`justOne`)
```javascript
// Removes only 1 matching document even if multiple exist
db.books.remove({ reviews: 5 }, { justOne: true })
```

---

## 5. Querying, Projections, Limits & Sorting (Lab 3)

### 5.1 Basic Query & Formatted Output
```javascript
// Find all documents
db.books.find()

// Find all documents formatted as clean JSON
db.books.find().pretty()
```

### 5.2 Projection (Selecting Specific Fields)
Syntax: `db.collection.find(query, projection)`  
`1` = include field, `0` = suppress field (Note: `_id` is included by default unless set to `0`).

```javascript
// Return only 'id' (plus default '_id')
db.books.find({}, { id: 1 })

// Return only 'title' and 'reviews'
db.books.find({}, { title: 1, reviews: 1 })

// Return only 'title' without '_id'
db.books.find({}, { title: 1, _id: 0 })
```

### 5.3 Limit
Restricts count of returned documents:
```javascript
db.books.find().limit(3)
```

### 5.4 Sorting
`1` for Ascending, `-1` for Descending:
```javascript
// Ascending by reviews
db.books.find().sort({ reviews: 1 })

// Descending by reviews
db.books.find().sort({ reviews: -1 })
```

---

## 6. Relational / Comparison Operators (Lab 3)

| Operator | Meaning | Syntax Pattern |
| :--- | :--- | :--- |
| *(implicit)* | Equality (`=`) | `{ field: value }` |
| `$lt` | Less Than (`<`) | `{ field: { $lt: value } }` |
| `$lte` | Less Than or Equal (`<=`) | `{ field: { $lte: value } }` |
| `$gt` | Greater Than (`>`) | `{ field: { $gt: value } }` |
| `$gte` | Greater Than or Equal (`>=`) | `{ field: { $gte: value } }` |
| `$ne` | Not Equal (`!=`) | `{ field: { $ne: value } }` |

### Practical Examples
```javascript
// Exact match
db.books.find({ title: "IDS" }).pretty()

// Less than (< 4)
db.books.find({ reviews: { $lt: 4 } }).pretty()

// Less than or equal (<= 3)
db.books.find({ reviews: { $lte: 3 } }).pretty()

// Greater than (> 4)
db.books.find({ reviews: { $gt: 4 } }).pretty()

// Greater than or equal (>= 5)
db.books.find({ reviews: { $gte: 5 } }).pretty()

// Not equal (!= 5)
db.books.find({ reviews: { $ne: 5 } }).pretty()
```

---

## 7. Logical Operators (Lab 3)

### 7.1 Logical AND (`$and` / Implicit)
```javascript
// Explicit $and syntax
db.books.find({
  $and: [
    { title: "IDS" },
    { reviews: 3 }
  ]
}).pretty()

// Equivalent implicit comma-separated AND
db.books.find({ title: "IDS", reviews: 3 }).pretty()
```

### 7.2 Logical OR (`$or`)
Matches if any condition in the array is true:
```javascript
db.books.find({
  $or: [
    { title: "IDS" },
    { reviews: 5 }
  ]
}).pretty()
```
`