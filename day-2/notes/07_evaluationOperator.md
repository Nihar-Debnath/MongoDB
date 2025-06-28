## 🔍 What Are Evaluation Operators?

**Evaluation operators** in MongoDB are used to **compare fields with expressions**, **perform pattern matching**, or **apply custom JavaScript logic** when filtering documents.

---

## 🧠 Why Are Evaluation Operators Needed?

Evaluation operators help you:

* Search using **patterns** (like names starting with "A")
* Use **regular expressions** for powerful text matching
* Apply **custom logic** with JavaScript (`$where`)
* Use **full-text search** (`$text`)

They allow **flexible and complex querying** in ways that basic comparison operators can't.

---

## 🔧 List of MongoDB Evaluation Operators

| Operator | Description                                                           |
| -------- | --------------------------------------------------------------------- |
| `$regex` | Match string using **regular expressions**                            |
| `$expr`  | Compare **fields or expressions** together                            |
| `$mod`   | Check if field value divided by a number has a specific **remainder** |
| `$text`  | Perform **full-text search**                                          |
| `$where` | Run **custom JavaScript expression**                                  |

---

## ✅ 1. `$regex` — Regular Expression Match

### 📖 Find documents where `name` starts with "A":

```js
db.users.find({ name: { $regex: /^A/ } })
```

* `^A` means: starts with "A"

---

## ✅ 2. `$expr` — Use Expressions in Query

### 📖 Find documents where `score` is greater than `passingScore`:

```js
db.results.find({
  $expr: { $gt: ["$score", "$passingScore"] }
})
```

* `$expr` lets you **compare fields** inside the same document.

---

## ✅ 3. `$mod` — Modulo Operator

### 📖 Find documents where `age` is an even number:

```js
db.users.find({ age: { $mod: [2, 0] } })
```

* Syntax: `$mod: [divisor, remainder]`

---

## ✅ 4. `$text` — Full-Text Search

### 📖 Find documents matching the word "pizza":

```js
db.recipes.find({ $text: { $search: "pizza" } })
```

> 🛠 Requires a **text index** on the fields you want to search.

---

## ✅ 5. `$where` — Custom JavaScript Logic

### 📖 Find users whose age is greater than their marks:

```js
db.users.find({
  $where: function () {
    return this.age > this.marks;
  }
})
```

> ⚠️ `$where` is **powerful but slow** and should be used **sparingly**.

---

## 🧪 Sample Document

```json
{
  name: "Alice",
  age: 22,
  score: 85,
  passingScore: 60
}
```

Use this in `users` or `results` collections to test the examples above.

---

## ✅ Summary Table

| Operator | Use Case                             | Example                                          |
| -------- | ------------------------------------ | ------------------------------------------------ |
| `$regex` | Pattern matching                     | `{ name: { $regex: /^A/ } }`                     |
| `$expr`  | Compare fields in same document      | `{ $expr: { $gt: ["$score", "$passingScore"] }}` |
| `$mod`   | Find numbers with specific remainder | `{ age: { $mod: [2, 0] } }`                      |
| `$text`  | Full-text search in string fields    | `{ $text: { $search: "pizza" } }`                |
| `$where` | Custom logic in JS                   | `{ $where: "this.age > this.marks" }`            |



---
---



## ✅ `$jsonSchema` — Validate Documents Using JSON Schema

### 📖 What it does:

Allows you to **validate documents** against a defined **JSON Schema**.
It is mainly used in **schema validation** for collections.

---

### 💡 Why It's Needed:

MongoDB is schema-less by default, but `$jsonSchema` helps:

* **Enforce structure** in collections
* **Validate data types**, required fields, formats, etc.
* Prevent invalid or incomplete documents from being inserted

---

### 🧪 Example:

```js
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "age"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        age: {
          bsonType: "int",
          minimum: 0,
          description: "must be a non-negative integer"
        }
      }
    }
  }
})
```

### 🔎 Explanation:

* Requires `name` and `age`
* `name` must be a string
* `age` must be a non-negative integer

---

### ✅ Query Example Using `$jsonSchema` (from `find()`):

```js
db.users.find({
  $jsonSchema: {
    required: ["name"],
    properties: {
      name: { bsonType: "string" }
    }
  }
})
```

> 🔧 Use this to **search** documents that match a schema — not just validate.

---

## ✅ Updated Full List of Evaluation Operators

| Operator      | Description                                  |
| ------------- | -------------------------------------------- |
| `$regex`      | Pattern matching using regex                 |
| `$expr`       | Compare fields or use expressions            |
| `$mod`        | Match modulo result                          |
| `$text`       | Perform full-text search                     |
| `$where`      | Use custom JavaScript condition              |
| `$jsonSchema` | Validate documents against JSON schema rules |



