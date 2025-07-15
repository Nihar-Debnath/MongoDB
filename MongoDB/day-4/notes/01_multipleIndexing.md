### 📘 Multiple Indexing in MongoDB Explained (with examples)

In **MongoDB**, **multiple indexing** refers to the ability to **create more than one index** on a collection. Each index can be on **different fields** or **combinations of fields** to improve query performance for different query patterns.

---

### 🔍 Why Use Multiple Indexes?

MongoDB uses indexes to **speed up queries**. But one index can't optimize all kinds of queries.

**Example:**

Imagine a `users` collection:

```js
{
  name: "John",
  age: 25,
  email: "john@example.com",
  city: "Kolkata"
}
```

You might have different queries like:

1. `db.users.find({ name: "John" })`
2. `db.users.find({ age: { $gt: 20 } })`
3. `db.users.find({ city: "Kolkata", age: { $lt: 30 } })`

To optimize all these queries, you may want:

* One index on `name`
* Another on `age`
* A compound index on `city` and `age`

---

### ✅ Creating Multiple Indexes

```js
// Index on name
db.users.createIndex({ name: 1 })

// Index on age
db.users.createIndex({ age: 1 })

// Compound index on city and age
db.users.createIndex({ city: 1, age: -1 })
```

You can check all indexes with:

```js
db.users.getIndexes()
```

---

### ⚠️ Important Notes:

