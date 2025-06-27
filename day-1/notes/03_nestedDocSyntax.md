```sql
use demoDb

db.students.insertOne({name:"bijan",age:44})

db.students.find()

db.students.updateOne({name:"bijan"},{$set:{ idCard:{hasPanCard:true, hasAadharCard: true}}})

db.students.findOne({name:"bijan"})

db.students.updateMany({},{$set:{ hobbies:["cricket","betting"]}})

db.students.find({hobbies:'betting'})

db.students.updateOne({name:"nihar"},{$set:{ hobbies:['coding','cooking']}})

db.students.find({'idCard.hasPanCard' : true})

db.students.find({ idCard: { hasPanCard: true, hasAadharCard:true } })

```

## 🔹 Step-by-Step Explanation

### 1. **`use demoDb`**

```js
use demoDb
```

✅ **Purpose:**
This command **switches** the current database to `demoDb`.

* If `demoDb` does not exist yet, MongoDB will **create it automatically** when you insert data.

---

### 2. **Create a Document in the `students` Collection**

✅ Use:

```js
db.students.insertOne({ name: "nihar", age: 14 })
```

---

### 3. **Insert Another Document**

```js
db.students.insertOne({ name: "bijan", age: 44 })
```

✅ **Purpose:**
Adds a new document with the fields `name` and `age` into the `students` collection.

---

### 4. **Find All Students**

```js
db.students.find()
```

✅ **Purpose:**
Fetches **all documents** from the `students` collection.

---

### 5. **Update One Document: Add a Nested Field**

```js
db.students.updateOne(
  { name: "bijan" },
  { $set: { idCard: { hasPanCard: true, hasAadharCard: true } } }
)
```

✅ **Purpose:**
Finds the document where `name` is `"bijan"` and **adds/updates** an embedded field (`idCard`) that contains two subfields:

```json
idCard: {
  hasPanCard: true,
  hasAadharCard: true
}
```

---

### 6. **Find One Document Matching a Filter**

```js
db.students.findOne({ name: "bijan" })
```

✅ **Purpose:**
Finds **just one** document from `students` where `name` is `"bijan"`.

---

### 7. **Update All Documents: Add a Common Field**

```js
db.students.updateMany({}, { $set: { hobbies: ["cricket", "betting"] } })
```

✅ **Purpose:**
Updates **all documents** (empty `{}` matches everything) by adding or replacing a `hobbies` array with:

```json
["cricket", "betting"]
```

---

### 8. **Find Documents Where Hobbies Include "betting"**

```js
db.students.find({ hobbies: "betting" })
```

✅ **Purpose:**
Finds all documents where the `hobbies` array contains `"betting"` as a value.

---

### 9. **Update Hobbies for Nihar Only**

```js
db.students.updateOne(
  { name: "nihar" },
  { $set: { hobbies: ["coding", "cooking"] } }
)
```

✅ **Purpose:**
Finds the student with name `"nihar"` and replaces their `hobbies` field with:

```json
["coding", "cooking"]
```

---

### 10. **Find Documents with Nested Field Value**

```js
db.students.find({ 'idCard.hasPanCard': true })
```

✅ **Purpose:**
Finds documents where the nested field `idCard.hasPanCard` is `true`.

---

## 🔸 What is a **Nested Document**?

A nested document is a document **inside another document**, like this:

```json
{
  name: "Alice",
  contact: {
    email: "alice@example.com",
    phone: "12345"
  }
}
```

Here, `contact` is a **nested document**.

---

## ❗ What is the **Nested Document Limit** in MongoDB?

MongoDB has the following limitations:

| Limit                 | Value               |
| --------------------- | ------------------- |
| Maximum document size | **16 MB**           |
| Maximum nesting depth | **100 levels deep** |

So, you can **nest sub-documents up to 100 levels**, like:

```json
{
  level1: {
    level2: {
      level3: {
        ...
      }
    }
  }
}
```

But practically, it's recommended to **keep nesting shallow** (2–3 levels) for performance and readability.

---
---


## 🔹 You asked:

> Why do we write
> `db.students.find({ 'idCard.hasPanCard': true })`
> and **not**
> `db.students.find({ idCard.hasPanCard: true })`?

---

## ❌ This is WRONG:

```js
db.students.find({ idCard.hasPanCard: true })  // ❌
```

### 🔴 Why is this wrong?

Because in JavaScript (and MongoDB shell is based on JavaScript):

```js
idCard.hasPanCard
```

is treated as **"read the value of hasPanCard from idCard"**.

But we are not defining any object here, so it gives an **error**:

```
SyntaxError: Unexpected token '.'
```

---

## ✅ This is CORRECT:

```js
db.students.find({ 'idCard.hasPanCard': true })  // ✅
```

### ✅ Why does this work?

In MongoDB, this **string key**:

```
'idCard.hasPanCard'
```

tells MongoDB:

👉 *“Look for documents where inside the `idCard` object, there is a field `hasPanCard` with value `true`.”*

> In simple words: this is how MongoDB lets you **search inside nested fields**.

---

## 🔍 Example Document:

Let’s say your document looks like this:

```json
{
  name: "bijan",
  idCard: {
    hasPanCard: true,
    hasAadharCard: true
  }
}
```

---

## ✅ You want to find:

> All documents where `"hasPanCard"` is `true` **inside** `idCard`.

Then the correct way is:

```js
db.students.find({ 'idCard.hasPanCard': true })
```

---

## 🔧 Bonus Tip: Exact Object Matching

If you do this:

```js
db.students.find({ idCard: { hasPanCard: true } })
```

MongoDB will **only match documents where `idCard` has exactly just**:

```json
{ hasPanCard: true }
```

It **won’t match** if there are **other fields** like `hasAadharCard`.

So always use **dot notation with quotes** for nested field matching.
