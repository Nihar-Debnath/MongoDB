Break down all the **arguments of `createIndex()`** in MongoDB in a **simple and practical** way.

---

## 🔧 Syntax of `createIndex()`

```js
db.collection.createIndex(indexSpec, options)
```

### ▶️ 1. **`indexSpec` (required)**

This is the **object that defines the fields** to index and their sort order.

```js
{ field1: 1, field2: -1 }
```

| Value        | Meaning                           |
| ------------ | --------------------------------- |
| `1`          | Ascending order                   |
| `-1`         | Descending order                  |
| `"text"`     | For full-text search              |
| `"2dsphere"` | For geospatial data               |
| `"hashed"`   | For hashed index (e.g., sharding) |

✅ **Examples:**

```js
{ age: 1 }                       // Single-field index
{ age: 1, name: -1 }             // Compound index
{ bio: "text" }                  // Text index
{ location: "2dsphere" }         // Geospatial
{ user_id: "hashed" }            // Hashed
```

---

### ▶️ 2. **`options` (optional)**

This is a second argument — an object where you can customize how the index behaves.

#### Common `options`:

| Option                                    | Type    | Description                                                              |
| ----------------------------------------- | ------- | ------------------------------------------------------------------------ |
| `name`                                    | String  | Custom name for the index                                                |
| `unique`                                  | Boolean | Enforces all values in the index to be unique                            |
| `sparse`                                  | Boolean | Only index documents **that have the field** (skip missing fields)       |
| `expireAfterSeconds`                      | Number  | TTL index: Automatically deletes documents after X seconds               |
| `background` (deprecated in MongoDB 4.2+) | Boolean | Build index in background (non-blocking)                                 |
| `partialFilterExpression`                 | Object  | Index only documents that match a condition (advanced filtering)         |
| `collation`                               | Object  | Case sensitivity / locale-specific sorting (useful for text comparisons) |
| `wildcardProjection`                      | Object  | For `$**` wildcard index: include/exclude fields                         |

---

### 📘 Examples of Using Options:

#### ✅ Unique Index:

```js
db.users.createIndex({ email: 1 }, { unique: true })
```

#### ✅ Sparse Index:

```js
db.users.createIndex({ phone: 1 }, { sparse: true })
```

#### ✅ TTL Index (Time-To-Live):

```js
db.logs.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

Deletes logs after 1 hour.

#### ✅ Partial Index:

```js
db.orders.createIndex(
  { status: 1 },
  { partialFilterExpression: { status: { $exists: true } } }
)
```

#### ✅ Custom Index Name:

```js
db.users.createIndex({ age: 1 }, { name: "ageIndex" })
```

#### ✅ Collation (case-insensitive index):

```js
db.users.createIndex(
  { name: 1 },
  { collation: { locale: "en", strength: 2 } }
)
```

---

### ✅ Summary

| Argument    | Purpose                                 |
| ----------- | --------------------------------------- |
| `indexSpec` | What fields to index and in what order  |
| `options`   | Extra behavior like unique, TTL, sparse |

---

### 🧠 TL;DR:

* `createIndex({ field: 1 })` → basic index
* `createIndex({ field: 1 }, { unique: true })` → adds rules
* Only index **what you query/sort/filter** — no need to index everything.

Let me know if you want a **real-world use case** for any specific type of index or options!


---
---
---

## ▶️ `options` in `createIndex()`: Fully Explained

---

### 1. 🔤 `name`

#### 📘 What it does:

Gives a **custom name** to the index.

#### ✅ Example:

```js
db.users.createIndex({ age: 1 }, { name: "AgeAscendingIndex" })
```

#### 🧠 Why use it?

* Makes it easier to **identify and drop** the index later.
* Without this, MongoDB auto-generates names like `age_1`.

---

### 2. 🛑 `unique`

#### 📘 What it does:

Ensures that **no two documents** can have the **same value** for the indexed field(s).

#### ✅ Example:

```js
db.users.createIndex({ email: 1 }, { unique: true })
```

#### 🧠 Why use it?

* Enforces **data integrity**.
* Useful for fields like `email`, `username`, or `ID number`.

#### ⚠️ Caution:

* If duplicates already exist, this will **fail**.
* Only use when **you are sure values should be unique**.

---

### 3. 🧹 `sparse`

#### 📘 What it does:

Indexes **only documents that have the field** — skips documents where the field is missing or null.

#### ✅ Example:

```js
db.users.createIndex({ phone: 1 }, { sparse: true })
```

#### 🧠 Why use it?

* Saves space.
* Avoids indexing documents with missing optional fields.
* Works well for fields that are **optional** or added later.

#### ⚠️ Caution:

* Sparse index **ignores null/missing fields**.
* Can't enforce uniqueness if some documents lack the field.

---

### 4. ⏰ `expireAfterSeconds` (TTL Index)

#### 📘 What it does:

Creates a **Time-To-Live** index — MongoDB **automatically deletes** documents after a certain time.

#### ✅ Example:

```js
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

Deletes documents **1 hour** after `createdAt`.

#### 🧠 Why use it?

* Automatically **cleans up old data**.
* Great for **logs**, **sessions**, **temporary data**, etc.

#### ⚠️ Note:

* Field must be a **`Date` object**.
* Only works on a **single field** (not compound index).


### ✅ **What this does (in plain English):**

This command **creates a TTL (Time-To-Live) index** on the `createdAt` field of the `sessions` collection.