---
---


## 🧩 Sample Data for Context

Assume we have a `products` collection like this:

```json
{
  name: "Laptop",
  category: "Electronics",
  price: 999.99,
  stock: 25,
  tags: ["gadget", "tech", "computer"],
  description: "High performance laptop",
  published: true,
  launchDate: ISODate("2024-11-01T00:00:00Z"),
  rating: 4.5
}
```

---

## ✅ 1. `$regex` — Pattern Matching

### 🔍 Match products that **start with "Lap"**

```js
db.products.find({ name: { $regex: /^Lap/ } })
```

### 🔍 Match products that **end with "top"**

```js
db.products.find({ name: { $regex: /top$/ } })
```

### 🔍 Match products whose **description contains "performance" (case-insensitive)**

```js
db.products.find({ description: { $regex: /performance/i } })
```

### 🔍 Match products with "tech" in **tags** array

```js
db.products.find({ tags: { $regex: /tech/i } })
```

---

## ✅ 2. `$expr` — Compare Fields in a Document

### 🔍 Find products where `stock` is **greater than** `price` (strange, but shows comparison)

```js
db.products.find({
  $expr: { $gt: ["$stock", "$price"] }
})
```

### 🔍 Find products where `rating` is **less than or equal to** 5

```js
db.products.find({
  $expr: { $lte: ["$rating", 5] }
})
```

### 🔍 Find products with `stock` > 0 **and** `published = true` (via \$expr + \$and)

```js
db.products.find({
  $expr: {
    $and: [
      { $gt: ["$stock", 0] },
      { $eq: ["$published", true] }
    ]
  }
})
```

---

## ✅ 3. `$mod` — Modulo (Remainder Match)

### 🔍 Find products where `stock` is **even**:

```js
db.products.find({
  stock: { $mod: [2, 0] }
})
```

### 🔍 Find products where `price` when divided by 100 leaves a **remainder of 99.99**

```js
db.products.find({
  price: { $mod: [100, 99.99] }
})
```

> 🧠 Rare, but can be useful for discounts, stock grouping, etc.

---

## ✅ 4. `$text` — Full-Text Search

### ⚠️ Requires a Text Index First:

```js
db.products.createIndex({ name: "text", description: "text" })
```

---

### 🔍 Search for products that mention **“laptop” or “performance”**

```js
db.products.find({
  $text: { $search: "laptop performance" }
})
```

### 🔍 Search for products that match **exact phrase “high performance”**

```js
db.products.find({
  $text: { $search: "\"high performance\"" }
})
```

### 🔍 Rank results using score:

```js
db.products.find(
  { $text: { $search: "laptop" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })
```

---

## ✅ 5. `$where` — Custom JavaScript Logic

### 🔍 Find products where `price` is **more than double** the `stock`

```js
db.products.find({
  $where: function () {
    return this.price > this.stock * 2;
  }
})
```

### 🔍 Find products where `tags` contains **more than 2 values**

```js
db.products.find({
  $where: "this.tags.length > 2"
})
```

> ⚠️ `$where` is powerful but **slow** and should be **avoided for large data**.

---

## ✅ 6. `$jsonSchema` — Schema Validation

You can use it in two ways:

### ✅ A. For Creating Collection Schema:

```js
db.createCollection("products", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "price", "stock"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string"
        },
        price: {
          bsonType: "double",
          minimum: 0,
          description: "must be a positive number"
        },
        stock: {
          bsonType: "int",
          minimum: 0,
          description: "must be a positive integer"
        },
        tags: {
          bsonType: "array"
        }
      }
    }
  }
})
```

---

### ✅ B. Use `$jsonSchema` in a Query

Find documents that match schema (like validation query):

```js
db.products.find({
  $jsonSchema: {
    required: ["name", "price"],
    properties: {
      price: { bsonType: "double" }
    }
  }
})
```

---

## 🧠 Summary Table

| Operator      | Used For                    | Example                              |
| ------------- | --------------------------- | ------------------------------------ |
| `$regex`      | Pattern matching in strings | `name: { $regex: /^Lap/ }`           |
| `$expr`       | Compare fields              | `$gt: ["$price", "$stock"]`          |
| `$mod`        | Remainder matching          | `stock: { $mod: [2, 0] }`            |
| `$text`       | Full-text search            | `{ $text: { $search: "laptop" } }`   |
| `$where`      | Custom JS logic             | `{ $where: "this.tags.length > 2" }` |
| `$jsonSchema` | Schema validation           | See above                            |
