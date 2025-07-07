**"Find hobbies per age group"** using **`$unwind`** — this is another effective way to flatten arrays before grouping.

---

## 📝 Step 1: Insert Sample Documents (same as before)

```js
db.users.insertMany([
  { name: "Alice", age: 25, hobbies: ["reading", "traveling"] },
  { name: "Bob", age: 25, hobbies: ["gaming", "reading"] },
  { name: "Charlie", age: 30, hobbies: ["hiking", "music"] },
  { name: "David", age: 25, hobbies: ["music", "coding"] },
  { name: "Eve", age: 30, hobbies: ["traveling", "cooking"] },
  { name: "Frank", age: 35, hobbies: ["gaming", "hiking"] }
])
```

---

## ✅ Step 2: Aggregation using `$unwind`

```js
db.users.aggregate([
  { $unwind: "$hobbies" },
  {
    $group: {
      _id: "$age",
      hobbiesPerAge: { $addToSet: "$hobbies" } // collect unique hobbies
    }
  }
])
```

---

### 🔍 Explanation:

| Stage     | What it does                                                             |
| --------- | ------------------------------------------------------------------------ |
| `$unwind` | Flattens each user's `hobbies` array → one document per hobby            |
| `$group`  | Groups by `age` and uses `$addToSet` to collect **unique** hobby strings |

---

### 🧾 Sample Output:

```js
[
  {
    _id: 25,
    hobbiesPerAge: ["reading", "traveling", "gaming", "music", "coding"]
  },
  {
    _id: 30,
    hobbiesPerAge: ["hiking", "music", "traveling", "cooking"]
  },
  {
    _id: 35,
    hobbiesPerAge: ["gaming", "hiking"]
  }
]
```

---

### 💡 Bonus Tips:

* If you want **duplicates** (i.e., no deduplication), use `$push` instead of `$addToSet`.
* You can also **count hobbies per age** by adding another field:

```js
{
  $group: {
    _id: "$age",
    totalHobbies: { $sum: 1 },
    hobbies: { $addToSet: "$hobbies" }
  }
}
```



---
---
---



## ✅ 1. **With `$unwind`**

```js
db.users.aggregate([
  { $unwind: "$hobbies" },
  {
    $group: {
      _id: "$age",
      hobbiesPerAge: { $addToSet: "$hobbies" }
    }
  }
])
```

### 🔍 What happens here?

* **`$unwind`** breaks each array in `hobbies` into **multiple documents** — one per hobby.
* Then **`$group`** collects these individual hobby strings **directly** into `hobbiesPerAge` using `$addToSet`, which ensures **uniqueness**.

### ✅ Example:

If a document is:

```js
{ name: "Alice", age: 25, hobbies: ["reading", "traveling"] }
```

After `$unwind`, it becomes:

```js
{ name: "Alice", age: 25, hobbies: "reading" }
{ name: "Alice", age: 25, hobbies: "traveling" }
```

So grouping collects **individual strings**.

---

## ❌ 2. **Without `$unwind`**

```js
db.users.aggregate([
  {
    $group: {
      _id: "$age",
      hobbiesPerAge: { $push: "$hobbies" }
    }
  }
])
```

### 🔍 What happens here?

* It directly **pushes the entire `hobbies` array** from each user into the group.
* This results in a **nested array** (array of arrays).
* **No deduplication** is done.

### ❗ Output will look like:

```js
{
  _id: 25,
  hobbiesPerAge: [
    ["reading", "traveling"],
    ["gaming", "reading"],
    ["music", "coding"]
  ]
}
```

You will need extra processing (like `$reduce` + `$setUnion`) to flatten and deduplicate.

---

## 🆚 Final Comparison Table:

| Feature                   | With `$unwind`               | Without `$unwind`                        |
| ------------------------- | ---------------------------- | ---------------------------------------- |
| How hobbies are grouped   | Individually (flat values)   | As arrays (nested arrays)                |
| Duplicates removed?       | Yes (with `$addToSet`)       | No (you get duplicates unless processed) |
| Result structure          | Flat array of unique hobbies | Array of arrays                          |
| Need extra processing?    | No                           | Yes (to flatten & deduplicate)           |
| Simpler for final output? | ✅ Yes                        | ❌ No                                     |

---

### ✅ Conclusion:

* Use **`$unwind` + `$addToSet`** when you want a **clean, flat list of unique hobbies** per age group.
* Use **`$push` without `$unwind`** if you want to keep the original arrays, or plan to process them later.




---
---


## ✅ What is `$addToSet`?

**`$addToSet`** is a **MongoDB aggregation operator** that:

> **Adds a value to an array** in a group result **only if it’s not already present**.

### 🧠 Think of it as:

> `$push` ➕ **automatic deduplication**

---

## 🔧 Syntax:

```js
{ $addToSet: <expression> }
```

Used inside a `$group` stage, typically like:

```js
{
  $group: {
    _id: "$age",
    uniqueHobbies: { $addToSet: "$hobbies" }
  }
}
```

---

## 🔍 Example:

Suppose your collection:

```js
{ name: "Alice", hobbies: "reading", age: 25 }
{ name: "Bob", hobbies: "gaming", age: 25 }
{ name: "Carol", hobbies: "reading", age: 25 }
{ name: "David", hobbies: "gaming", age: 30 }
```

Now run:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$age",
      uniqueHobbies: { $addToSet: "$hobbies" }
    }
  }
])
```

### ✅ Output:

```js
[
  { _id: 25, uniqueHobbies: ["reading", "gaming"] },
  { _id: 30, uniqueHobbies: ["gaming"] }
]
```

### 🔁 If you used `$push` instead:

```js
{ _id: 25, hobbies: ["reading", "gaming", "reading"] } // duplicates!
```

---

## ✅ Summary:

| Operator    | What it does                                | Allows Duplicates? |
| ----------- | ------------------------------------------- | ------------------ |
| `$push`     | Adds value to array                         | ✅ Yes              |
| `$addToSet` | Adds value to array **only if not present** | ❌ No               |

---

## 🛠️ Use Cases:

* Collecting **unique hobbies**, tags, categories, etc.
* Avoiding duplicate values when grouping.
* Creating **deduplicated lists** of email IDs, item names, etc.
