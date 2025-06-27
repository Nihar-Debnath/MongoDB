```js

// this will not gonna check
db.createCollection("nonfiction")

// this will gonna check
db.createCollection("nonfiction", {
  validator: {
    $jsonSchema: {
      required: ['name', 'price'],
      properties: {
        name: {
          bsonType: 'string',
          description: 'must be a string and required'
        },
        price: {
          bsonType: 'number',
          description: 'must be a number and required'
        }
      }
    }
  },
  validationAction: 'error'
})


```


This MongoDB code is used to **create a new collection** called `"nonfiction"` with **schema validation**. Schema validation ensures that every document inserted or updated in the collection must follow a specific structure.

---

### 🔍 Breakdown of the Code:

### 🔸 What It Does:

#### ✅ 1. `db.createCollection("nonfiction", { ... })`

* Creates a new collection named `nonfiction`.

#### ✅ 2. `validator: { $jsonSchema: { ... } }`

* Defines a **JSON Schema validator** that will enforce structure rules on inserted/updated documents.

#### ✅ 3. `required: ['name', 'price']`

* These fields are **mandatory** in any document — if either is missing, an error will be thrown.

#### ✅ 4. `properties: { ... }`

Defines the expected **data types** and rules for each field:

* `name`:

  * `bsonType: 'string'` → must be a **string**
  * Description for reference: `'must be a string and required'`

* `price`:

  * `bsonType: 'number'` → must be a **number**
  * Description: `'must be a number and required'`

#### ✅ 5. `validationAction: 'error'`

* If a document **violates the schema**, MongoDB will **throw an error** and **reject** the operation.

---

### 🧪 Example

#### ✅ Valid Document:

```js
db.nonfiction.insertOne({ name: "Atomic Habits", price: 499 })
```

#### ❌ Invalid Document (missing `price`):

```js
db.nonfiction.insertOne({ name: "Deep Work" })
// ❗ This will throw an error because 'price' is required
```

#### ❌ Invalid Document (wrong type):

```js
db.nonfiction.insertOne({ name: "Flow", price: "not a number" })
// ❗ Error: 'price' must be a number, not a string
```


---
---


```js
db.runCommand({
  collMod: 'nonfiction',
  validator: {
    $jsonSchema: {
      required: ['name', 'price', 'author'],
      properties: {
        name: {
          bsonType: 'string',
          description: 'must be a string and required'
        },
        price: {
          bsonType: 'number',
          description: 'must be a number and required'
        },
        author: {
          bsonType: 'array',
          description: 'must be an array of author objects and required',
          items: {
            bsonType: 'object',
            required: ['name', 'email'],
            properties: {
              name: { bsonType: 'string' },
              email: { bsonType: 'string' }
            }
          }
        }
      }
    }
  },
  validationAction: 'error'
})
```


## 🔍 What This Code Does:

### ✅ 1. `db.runCommand({ collMod: 'nonfiction', ... })`

* Modifies the **existing** collection called `"nonfiction"`.

---

### ✅ 2. `validator: { $jsonSchema: { ... } }`

* Defines the **schema rules** that every document must follow.

---

### ✅ 3. `required: ['name', 'price', 'author']`

* Every document **must** contain:

  * `name` (book title),
  * `price` (book cost),
  * `author` (array of authors)

---

### ✅ 4. `properties` (field validation):

#### 🔹 `name`:

```js
bsonType: 'string'
```

* Must be a string.
* Example: `"Atomic Habits"`

#### 🔹 `price`:

```js
bsonType: 'number'
```

* Must be a number.
* Example: `450`

#### 🔹 `author`:

```js
bsonType: 'array'
items: {
  bsonType: 'object',
  required: ['name', 'email'],
  ...
}
```

* Must be an **array of objects**
* Each object must have:

  * `name`: string
  * `email`: string

---

### ✅ 5. `validationAction: 'error'`

* If a document **doesn’t match the schema**, MongoDB will **reject it with an error**.

---

## ✅ Valid Example Document:

```js
{
  name: "The Psychology of Money",
  price: 399,
  author: [
    { name: "Morgan Housel", email: "morgan@example.com" }
  ]
}
```

---

## ❌ Invalid Document Examples:

### ❌ Missing field (`author`)

```js
{ name: "IKIGAI", price: 300 }
// ❗ Fails: 'author' is required
```

### ❌ Author not an array

```js
{
  name: "Deep Work",
  price: 400,
  author: { name: "Cal", email: "cal@example.com" }
}
// ❗ Fails: 'author' must be an array
```

### ❌ Author missing `email`

```js
{
  name: "Zero to One",
  price: 550,
  author: [{ name: "Peter Thiel" }]
}
// ❗ Fails: 'email' is required for each author
```
