```sql
db.employee.insertMany([
  { name: "Amit", age: 15 },
  { name: "Riya", age: 16 },
  { name: "Kunal", age: 14 },
  { name: "Sneha", age: 15 },
  { name: "Vikram", age: 7 },
  { name: "Priya", age: 16 },
  { name: "Rahul", age: 15 },
  { name: "Neha", age: 14 },
  { name: "Arjun", age: 16 },
  { name: "Isha", age: 7 },
  { name: "Rohit", age: 15 },
  { name: "Tina", age: 16 },
  { name: "Nikhil", age: 14 },
  { name: "Divya", age: 15 },
  { name: "Karan", age: 16 },
  { name: "Meena", age: 7 },
  { name: "Yash", age: 15 },
  { name: "Pooja", age: 16 }
])

db.employee.deleteMany({age:7})

db.employee.deleteOne({name:'Rohit'})

db.employee.deleteMany({})

db.employee.deleteMany({age:{$gt:7}})

```

## In your MongoDB collection, **there are 10 documents with the same `name`, for example `"Riya"`**.

---

Now you're wondering:

### ❓ What's the difference between using:

```js
db.students.updateOne({ name: "Riya" }, { $set: { age: 99 } })
```

vs.

```js
db.students.updateMany({ name: "Riya" }, { $set: { age: 99 } })
```

---

## 🟢 `updateOne(...)` – Updates **only the first match**

### ✅ What it does:

* **Finds the first** document where `name: "Riya"`.
* **Only that one** document will be updated.
* The rest of the `"Riya"` records **stay unchanged**.

### 🧾 Example:

If you had:

```json
[
  { name: "Riya", age: 16 },
  { name: "Riya", age: 17 },
  { name: "Riya", age: 15 },
  ...
  (10 total)
]
```

After `updateOne(...)`, result:

```json
[
  { name: "Riya", age: 99 }, // ✅ only one updated
  { name: "Riya", age: 17 },
  { name: "Riya", age: 15 },
  ...
]
```

---

## 🔵 `updateMany(...)` – Updates **all matches**

### ✅ What it does:

* Finds **all documents** where `name: "Riya"`.
* Updates **all of them** with `age: 99`.

### 🧾 Example Output:

```json
[
  { name: "Riya", age: 99 },
  { name: "Riya", age: 99 },
  { name: "Riya", age: 99 },
  ...
  (10 total updated)
]
```

---
---
---

Yes! The **difference between `One` and `Many`** also applies to **`insert`** and **`delete`** in MongoDB.

Let’s break it down simply:

---

## 🟢 1. **INSERT OPERATIONS**

### ✅ `insertOne(doc)`

* Inserts **a single document**.

```js
db.students.insertOne({ name: "Riya", age: 16 })
```

➡ Only 1 record is added.

---

### ✅ `insertMany([doc1, doc2, ...])`

* Inserts **multiple documents** at once.

```js
db.students.insertMany([
  { name: "Riya", age: 16 },
  { name: "Riya", age: 17 },
  { name: "Riya", age: 18 }
])
```

➡ 3 records with the same name get inserted.

> ⚠️ Even if the data looks similar, **every insert creates a new `_id`**, so they’re all treated as different documents.

---

## 🔴 2. **DELETE OPERATIONS**

### ✅ `deleteOne(filter)`

* Deletes **only the first** document that matches the condition.

```js
db.students.deleteOne({ name: "Riya" })
```

➡ Only **1 Riya** is deleted, even if there are 10 Riyas.

---

### ✅ `deleteMany(filter)`

* Deletes **all matching** documents.

```js
db.students.deleteMany({ name: "Riya" })
```

➡ **All 10 Riyas** will be deleted.