1. **Each index adds overhead** to inserts/updates — MongoDB must update all relevant indexes.
2. More indexes = more storage.
3. MongoDB uses **only one index per query** (unless it's a [compound index](https://www.mongodb.com/docs/manual/core/index-compound/) or [index intersection](#index-intersection)).

---

### 🔀 Index Intersection

MongoDB can sometimes **combine** two indexes to answer a query (called **index intersection**), but it’s slower than a single compound index.

Example:

```js
db.users.createIndex({ name: 1 })
db.users.createIndex({ city: 1 })

// Query:
db.users.find({ name: "John", city: "Kolkata" })
```

MongoDB may **combine** the `name` and `city` indexes if there's no compound index.

---

### ✅ Best Practices

* Create **compound indexes** for frequent multi-field queries.
* Use **`explain()`** to check which index is used:

```js
db.users.find({ city: "Kolkata", age: { $lt: 30 } }).explain("executionStats")
```

* Drop unused indexes to save space:

```js
db.users.dropIndex("name_1")
```

---
---
---

## 🔍 How MongoDB Chooses an Index

When you run a query, MongoDB goes through a **query planner** phase where it:

1. **Analyzes all available indexes**
2. **Generates possible execution plans**
3. **Chooses the “best” plan** based on estimated **performance cost**

You can inspect this using:

```js
db.collection.find(query).explain("executionStats")
```

---

## ✅ Index Selection Priorities

MongoDB considers several factors in this order:

| Priority | Index Type                          | Reason                          |
| -------: | ----------------------------------- | ------------------------------- |
|      1️⃣ | **Exact match with compound index** | Most efficient                  |
|      2️⃣ | **Covered query index**             | No document fetch needed        |
|      3️⃣ | **Single field index**              | Faster than collection scan     |
|      4️⃣ | **Index intersection**              | Useful but slower than compound |
|      5️⃣ | **COLLSCAN (collection scan)**      | Last resort, slowest            |

---

### 🎯 Example:

Assume these indexes exist:

```js
db.users.createIndex({ name: 1 })
db.users.createIndex({ age: 1 })
db.users.createIndex({ name: 1, age: 1 })
```

And you run this query:

```js
db.users.find({ name: "John", age: 25 })
```

#### MongoDB Query Planner:

* Sees the **compound index** `{name: 1, age: 1}` as the best match (perfect fit).
* Ignores `{name: 1}` and `{age: 1}`.
* Picks the plan using the compound index.

You can verify it using:

```js
db.users.find({ name: "John", age: 25 }).explain("executionStats")
```

Look for:

```json
"winningPlan": {
  "stage": "IXSCAN",
  "inputStage": { ... },
  "indexName": "name_1_age_1"
}
```

---

## 🧠 What is a **Covered Query**?

If **all the fields in the query + projection** are in the index, MongoDB doesn’t need to look into the document at all.

```js
db.users.find({ name: "John" }, { name: 1, _id: 0 })
```

If there's an index on `{name: 1}`, MongoDB can return results **directly from the index** = ✅ faster.

---

## 🔁 What if Multiple Indexes Can Be Used?

If no perfect index fits the query, MongoDB may try **index intersection**:

Example:

```js
db.users.createIndex({ name: 1 })
db.users.createIndex({ age: 1 })

db.users.find({ name: "Alice", age: { $gt: 20 } })
```

MongoDB might **combine both indexes**, but this is less efficient than a **compound index**.

---

## ⚠️ When MongoDB Chooses the Wrong Index

Sometimes MongoDB may choose an index that isn't ideal. You can:

1. Use `.hint()` to **force MongoDB** to use a specific index:

   ```js
   db.users.find({ name: "John" }).hint({ name: 1 })
   ```

2. Use `.explain()` to compare different plans and index usage.

---
---
---

```js
db.users.find({ name: "Emma Jones" }).explain("allPlansExecution")
```

Given the indexes:

```js
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { name: 1, age: 1 }, name: 'name_1_age_1' },
  { v: 2, key: { name: 1 }, name: 'name_1' }
]
```

---

### 🔍 What Does the Query Look Like?

```js
db.users.find({ name: "Emma Jones" })
```

This query **only filters by the `name` field**.

---

### 🧠 How Will MongoDB Respond?

When you use:

```js
.explain("allPlansExecution")
```

MongoDB will return **all candidate query plans** (not just the winning one), including:

* The **winning plan**
* **Rejected plans**
* Stats like number of documents returned, index used, etc.

---

### 📌 Now, Which Indexes Match This Query?

Let’s break down the options:

#### ✅ Candidate Indexes:

1. **`name_1`** – *Perfect match* for `{ name: "Emma Jones" }`
2. **`name_1_age_1`** – Can still be used because of **prefix matching**. The index is sorted on `{name, age}`, and since the query filters on `name`, it can use the first part of the compound index.

#### ❌ Not useful:

* **`_id_`** – Irrelevant here; only used when querying `_id`.

---

### ⚔️ So What Will Happen?

MongoDB will:

1. Consider **both `name_1` and `name_1_age_1`** as candidates
2. Create query plans using **both indexes**
3. **Evaluate** each plan’s performance (e.g., number of documents examined, keys scanned, execution time)
4. **Select a winning plan** (usually the more efficient one — most likely `name_1`)
5. Return execution stats for **all plans**

---

### 🧪 What You Will See in `allPlansExecution` Output

Example (simplified):

```json
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "IXSCAN",
      "indexName": "name_1"
    },
    "rejectedPlans": [
      {
        "stage": "IXSCAN",
        "indexName": "name_1_age_1"
      }
    ]
  },
  "executionStats": {
    "executionStages": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",
        "indexName": "name_1"
      }
    },
    "allPlansExecution": [
      {
        "indexName": "name_1",
        "nReturned": 1,
        "executionStages": { ... }
      },
      {
        "indexName": "name_1_age_1",
        "nReturned": 1,
        "executionStages": { ... }
      }
    ]
  }
}
```

---

### ✅ Final Answer:

When you run:

```js
db.users.find({ name: "Emma Jones" }).explain("allPlansExecution")
```

MongoDB will:

* Try both the `name_1` and `name_1_age_1` indexes
* Collect stats on both
* Pick the **best performing** plan (most likely `name_1`)
* Show you performance metrics of **all** plans




---
---



```js
db.users.find({ name: "Emma Jones" }).explain("allPlansExecution")
```

---

> *“In case of multiple indexes, MongoDB checks the performance of index on a sample of documents once the query is run and sets it as Winning plan.”*

✔️ This is absolutely correct.

When **multiple indexes** can satisfy a query (like `name_1` and `name_1_age_1` in your case), MongoDB does:

* A **short trial period** called the **“plan trial”**
* Each candidate index runs on a **sample of documents**
* MongoDB **measures performance** (how many documents scanned, returned, execution time, etc.)
* The **best-performing plan** is stored as the **“winning plan”**

> *“Then for second query of similar type it doesn't race them again. It stores that winning plan in cache.”*

✔️ This is also correct.

* MongoDB stores the winning plan in a **Plan Cache**
* So the **next time you run the same or a similar query**, it will **skip the trial** and directly reuse the winning plan.
* This improves performance.

---


> **Cache is reset after:**
>
> 1. After 1000 writes
> 2. Index is reset
> 3. Mongo server is restarted
> 4. Other indexes are manipulated

✔️ Yes, the **plan cache** isn't permanent. MongoDB **invalidates it** when:

1. **1000 writes** happen in the collection (because data distribution might have changed)
2. An **index is dropped or created**
3. **Server restarts**
4. Index is **rebuilt or modified** in any way

➡️ After cache reset, the **next query will go through the race again**, and a new winning plan will be selected.

---

This explains the two options of `.explain()`:

### 1. `explain("executionStats")`

* Returns the **winning plan only**
* Shows **how many documents were scanned**, how much time it took, etc.

### 2. `explain("allPlansExecution")`

* Returns **all candidate plans** (like a race)
* Shows stats for **each** one: how many docs they scanned, returned, etc.
* Helps you **understand why** MongoDB picked a certain index
* Used for **performance tuning** or debugging

---

## 🎯 Back to Your Query

```js
db.users.find({ name: "Emma Jones" }).explain("allPlansExecution")
```

### First time:

* MongoDB will **evaluate both**:

  * `name_1`
  * `name_1_age_1`
* Compare performance
* Pick the **faster one as winning plan**
* Store in **plan cache**

### Second time:

* MongoDB **skips evaluation**
* Directly uses the **cached winning plan**

### Later:

* If you restart the server or insert 1000 new docs — the cache is **cleared**, and MongoDB will **re-race** the indexes on the next run.
