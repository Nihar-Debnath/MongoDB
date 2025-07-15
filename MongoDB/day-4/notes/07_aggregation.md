# 🧠 Aggregation Framework

---

## 🧩 What is Aggregation in MongoDB?

> Aggregation is a **framework** that processes **multiple documents** and **transforms** them into a **summarized result** — like totals, averages, grouped reports, etc.

---

### 🧃 In Simple Words:

Aggregation = MongoDB’s version of **SQL’s `GROUP BY`, `JOIN`, `SUM()`, `AVG()`**, etc.

---

## ✅ Why is Aggregation Needed?

Because **simple queries** (`find()`, `sort()`, `limit()`) only fetch raw documents.

But real-world apps need:

| Need in Real App                              | Example Aggregation |
| --------------------------------------------- | ------------------- |
| Show **total sales** today                    | `sum(price)`        |
| Group users by **country** and **count**      | `group by country`  |
| Get **average review score** for each product | `avg(rating)`       |
| Extract **top 3 best-selling items**          | `sort + limit`      |
| Combine customer + order data (like SQL JOIN) | `$lookup`           |

---

## 📦 Aggregation Uses a **Pipeline**

MongoDB aggregation uses a **pipeline of stages** — each stage **modifies** the documents and passes them to the next.

```js
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $group: { _id: "$customerId", total: { $sum: "$price" } } },
  { $sort: { total: -1 } }
])
```

> Think of it like a **factory line**:
> documents go in → transformed at each stage → summary comes out

---

## 🔧 Basic Pipeline Stages

| Stage      | What It Does                                                 |
| ---------- | ------------------------------------------------------------ |
| `$match`   | Filter documents (like `find()`)                             |
| `$group`   | Group by field and calculate aggregates (sum, avg, count...) |
| `$project` | Pick or reshape fields (like `SELECT`)                       |
| `$sort`    | Sort by any field                                            |
| `$limit`   | Limit number of results                                      |
| `$lookup`  | JOIN with another collection                                 |
| `$unwind`  | Break apart arrays inside documents                          |

---

## 🔍 Real-World Example: Total Sales by Customer

### Data in `orders` collection:

```js
{
  _id: 1,
  customer: "Alice",
  item: "Phone",
  price: 800,
  status: "delivered"
}
{
  _id: 2,
  customer: "Bob",
  item: "TV",
  price: 1200,
  status: "delivered"
}
{
  _id: 3,
  customer: "Alice",
  item: "Laptop",
  price: 1500,
  status: "delivered"
}
```

---

### 🛠️ Aggregation to get **total spent by each customer**

```js
db.orders.aggregate([
  { $match: { status: "delivered" } }, // Only delivered orders
  {
    $group: {
      _id: "$customer",                 // Group by customer name
      totalSpent: { $sum: "$price" }   // Add up the price
    }
  }
])
```

### 🧾 Output:

```js
[
  { _id: "Alice", totalSpent: 2300 },
  { _id: "Bob", totalSpent: 1200 }
]
```

---

## 🔗 Example with JOIN using `$lookup`

Let’s say:

* You have `orders` collection
* And `customers` collection with customer info

You can combine them:

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  }
])
```

This joins data just like SQL:

```sql
SELECT * FROM orders
JOIN customers ON orders.customerId = customers._id
```

---

## 🎯 Why Aggregation is Better Than Doing in App Code

| Without Aggregation                | With Aggregation                         |
| ---------------------------------- | ---------------------------------------- |
| Fetch data in app, then process it | Let MongoDB do filtering, grouping, math |
| Slow for big datasets              | Fast, uses internal indexes and RAM      |
| More network usage                 | Less data sent to client                 |

---

## ✅ Summary: Aggregation = MongoDB's Data Engine

| What       | Why You Need It                               |
| ---------- | --------------------------------------------- |
| `$match`   | Filter data like `.find()`                    |
| `$group`   | Do math and summaries (total, avg, count)     |
| `$sort`    | Sort results                                  |
| `$lookup`  | Join collections like SQL                     |
| `$project` | Shape output (rename, hide, calculate fields) |
| `$limit`   | Limit number of results                       |
| `$unwind`  | Explode arrays                                |

---
---
---



# 📦 What is an Aggregation Pipeline?

> An **aggregation pipeline** in MongoDB is like a **conveyor belt** of operations.
> Each stage in the pipeline does **something to the documents** — filters, transforms, groups, etc.

```
Documents ---> [$match] ---> [$group] ---> [$project] ---> Result
```

Each stage takes **input documents**, **modifies or filters them**, and **passes them** to the next stage.

---

# 📌 Your Data (Sample)

```js
[
  { _id: 1, name: 'John Doe', age: 35, gender: 'male' },
  { _id: 2, name: 'Jane Smith', age: 40, gender: 'female' },
  { _id: 3, name: 'Michael Johnson', age: 45, gender: 'male' },
  { _id: 4, name: 'Emily Williams', age: 30, gender: 'female' },
  { _id: 5, name: 'Robert Brown', age: 38, gender: 'male' },
  { _id: 6, name: 'Emma Jones', age: 33, gender: 'female' }
]
```

---

## ✅ 1. Group by Gender — Count How Many

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",      // Group by gender
      totalUsers: { $sum: 1 }
    }
  }
])
```

