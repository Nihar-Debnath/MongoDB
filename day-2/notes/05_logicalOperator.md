# 🔍 MongoDB Logical Operators

Logical operators allow you to **combine multiple conditions** in a query. Here's a quick list:

| Operator | Meaning                                                                    |
| -------- | -------------------------------------------------------------------------- |
| `$and`   | All conditions must be true                                                |
| `$or`    | At least one condition must be true                                        |
| `$nor`   | None of the conditions must be true                                        |
| `$not`   | Negates a condition (returns documents that **don’t** match the condition) |

---

## ✅ 1. `$and` — All Conditions Must Match

### 📖 Description:

Returns documents where **all conditions** in the array are true.

### 💡 Example:

Find students whose `age` is greater than 20 **and** `marks` are more than 80:

```js
db.students.find({
  $and: [
    { age: { $gt: 20 } },
    { marks: { $gt: 80 } }
  ]
})
```

🔹 This is the same as:

```js
db.students.find({
  age: { $gt: 20 },
  marks: { $gt: 80 }
})
```

---

## ✅ 2. `$or` — At Least One Condition Must Match

### 📖 Description:

Returns documents that match **at least one** of the conditions.

### 💡 Example:

Find students whose `age` is less than 20 **or** `marks` are less than 50:

```js
db.students.find({
  $or: [
    { age: { $lt: 20 } },
    { marks: { $lt: 50 } }
  ]
})
```

---

## ✅ 3. `$nor` — None of the Conditions Should Match

### 📖 Description:

Returns documents **that do NOT match any** of the conditions.

### 💡 Example:

Find students whose age is **not less than 20** and marks are **not greater than 80**:

```js
db.students.find({
  $nor: [
    { age: { $lt: 20 } },
    { marks: { $gt: 80 } }
  ]
})
```

👉 This returns students with `age >= 20` **and** `marks <= 80`.

---

## ✅ 4. `$not` — Negates a Condition

### 📖 Description:

Returns documents that **do not match** the specified condition.

> `$not` works **inside** a condition — not at the top level like `$or`.

### 💡 Example:

Find students whose `marks` are **not greater than 80**:

```js
db.students.find({
  marks: { $not: { $gt: 80 } }
})
```

---

## 🧪 Sample Document for Practice

```json
{
  name: "Alice",
  age: 22,
  marks: 85
}
```

Use this in a `students` collection to test the above queries.

---

## 🧠 Summary Table

| Operator | Purpose                             | Example                                                    |
| -------- | ----------------------------------- | ---------------------------------------------------------- |
| `$and`   | All conditions must be true         | `{ $and: [{ age: { $gt: 20 } }, { marks: { $gt: 80 } }] }` |
| `$or`    | Any one condition must be true      | `{ $or: [{ age: { $lt: 20 } }, { marks: { $lt: 50 } }] }`  |
| `$nor`   | None of the conditions must be true | `{ $nor: [{ age: { $lt: 20 } }, { marks: { $gt: 80 } }] }` |
| `$not`   | Negates a condition                 | `{ marks: { $not: { $gt: 80 } } }`                         |
