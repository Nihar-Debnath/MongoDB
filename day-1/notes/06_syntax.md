```sql
db.employee.insertMany([
  { name: "Amit", age: 15 },
  { name: "Riya", age: 16 },
  { name: "Kunal", age: 14 },
  { name: "Sneha", age: 15 },
  { name: "Vikram", age: 17 },
  { name: "Priya", age: 16 },
  { name: "Rahul", age: 15 },
  { name: "Neha", age: 14 },
  { name: "Arjun", age: 16 },
  { name: "Isha", age: 17 },
  { name: "Rohit", age: 15 },
  { name: "Tina", age: 16 },
  { name: "Nikhil", age: 14 },
  { name: "Divya", age: 15 },
  { name: "Karan", age: 16 },
  { name: "Meena", age: 17 },
  { name: "Yash", age: 15 },
  { name: "Pooja", age: 16 }
])

db.employee.updateOne({_id: ObjectId('685e9bcbf65009299b810d89')},{$set:{age:7}})

db.employee.updateMany({age:17},{$set:{age:9}})

db.employee.updateMany({age:16},{$set:{isEligible:true}})

db.employee.updateMany({age:{$gte:9}},{$set:{isEligible:false}})
```

## 🔹 1. Insert Many Documents

```js
db.employee.insertMany([
  { name: "Amit", age: 15 },
  { name: "Riya", age: 16 },
  { name: "Kunal", age: 14 },
  { name: "Sneha", age: 15 },
  { name: "Vikram", age: 17 },
  { name: "Priya", age: 16 },
  { name: "Rahul", age: 15 },
  { name: "Neha", age: 14 },
  { name: "Arjun", age: 16 },
  { name: "Isha", age: 17 },
  { name: "Rohit", age: 15 },
  { name: "Tina", age: 16 },
  { name: "Nikhil", age: 14 },
  { name: "Divya", age: 15 },
  { name: "Karan", age: 16 },
  { name: "Meena", age: 17 },
  { name: "Yash", age: 15 },
  { name: "Pooja", age: 16 }
])
```

### ✅ What it does:

* Inserts 18 employee documents into the `employee` collection.
* Each has only `name` and `age`.

---

## 🔹 2. Update One Document by `_id`

```js
db.employee.updateOne(
  { _id: ObjectId('685e9bcbf65009299b810d89') },
  { $set: { age: 7 } }
)
```

### ✅ What it does:

* Finds **one** document by `_id`.
* Sets its `age` to `7`.

### 🔎 Output:

Only the employee with that `_id` will have their age changed to `7`.

**Before:**

```json
{ _id: ..., name: "Arjun", age: 16 }
```

**After:**

```json
{ _id: ..., name: "Arjun", age: 7 }
```

> ⚠️ Note: The name is just an example. The real document depends on that exact `_id`.

---

## 🔹 3. Update All Where Age is 17 → Set Age to 9

```js
db.employee.updateMany(
  { age: 17 },
  { $set: { age: 9 } }
)
```

### ✅ What it does:

* Finds **all employees** with `age = 17`
* Changes their `age` to `9`.

### 🔎 Before:

```json
{ name: "Vikram", age: 17 }
{ name: "Isha", age: 17 }
{ name: "Meena", age: 17 }
```

### 🔎 After:

```json
{ name: "Vikram", age: 9 }
{ name: "Isha", age: 9 }
{ name: "Meena", age: 9 }
```

So, total 3 documents get modified.

---

## 🔹 4. Update All Where Age is 16 → Set isEligible: true

```js
db.employee.updateMany(
  { age: 16 },
  { $set: { isEligible: true } }
)
```

### ✅ What it does:

* Finds all documents with `age = 16`
* Adds/updates a new field: `isEligible: true`

### 🔎 Matching Employees:

```json
{ name: "Riya", age: 16, isEligible: true }
{ name: "Priya", age: 16, isEligible: true }
{ name: "Arjun", age: 16, isEligible: true }
{ name: "Tina", age: 16, isEligible: true }
{ name: "Karan", age: 16, isEligible: true }
{ name: "Pooja", age: 16, isEligible: true }
```

✅ Total: 6 employees updated.

---

## 🔹 5. Update All with Age ≥ 9 → Set isEligible: false

```js
db.employee.updateMany(
  { age: { $gte: 9 } },
  { $set: { isEligible: false } }
)
```

### ✅ What it does:

* Finds all employees where `age` is **greater than or equal to 9**
* Sets `isEligible: false`

This will:

1. **Overwrite** `isEligible: true` set earlier for age 16.
2. Set `isEligible: false` for anyone aged 9, 14, 15, 16, etc.

### 🔎 Matching Employees:

Ages that match: 9, 14, 15, 16

### 🔎 Example Final Document:

```json
{ name: "Riya", age: 16, isEligible: false }
```

✅ All previous `isEligible: true` values for age 16 get **overwritten**.

---

## ✅ Final Notes:

### 🎯 How Fields Changed:

| Name   | Age Before | Age After | isEligible     |
| ------ | ---------- | --------- | -------------- |
| Vikram | 17         | 9         | false          |
| Isha   | 17         | 9         | false          |
| Meena  | 17         | 9         | false          |
| Riya   | 16         | 16        | true → false   |
| Priya  | 16         | 16        | true → false   |
| Tina   | 16         | 16        | true → false   |
| etc.   | …          | …         | updated if ≥ 9 |
