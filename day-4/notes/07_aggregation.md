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