> ⏱️ It tells MongoDB:
> “Automatically delete any document from the `sessions` collection **1 hour (3600 seconds)** after its `createdAt` timestamp.”


### 🔍 Breakdown of Each Part:

| Part                           | Meaning                                        |
| ------------------------------ | ---------------------------------------------- |
| `db.sessions`                  | The collection named `sessions`                |
| `.createIndex(...)`            | You're creating an index                       |
| `{ createdAt: 1 }`             | Index the `createdAt` field in ascending order |
| `{ expireAfterSeconds: 3600 }` | Set the TTL to **3600 seconds = 1 hour**       |


### 🧪 Example Use Case:

Let’s say you store login sessions like this:

```js
{
  userId: "u123",
  token: "abcd1234",
  createdAt: new Date()  // Current time
}
```

With this index in place:

* MongoDB will **automatically delete** this session document **1 hour after `createdAt`**.
* You don’t need to manually clean up old sessions.


### ⚠️ Important Notes:

1. 🔸 `createdAt` **must be a `Date` object** (`ISODate`) — not a string or timestamp.

   ```js
   new Date() ✅   |  "2024-07-04T12:00:00Z" ❌
   ```
2. 🔸 The deletion is **not immediate** — MongoDB’s background process checks for expired documents **every 60 seconds**, so there's a small delay.
3. 🔸 Only one TTL index is allowed per collection.


### 🧠 Summary:

This command is for **automatic expiration** of old documents, commonly used for:

* ✅ Login sessions
* ✅ Password reset tokens
* ✅ Temporary data like logs or cache

---

### ✅ **It deletes the entire document, not just the index.**

When you create a **TTL index** like this:

```js
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

MongoDB will:

* **Track the value** of `createdAt` in each document using the index
* After 3600 seconds (1 hour), it will **delete the entire document** whose `createdAt` is older than 1 hour

---

### 🔥 What it does NOT do:

* ❌ It does **not delete the `createdAt` field**
* ❌ It does **not remove the index**
* ❌ It does **not modify the document** — it **removes the whole thing**

---

### 📘 Example:

#### You insert:

```js
db.sessions.insertOne({
  userId: "u1",
  token: "abc123",
  createdAt: new Date()  // let's say at 2:00 PM
})
```

Then MongoDB will automatically:

* At **3:00 PM or slightly after**, delete the whole document

```js
// Document gone:
{ userId: "u1", token: "abc123", createdAt: ... }
```

---

### ✅ Summary:

| Action             | Result                          |
| ------------------ | ------------------------------- |
| TTL index triggers | When time exceeds given seconds |
| What gets deleted  | The **entire document**         |
| What stays         | The index itself stays          |

---
---

### 5. ⚙️ `background` (deprecated)

#### 📘 What it did:

Allowed building the index in the **background** while reads/writes continued.

#### ✅ Old Example:

```js
db.users.createIndex({ age: 1 }, { background: true })
```

#### 🧠 Why it mattered:

* Prevented blocking operations in earlier MongoDB versions.

#### ⚠️ Status:

* **Deprecated since MongoDB 4.2**
* All indexes are built **non-blocking by default** now.

---

### 6. 🎯 `partialFilterExpression`

#### 📘 What it does:

Indexes **only the documents that match a filter condition**.

#### ✅ Example:

```js
db.orders.createIndex(
  { status: 1 },
  { partialFilterExpression: { status: { $exists: true } } }
)
```

#### 🧠 Why use it?

* Reduces index size.
* Useful when you want to index **only active/important** documents.

#### Real use case:

```js
db.users.createIndex(
  { lastLogin: 1 },
  { partialFilterExpression: { isActive: true } }
)
```

👉 Only indexes users who are active.

---

### 7. 🈹 `collation`

#### 📘 What it does:

Controls **how strings are compared**, like **case sensitivity** and **language sorting**.

#### ✅ Example (case-insensitive index):

```js
db.users.createIndex(
  { name: 1 },
  { collation: { locale: "en", strength: 2 } }
)
```

#### 🧠 Why use it?

* Lets you do case-insensitive searches (`"Alice"` == `"alice"`)
* Supports **sorting by locale** (like French or Chinese rules)

#### ⚠️ Tip:

* You must use the **same collation in your queries** too!

---

### 8. 🧪 `wildcardProjection`

#### 📘 What it does:

Used with **wildcard indexes** (`$**`) to include or exclude certain fields.

#### ✅ Example:

```js
db.logs.createIndex(
  { "$**": 1 },
  { wildcardProjection: { metadata: 0 } }  // exclude metadata
)
```

#### 🧠 Why use it?

* Useful when documents have **dynamic or unknown fields**.
* Saves space by not indexing everything.

---

## ✅ Summary Table

| Option                    | Purpose                                                             |
| ------------------------- | ------------------------------------------------------------------- |
| `name`                    | Set a custom index name                                             |
| `unique`                  | Ensure values are unique                                            |
| `sparse`                  | Skip documents with missing fields                                  |
| `expireAfterSeconds`      | Auto-delete old documents                                           |
| `background`              | Build index without blocking (now default)                          |
| `partialFilterExpression` | Index only documents that match a filter                            |
| `collation`               | Case-insensitive / locale-aware sorting                             |
| `wildcardProjection`      | Control which fields to include/exclude in a wildcard (`$**`) index |
