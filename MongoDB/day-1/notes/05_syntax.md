```sql
show collections

db.students.insertMany([
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


db.students.find({}).count()

db.students.find({}).toArray()

db.students.find({}).forEach(x => printjson(x))

db.students.find().limit(10)

db.students.find({age:{$lt:17}})

db.students.find({age:{$gte:16}}).count()

db.students.find({age:{$gt:5,$lt:15}})

db.students.find({_id: ObjectId('685e933ec9736e3901748a5f')})

db.employee.find({},{name:1,_id:0})

```

## 🧪 Scenario:

You have a collection with **21 documents**, and you run:

```js
db.students.find({})
```

and then:

```js
db.students.findOne({})
```

Both are using **empty filters**, i.e., no condition — so both commands are asking:

> "Give me document(s) — I don’t care which."

But their **behavior is different**.

---

## 🟢 1. `db.students.find({})` — Returns a **Cursor**

### ✅ What Happens?

* It **doesn't return all documents at once**.
* Instead, it gives you a **cursor** — a kind of **pointer** to the result set in the database.
* The cursor **lazily loads documents** in batches (e.g., 20 at a time by default).

### 🔍 What You’ll See:

You may only see **20 documents printed**, even though there are **21 documents** in total.

> The last one won’t show until you scroll or explicitly tell the cursor to continue.

That’s why people say:

> “Use `.toArray()` or `.forEach()` to exhaust the cursor.”

---

### 🔧 How to Use the Cursor:

#### ➤ View all documents:

```js
db.students.find({}).toArray()
```

→ Returns all 21 documents as a proper JavaScript array.

#### ➤ Iterate over each document:

```js
db.students.find({}).forEach(doc => printjson(doc))
```

---

## 🔵 2. `db.students.findOne({})` — Returns a **Single Document**

### ✅ What Happens?

* This command **stops immediately** after finding the first document.
* It directly returns **one document** as a JavaScript object.

### 🧾 Example Output:

```json
{
  _id: ObjectId("..."),
  name: "Riya",
  age: 16
}
```

So even though 21 documents exist, you get **just one**.

---

## 🧠 What is a Cursor in Simple Terms?

Think of it like a **bookmark or pointer** for a list of results.

* It's not the **data itself**, but a **way to read** it step by step.
* It helps MongoDB avoid loading **all 21 documents** into memory at once.

---

## ⚠️ Why You See 20, Not 21 in Compass/shell

MongoDB shell (or Compass) by default **displays the first batch** (usually **20 documents**) from the cursor.

You can:

* **Scroll down**, and Mongo will fetch the rest.
* Use `.toArray()` to force all documents to load.
* Use `.limit(n)` to control how many you want.


---
---
---


## ✅ What is `$` in MongoDB?

In MongoDB queries, the **`$` sign** is used to represent **operators**.

> These operators are **special commands** that MongoDB understands — like comparison, logic, update, etc.

---

### 🔹 Common Categories of `$` Operators:

| Category       | Examples                | Purpose                         |
| -------------- | ----------------------- | ------------------------------- |
| **Comparison** | `$lt`, `$gt`, `$eq`     | Less than, greater than, equals |
| **Logical**    | `$and`, `$or`, `$not`   | Combine multiple conditions     |
| **Update**     | `$set`, `$inc`          | Change or add values            |
| **Array**      | `$in`, `$push`, `$size` | Work with arrays                |
| **Element**    | `$exists`, `$type`      | Check fields’ presence or type  |

---

## 🔍 Now your query:

```js
db.students.find({ age: { $lt: 17 } })
```

### 🔎 Let’s understand it piece-by-piece:

| Part                   | Meaning                             |
| ---------------------- | ----------------------------------- |
| `db.students`          | Refers to the `students` collection |
| `.find(...)`           | Used to **search** for documents    |
| `{ age: { $lt: 17 } }` | Filter condition: **age < 17**      |

### ✅ What does `$lt` mean?

`$lt` = **less than** (lt = "less than")

So the query means:

> "Find all students where the `age` field is **less than 17**."

---

### 🧾 Example Documents:

If your collection contains:

```json
{ name: "Amit", age: 16 }
{ name: "Riya", age: 17 }
{ name: "Sneha", age: 15 }
{ name: "Vikram", age: 18 }
```

Then this query:

```js
db.students.find({ age: { $lt: 17 } })
```

Will return:

```json
{ name: "Amit", age: 16 }
{ name: "Sneha", age: 15 }
```

Because only these students have age **less than 17**.

---

## ✅ Other Comparison Operators with `$`

| Operator | Meaning               | Example                       |
| -------- | --------------------- | ----------------------------- |
| `$lt`    | Less than             | `{ age: { $lt: 18 } }`        |
| `$lte`   | Less than or equal    | `{ age: { $lte: 18 } }`       |
| `$gt`    | Greater than          | `{ age: { $gt: 20 } }`        |
| `$gte`   | Greater than or equal | `{ age: { $gte: 20 } }`       |
| `$eq`    | Equal to              | `{ age: { $eq: 18 } }`        |
| `$ne`    | Not equal to          | `{ age: { $ne: 18 } }`        |
| `$in`    | Value in array        | `{ age: { $in: [15, 17] } }`  |
| `$nin`   | Value not in array    | `{ age: { $nin: [15, 17] } }` |


---
---
---

## 📘 Full Command:

```js
db.employee.find({}, { name: 1, _id: 0 })
```

This command is a **MongoDB query with projection**.

---

### 🔍 Breakdown:

| Part                         | Meaning                                                   |
| ---------------------------- | --------------------------------------------------------- |
| `db.employee`                | Use the **employee** collection                           |
| `.find(...)`                 | Find documents matching the filter                        |
| First `{}`                   | Filter — means **"no filter"**, so get **all documents**  |
| Second `{ name: 1, _id: 0 }` | **Projection** — choose what fields to show in the result |

---

### 🎯 What is Projection?

Projection = You choose **which fields** you want to **include or exclude** in the output.

---

## ✅ Explanation of `{ name: 1, _id: 0 }`

| Field     | What it does                                          |
| --------- | ----------------------------------------------------- |
| `name: 1` | ✅ Include the `name` field in the output              |
| `_id: 0`  | ❌ Exclude the `_id` field (which is shown by default) |

> **By default**, MongoDB always includes `_id`, **unless** you explicitly turn it off with `_id: 0`.

---

### 🧾 Example:

Assume your collection has this:

```json
{ _id: 1, name: "Riya", age: 16 }
{ _id: 2, name: "Amit", age: 15 }
{ _id: 3, name: "Sneha", age: 17 }
```

Your query:

```js
db.employee.find({}, { name: 1, _id: 0 })
```

Gives:

```json
{ name: "Riya" }
{ name: "Amit" }
{ name: "Sneha" }
```

> **Only the `name` field is shown**. `_id` is **hidden**, and `age` is **not selected**.

---

## 🔁 Other Examples:

### ✅ Show only age, hide \_id:

```js
db.employee.find({}, { age: 1, _id: 0 })
```

### ✅ Show name and age:

```js
db.employee.find({}, { name: 1, age: 1, _id: 0 })
```

### ❌ Mixing include and exclude ❌

This will throw an error:

```js
db.employee.find({}, { name: 1, age: 0 }) // ❌ Can't mix 1 and 0 except for _id
```

---
---
---


## You mean to say that the 0 only for the _id and not for others, others will automatically dont show unless we implicitely writes 1

## ✅ Short Answer:

Yes, **only `_id` can be excluded (0) while including others (1)**.
For other fields, **you must choose either include OR exclude** — **not both** in the same projection.

---

## ✅ Projection Rules Recap:

### ✅ Rule 1: You can either:

* ✅ Include specific fields with `1`
  **OR**
* ❌ Exclude specific fields with `0`

But **you cannot mix them** (with **one exception: `_id`**).

---

## ❗ The Exception:

You **can exclude `_id` (`_id: 0`)** even if you're including other fields.

### ✅ Allowed:

```js
db.students.find({}, { name: 1, _id: 0 })  // ✅ OK
```

### ❌ Not Allowed:

```js
db.students.find({}, { name: 1, age: 0 })  // ❌ ERROR
```

> ❗ You can't mix `name: 1` and `age: 0`. MongoDB will throw:

```
Error: Projection cannot have a mix of inclusion and exclusion.
```

---

## ✅ Default Behavior

If you don’t specify a field in the projection:

* It's **excluded by default**, **except** for `_id`, which is **included by default**.

Example:

```js
db.students.find({}, { name: 1 })  // Will include: name and _id
```

To **hide \_id**, you must do:

```js
db.students.find({}, { name: 1, _id: 0 })
```