### 🔍 Output:

```js
[
  { _id: "male", totalUsers: 3 },
  { _id: "female", totalUsers: 3 }
]
```

**Explanation:**

* `$group` takes all documents and groups them by `gender`
* `$sum: 1` adds 1 for every user in each group


### 📌 What is `_id` inside `$group` in MongoDB aggregation?

In **MongoDB’s aggregation framework**, the `_id` field inside the `$group` stage **is not the same** as the `_id` field of your original document (like `{ _id: ObjectId("...") }`).

Instead, **`_id` in `$group` defines the *key* by which documents are grouped together**.


### 💡 Example:

Suppose your `users` collection looks like this:

```js
{ _id: 1, name: "John", gender: "male" }
{ _id: 2, name: "Jane", gender: "female" }
{ _id: 3, name: "Alex", gender: "male" }
```

Now your aggregation:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",               // <- GROUP BY "gender" field
      totalUsers: { $sum: 1 }       // <- COUNT users per gender
    }
  }
])
```

**Result:**

```js
[
  { _id: "male", totalUsers: 2 },
  { _id: "female", totalUsers: 1 }
]
```

So here:

* `_id: "male"` means this group includes all documents with `gender: "male"`
* `_id` is just the **group key**, and **you can use any field(s)** (even multiple fields or expressions).


### ✅ Is it related to the `_id` of documents?

**No.** It’s **not related** to the `_id` field from the documents unless **you explicitly group by `$_id`**:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$_id",               // This now groups by each document's _id
      something: { $sum: 1 }
    }
  }
])
```

This would return one group per document — which is basically pointless, because `_id` is unique by default.

### 🔁 Summary

| `_id` in `$group`                         | Meaning                                |
| ----------------------------------------- | -------------------------------------- |
| `_id: "$gender"`                          | Group by the `gender` field            |
| `_id: "$_id"`                             | Group by the original document’s `_id` |
| `_id: { gender: "$gender", age: "$age" }` | Group by multiple fields               |


### ✅ What does the `$` sign mean in MongoDB?

The `$` **is used to reference fields** from the input documents.

So:

* `"$gender"` means: **“Use the value of the `gender` field”**
* `"$age"` means: **“Use the value of the `age` field”**
* `"$name"` → “Value of `name` field”, and so on.


