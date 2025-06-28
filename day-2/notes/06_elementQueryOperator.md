## 🔹 What are Element Query Operators in MongoDB?

**Element query operators** let you **check the existence and data type** of a **field** in a document.

They help you answer questions like:

* Does this field **exist** in the document?
* Is the value stored in this field a **number**, **string**, **array**, etc.?

---

## 🧠 Why Are They Needed?

In MongoDB (which is schema-less by default), not all documents are required to have the same fields.

So, element operators help:

* Find documents **missing a field**
* Find documents **having a field**
* Find documents where a field is of a **specific type** (e.g., `int`, `array`, `string`)

---

## 🔧 Element Operators List (with Examples)

| Operator  | Purpose                                         |
| --------- | ----------------------------------------------- |
| `$exists` | Check if a field **exists** or not              |
| `$type`   | Check if a field is of a **specific BSON type** |

---

### ✅ 1. `$exists`

### 📖 Purpose:

Checks whether a field **exists** (`true`) or **does not exist** (`false`).

---

### 🔹 Example 1: Field **exists**

Find documents where `age` field **exists**:

```js
db.students.find({ age: { $exists: true } })
```

This will return:

```json
{ name: "Alice", age: 23 }
{ name: "Bob", age: 20 }
```

---

### 🔹 Example 2: Field **does NOT exist**

Find documents where `marks` field **does NOT exist**:

```js
db.students.find({ marks: { $exists: false } })
```

This will return documents **missing** the `marks` field.

---

### ✅ 2. `$type`

### 📖 Purpose:

Checks whether the value of a field is of a **specific BSON data type**.

---

### 📜 Common Type Codes:

| Type    | Code | Alternate Name |
| ------- | ---- | -------------- |
| Double  | 1    | `"double"`     |
| String  | 2    | `"string"`     |
| Object  | 3    | `"object"`     |
| Array   | 4    | `"array"`      |
| Boolean | 8    | `"bool"`       |
| Date    | 9    | `"date"`       |
| Null    | 10   | `"null"`       |
| Int32   | 16   | `"int"`        |
| Int64   | 18   | `"long"`       |

---

### 🔹 Example 1: Field is a string

Find documents where `name` is a string:

```js
db.students.find({ name: { $type: "string" } })
```

---

### 🔹 Example 2: Field is a number (int32)

```js
db.students.find({ age: { $type: 16 } })
```

Or:

```js
db.students.find({ age: { $type: "int" } })
```

---

### 🔹 Example 3: Field is an array

```js
db.students.find({ hobbies: { $type: "array" } })
```

---

### 🔹 Example 4: Field is of **any of multiple types**

You can pass an array of types:

```js
db.students.find({
  score: { $type: ["int", "double"] }
})
```

This matches documents where `score` is **either an int or a double**.

---

## 🧪 Sample Document

```json
{
  name: "Alice",
  age: 22,
  hobbies: ["reading", "coding"],
  active: true
}
```

You could test:

* `{ age: { $type: "int" } }`
* `{ hobbies: { $exists: true } }`
* `{ marks: { $exists: false } }`

---

## ✅ Summary Table

| Operator                   | Use Case              | Example                                   |
| -------------------------- | --------------------- | ----------------------------------------- |
| `$exists: true`            | Field exists          | `{ age: { $exists: true } }`              |
| `$exists: false`           | Field is missing      | `{ marks: { $exists: false } }`           |
| `$type: "string"`          | Field is a string     | `{ name: { $type: "string" } }`           |
| `$type: ["int", "double"]` | One of multiple types | `{ score: { $type: ["int", "double"] } }` |
