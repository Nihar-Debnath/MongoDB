Absolutely! In MongoDB, **comparison operators** are used in **queries** to filter documents based on the values of specific fields.

---

# 🔍 All MongoDB Comparison Operators (with Examples)

Here's a list of all major comparison operators:

| Operator | Meaning                   |
| -------- | ------------------------- |
| `$eq`    | Equal to                  |
| `$ne`    | Not equal to              |
| `$gt`    | Greater than              |
| `$gte`   | Greater than or equal to  |
| `$lt`    | Less than                 |
| `$lte`   | Less than or equal to     |
| `$in`    | Value in a list/array     |
| `$nin`   | Value not in a list/array |

---

## ✅ 1. `$eq` — **Equal To**

Find documents where `age` is exactly 25.

```js
db.students.find({ age: { $eq: 25 } })
```

Shortcut (same thing):

```js
db.students.find({ age: 25 })
```

---

## ✅ 2. `$ne` — **Not Equal To**

Find students whose `age` is **not 25**.

```js
db.students.find({ age: { $ne: 25 } })
```

---

## ✅ 3. `$gt` — **Greater Than**

Find students whose `age` is **greater than 25**.

```js
db.students.find({ age: { $gt: 25 } })
```

---

## ✅ 4. `$gte` — **Greater Than or Equal To**

Find students whose `age` is **greater than or equal to 25**.

```js
db.students.find({ age: { $gte: 25 } })
```

---

## ✅ 5. `$lt` — **Less Than**

Find students whose `age` is **less than 25**.

```js
db.students.find({ age: { $lt: 25 } })
```

---

## ✅ 6. `$lte` — **Less Than or Equal To**

Find students whose `age` is **less than or equal to 25**.

```js
db.students.find({ age: { $lte: 25 } })
```

---

## ✅ 7. `$in` — **Value in a List**

Find students whose `name` is **either "Alice" or "Bob"**.

```js
db.students.find({ name: { $in: ["Alice", "Bob"] } })
```

---

## ✅ 8. `$nin` — **Value NOT in a List**

Find students whose `name` is **not "Alice" or "Bob"**.

```js
db.students.find({ name: { $nin: ["Alice", "Bob"] } })
```

---

## 💡 BONUS: Combine Multiple Conditions

Find students aged **between 20 and 30**:

```js
db.students.find({ age: { $gte: 20, $lte: 30 } })
```

---

## 🧪 Sample Document for Testing

Here’s a document you can use in a collection named `students`:

```js
{
  name: "Alice",
  age: 25,
  marks: 80
}
```

---

## 🧠 Summary Table

| Operator | Description           | Example                               |
| -------- | --------------------- | ------------------------------------- |
| `$eq`    | Equals                | `{ age: { $eq: 25 } }`                |
| `$ne`    | Not equals            | `{ age: { $ne: 25 } }`                |
| `$gt`    | Greater than          | `{ age: { $gt: 25 } }`                |
| `$gte`   | Greater than or equal | `{ age: { $gte: 25 } }`               |
| `$lt`    | Less than             | `{ age: { $lt: 25 } }`                |
| `$lte`   | Less than or equal    | `{ age: { $lte: 25 } }`               |
| `$in`    | Value in array        | `{ name: { $in: ["Alice", "Bob"] }}`  |
| `$nin`   | Value not in array    | `{ name: { $nin: ["Alice", "Bob"] }}` |
