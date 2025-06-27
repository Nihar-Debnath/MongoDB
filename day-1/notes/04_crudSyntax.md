## 🟢 **C – CREATE**

> Add new documents to a collection.

### ✅ Common Methods:

| Method               | Description                             |
| -------------------- | --------------------------------------- |
| `insertOne(doc)`     | Inserts **one** document.               |
| `insertMany([docs])` | Inserts **multiple** documents at once. |

### 🔧 Examples:

```js
db.users.insertOne({ name: "Alice", age: 25 })

db.users.insertMany([
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 35 }
])
```

---

## 🔵 **R – READ**

> Retrieve documents from a collection.

### ✅ Common Methods:

| Method                       | Description                                     |
| ---------------------------- | ----------------------------------------------- |
| `find(query)`                | Returns a **cursor** to all matching documents. |
| `findOne(query)`             | Returns the **first matching** document.        |
| `find().limit(n)`            | Limits results to `n` documents.                |
| `find().sort({field: 1/-1})` | Sorts results ascending (1) or descending (-1). |
| `find().skip(n)`             | Skips first `n` documents.                      |

### 🔧 Examples:

```js
db.users.find({ age: { $gt: 25 } }) // all users with age > 25

db.users.findOne({ name: "Bob" }) // one user named Bob

db.users.find().limit(5).sort({ age: -1 }) // top 5 oldest users
```

---

## 🟡 **U – UPDATE**

> Modify existing documents.

### ✅ Common Methods:

| Method                            | Description                         |
| --------------------------------- | ----------------------------------- |
| `updateOne(filter, update)`       | Updates the **first match**.        |
| `updateMany(filter, update)`      | Updates **all matching** documents. |
| `replaceOne(filter, replacement)` | Replaces a document **completely**. |

### ✅ Common Operators for `$set`, `$inc`, `$unset`, `$push`, etc.

### 🔧 Examples:

```js
db.users.updateOne(
  { name: "Alice" },
  { $set: { age: 28 } }
)

db.users.updateMany(
  { age: { $lt: 30 } },
  { $inc: { age: 1 } }
)

db.users.replaceOne(
  { name: "Bob" },
  { name: "Bob", city: "Delhi" }
)
```

---

## 🔴 **D – DELETE**

> Remove documents from a collection.

### ✅ Common Methods:

| Method               | Description                              |
| -------------------- | ---------------------------------------- |
| `deleteOne(filter)`  | Deletes the **first matching** document. |
| `deleteMany(filter)` | Deletes **all matching** documents.      |

### 🔧 Examples:

```js
db.users.deleteOne({ name: "Charlie" })

db.users.deleteMany({ age: { $gt: 60 } })
```

---

## 🔘 Bonus: `save()` (DEPRECATED)

* Older method: `db.collection.save(doc)`
* Automatically inserts if not exists or updates if `_id` exists.
* Not recommended in newer MongoDB versions.

