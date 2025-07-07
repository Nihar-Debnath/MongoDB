## 📝 Step 1: Insert Sample Documents

Each user has a `scores` array of 5 integers and an `age`.

```js
db.users.insertMany([
  { name: "Alice", age: 25, scores: [80, 85, 90, 75, 88] },
  { name: "Bob", age: 19, scores: [60, 65, 70, 55, 68] },
  { name: "Charlie", age: 30, scores: [90, 92, 88, 95, 94] },
  { name: "David", age: 22, scores: [70, 72, 68, 75, 74] },
  { name: "Eve", age: 18, scores: [50, 55, 60, 52, 58] }
])
```

---

## 🎯 Goal:

> Find the **average of all `scores`** for users where `age > 20`.


---
---
---


## 🔍 What is `$filter` in MongoDB?

**`$filter`** is an **aggregation operator** that:

> **Returns a subset of an array** by keeping only elements that match a condition.

It’s used inside **`$project`**, **`$addFields`**, or expressions — to filter arrays.

---

### ✅ Syntax of `$filter`:

```js
{
  $filter: {
    input: <array>,
    as: <variable_name>,
    cond: <condition involving that variable>
  }
}
```

---

### 📌 Example: Keep only scores > 80

```js
{
  $project: {
    filteredScores: {
      $filter: {
        input: "$scores",       // array field
        as: "score",            // alias for each element
        cond: { $gt: ["$$score", 80] }  // condition on each element
      }
    }
  }
}
```

If scores are `[75, 90, 60, 85]`, output will be `[90, 85]`.

---

## 🛑 Why `$filter` is NOT useful for your current problem

Your goal is:

> **Find the average of all `scores`** from users where `age > 20`

But `$filter` only works **inside a single document** — it cannot filter **documents** (users) based on their fields (`age`). That’s what `$match` is for.

---

## ✅ Correct way to solve this:

### 🧩 Step-by-step plan:

1. Filter users with `age > 20` → `$match`
2. Break the `scores` array into individual values → `$unwind`
3. Average those values → `$group` + `$avg`

---

### ✅ Final Query:

```js
db.students.aggregate([
  {
    $match: {
      age: { $gt: 20 }  // Step 1: Keep users with age > 20
    }
  },
  {
    $unwind: "$scores"  // Step 2: Flatten scores arrays
  },
  {
    $group: {
      _id: null,
      averageScore: { $avg: "$scores" }  // Step 3: Compute average
    }
  }
])
```

---

### 🧾 Example Data:

```js
{ name: "Alice", age: 25, scores: [80, 90, 85, 95, 88] }
{ name: "Bob", age: 19, scores: [70, 60, 65, 55, 68] }
{ name: "Charlie", age: 30, scores: [95, 92, 90, 94, 93] }
```

Only **Alice and Charlie** qualify (`age > 20`), so the query will:

* Combine all their `scores` (10 total values)
* Output the average

---

## 🔁 If you wanted to use `$filter`

You would use it **only to filter values inside the `scores` array**, not based on `age`. For example:

```js
{
  $project: {
    highScores: {
      $filter: {
        input: "$scores",
        as: "score",
        cond: { $gte: ["$$score", 90] }
      }
    }
  }
}
```

---

## ✅ Summary

| Concept           | Purpose                                           |
| ----------------- | ------------------------------------------------- |
| `$filter`         | Filters items **inside an array** (per document)  |
| `$match`          | Filters **documents** (e.g., users with age > 20) |
| `$unwind`         | Flattens arrays into individual elements          |
| `$group` + `$avg` | Used to calculate average over values             |