### 🔍 In your example:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$gender",              // <– "$gender" references the gender field from each document
      totalUsers: { $sum: 1 }      // <– $sum is an aggregation operator
    }
  }
])
```

* Here, `"$gender"` tells MongoDB:

  > "Group all documents where the **value of `gender`** is the same."

So if two documents have `gender: "male"`, they go in the same group.


### 🔁 Contrast without `$`

If you wrote:

```js
_id: "gender"
```

Then **MongoDB would literally use the string `"gender"`** as the group key — not the value of the `gender` field.

Example output:

```js
{ _id: "gender", totalUsers: 3 }  // ← Not grouped by actual gender, just a label
```

Which is **not what you want** when grouping by data field values.


### 🔧 Summary of `$` usage:

| Expression  | Meaning                                     |
| ----------- | ------------------------------------------- |
| `"$field"`  | Value of the field named `field`            |
| `"$gender"` | Value of the `gender` field                 |
| `"$age"`    | Value of the `age` field                    |
| `"gender"`  | Just the string `"gender"`, no field access |

---

## ✅ 2. Group by Gender — Get Average Age

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

### 🔍 Output:

```js
[
  { _id: "male", averageAge: 39.33 },
  { _id: "female", averageAge: 34.33 }
]
```

**Explanation:**

* `$avg` calculates the average of `age` for each gender group

---

## ✅ 3. Show All Users Older Than 35

```js
db.users.aggregate([
  {
    $match: {
      age: { $gt: 35 } // Greater than 35
    }
  }
])
```

### 🔍 Output:

```js
[
  { _id: 2, name: 'Jane Smith', age: 40, gender: 'female' },
  { _id: 3, name: 'Michael Johnson', age: 45, gender: 'male' },
  { _id: 5, name: 'Robert Brown', age: 38, gender: 'male' }
]
```

**Explanation:**

* `$match` is like a filter (WHERE clause)

---

## ✅ 4. Sort by Age Descending

```js
db.users.aggregate([
  {
    $sort: { age: -1 } // -1 = descending
  }
])
```

**Explanation:**

* `$sort` orders the documents based on the age from highest to lowest

---

## ✅ 5. Show Only Name and Age for Females (Use `$project`)

```js
db.users.aggregate([
  { $match: { gender: "female" } },
  {
    $project: {
      _id: 0,         // Hide _id
      name: 1,
      age: 1
    }
  }
])
```

### 🔍 Output:

```js
[
  { name: 'Jane Smith', age: 40 },
  { name: 'Emily Williams', age: 30 },
  { name: 'Emma Jones', age: 33 }
]
```

**Explanation:**

* `$project` reshapes the document and only includes `name` and `age`

---

## 🧠 Summary: Key Pipeline Operators You Used

| Operator   | Purpose                             |
| ---------- | ----------------------------------- |
| `$match`   | Filter data (like SQL `WHERE`)      |
| `$group`   | Group and do math (like `GROUP BY`) |
| `$project` | Select/rename fields in output      |
| `$sort`    | Order results                       |



---
---
---



In the `names` field:

```js
names: { $push: "$name" }
```

MongoDB is **collecting (pushing) the `name` field values** of all documents in each group into an **array**.

---

### 🔍 Meaning:

For each group (based on `_id: "$age"`), it will:

* Take the value of `name` from each document in the group
* Add (`$push`) it into the `names` array for that group

---

### ✅ Example:

If the collection is:

```js
{ name: "Alice", age: 25 }
{ name: "Bob", age: 25 }
{ name: "Charlie", age: 30 }
```

The output will be:

```js
[
  { _id: 25, names: ["Alice", "Bob"] },
  { _id: 30, names: ["Charlie"] }
]
```

So `names` becomes a list of names for each `age` group.



---
---
---














### 🔍 What is `$$ROOT` in MongoDB?

`$$ROOT` is a **system variable** that represents the **entire input document** currently being processed in the aggregation pipeline.

---

### 🔍 What does this do?

```js
names: { $push: "$$ROOT" }
```

This means:

> "For each group (by `age`), push the **entire document** into the `names` array."

So instead of just pushing the `name` field (like before), you're pushing the **full original document**.

---

### ✅ Example:

Assume this collection:

```js
{ _id: 1, name: "Alice", age: 25, gender: "female" }
{ _id: 2, name: "Bob", age: 25, gender: "male" }
{ _id: 3, name: "Charlie", age: 30, gender: "male" }
```

Aggregation:

```js
db.users.aggregate([
  {
    $group: {
      _id: "$age",
      names: { $push: "$$ROOT" }
    }
  }
])
```

**Result:**

```js
[
  {
    _id: 25,
    names: [
      { _id: 1, name: "Alice", age: 25, gender: "female" },
      { _id: 2, name: "Bob", age: 25, gender: "male" }
    ]
  },
  {
    _id: 30,
    names: [
      { _id: 3, name: "Charlie", age: 30, gender: "male" }
    ]
  }
]
```

---

### 🔁 Summary:

| Expression        | Meaning                                         |
| ----------------- | ----------------------------------------------- |
| `$push: "$name"`  | Push only the `name` field value into the array |
| `$push: "$$ROOT"` | Push the **entire document** into the array     |