# 🧩 What is `$group` in MongoDB?

The `$group` stage in an aggregation pipeline is used to **group documents by a specific field** (or expression), and then **perform calculations** like:

* count
* sum
* average
* min / max
* collect into arrays

---

## 💡 Think of `$group` like SQL's `GROUP BY`

### SQL:

```sql
SELECT gender, COUNT(*) FROM users GROUP BY gender;
```

### MongoDB:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",
      count: { $sum: 1 }
    }
  }
])
```

Here, `_id` is the **group key**.

---

# 🔧 Syntax of `$group`

```js
{
  $group: {
    _id: <expression>,      // Grouping field
    <newField>: { <accumulator>: <expression> },
    ...
  }
}
```

---

# 🛠️ Common Accumulator Operators Used in `$group`

| Operator    | Purpose                                 | Example                             |
| ----------- | --------------------------------------- | ----------------------------------- |
| `$sum`      | Total sum or count                      | `{ $sum: "$age" }` or `{ $sum: 1 }` |
| `$avg`      | Average value                           | `{ $avg: "$score" }`                |
| `$min`      | Minimum value                           | `{ $min: "$price" }`                |
| `$max`      | Maximum value                           | `{ $max: "$rating" }`               |
| `$first`    | First document in group (based on sort) | `{ $first: "$name" }`               |
| `$last`     | Last document in group                  | `{ $last: "$name" }`                |
| `$push`     | Puts values into an array               | `{ $push: "$name" }`                |
| `$addToSet` | Like `$push`, but removes duplicates    | `{ $addToSet: "$tag" }`             |
| `$count`    | Alias for `{ $sum: 1 }`                 | (not an operator, but a shortcut)   |

---

# 📦 Real Examples from Your Dataset

### ✅ 1. Group by Gender — Count Users

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",
      totalUsers: { $sum: 1 }
    }
  }
])
```

🧠 `_id` becomes the group key (e.g., "male", "female")
🧮 `$sum: 1` counts each document

---

### ✅ 2. Group by Gender — Average Age

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",
      averageAge: { $avg: "$age" }
    }
  }
])
```

---

### ✅ 3. Group by Gender — Show All Names (using `$push`)

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",
      names: { $push: "$name" }
    }
  }
])
```

🧾 Output:

```js
[
  {
    _id: "male",
    names: ["John Doe", "Michael Johnson", "Robert Brown"]
  },
  {
    _id: "female",
    names: ["Jane Smith", "Emily Williams", "Emma Jones"]
  }
]
```

---

### ✅ 4. Group by Gender — Show Unique Names (using `$addToSet`)

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",
      uniqueNames: { $addToSet: "$name" }
    }
  }
])
```

> Same as `$push`, but **removes duplicates** if any.

---

### ✅ 5. Group All Users (No Group Key)

```js
db.users.aggregate([
  {
    $group: {
      _id: null,
      totalUsers: { $sum: 1 },
      averageAge: { $avg: "$age" },
      allNames: { $push: "$name" }
    }
  }
])
```

🧠 `_id: null` means: group all documents into one group.

---

# 🔍 Advanced `$group` Trick: Group by Expression

You can group by **a calculated field**, not just existing ones.

### Example: Group by age category

```js
db.users.aggregate([
  {
    $group: {
      _id: {
        $cond: [
          { $gte: ["$age", 40] },
          "40 and above",
          "below 40"
        ]
      },
      count: { $sum: 1 }
    }
  }
])
```

---

# 📌 Summary

| Feature             | Example                          | Notes                         |
| ------------------- | -------------------------------- | ----------------------------- |
| Count               | `{ $sum: 1 }`                    | Classic counting              |
| Sum field           | `{ $sum: "$price" }`             | Adds up a field               |
| Average             | `{ $avg: "$score" }`             | Mean value                    |
| First/Last          | `{ $first: "$name" }`            | After sorting, gets 1st/last  |
| Collect in Array    | `{ $push: "$item" }`             | All values (can have dupes)   |
| Unique Array        | `{ $addToSet: "$item" }`         | Removes duplicates            |
| Group by Expression | `$cond`, `$substr`, `$year`, etc | Group based on computed logic |
