```js
db.uesrs.insertMany([ 
    { "_id": 1, "name": "John Doe", "age": 35, "gender": "male" },
    { "_id": 2, "name": "Jane Smith", "age": 40, "gender": "female" },
    { "_id": 3, "name": "Michael Johnson", "age": 45, "gender": "male" },
    { "_id": 4, "name": "Emily Williams", "age": 30, "gender": "female" },
    { "_id": 5, "name": "Robert Brown", "age": 38, "gender": "male" },
    { "_id": 6, "name": "Emma Jones", "age": 33, "gender": "female" },
    { "_id": 7, "name": "William Davis", "age": 37, "gender": "male" },
    { "_id": 8, "name": "Olivia Miller", "age": 41, "gender": "female" },
    { "_id": 9, "name": "David Wilson", "age": 36, "gender": "male" },
    { "_id": 10, "name": "Sophia Moore", "age": 32, "gender": "female" },
    { "_id": 11, "name": "Richard Taylor", "age": 39, "gender": "male" },
    { "_id": 12, "name": "Isabella Anderson", "age": 43, "gender": "female" },
    { "_id": 13, "name": "Joseph Thomas", "age": 34, "gender": "male" },
    { "_id": 14, "name": "Mia Jackson", "age": 42, "gender": "female" },
    { "_id": 15, "name": "Charles White", "age": 36, "gender": "male" },
    { "_id": 16, "name": "Abigail Harris", "age": 31, "gender": "female" },
    { "_id": 17, "name": "Daniel Martin", "age": 44, "gender": "male" },
    { "_id": 18, "name": "Evelyn Thompson", "age": 38, "gender": "female" },
    { "_id": 19, "name": "Matthew Garcia", "age": 37, "gender": "male" },
    { "_id": 20, "name": "Sofia Martinez", "age": 35, "gender": "female" },
    { "_id": 21, "name": "Andrew Robinson", "age": 40, "gender": "male" },
    { "_id": 22, "name": "Grace Clark", "age": 33, "gender": "female" },
    { "_id": 23, "name": "Joshua Rodriguez", "age": 39, "gender": "male" },
    { "_id": 24, "name": "Avery Lewis", "age": 42, "gender": "female" },
    { "_id": 25, "name": "Christopher Lee", "age": 37, "gender": "male" },
    { "_id": 26, "name": "Chloe Walker", "age": 31, "gender": "female" },
    { "_id": 27, "name": "Kevin Hall", "age": 44, "gender": "male" },
    { "_id": 28, "name": "Zoey Allen", "age": 38, "gender": "female" },
    { "_id": 29, "name": "Brian Young", "age": 36, "gender": "male" },
    { "_id": 30, "name": "Harper King", "age": 34, "gender": "female" }
])

db.users.find({age:{$lte:30}}).explain()

```

Great! Let's walk through what happens when you run:

```js
db.users.find({ age: { $lte: 30 } }).explain()
```

Assuming:

* You inserted the 30 users as shown.
* **You did NOT create an index** on the `age` field.

---

### 🔍 What `.explain()` Does:

The `.explain()` method shows **how MongoDB will execute your query**. It tells you whether it will use:

* a **COLLSCAN** (collection scan)
* an **IXSCAN** (index scan)
* and other details like execution time and number of documents examined.

---

### 🧪 In Your Case:

#### Query:

```js
db.users.find({ age: { $lte: 30 } }).explain()
```

#### MongoDB Behavior (since no index on `age`):

MongoDB will perform a **COLLSCAN**, which means:

* It will **go through all 30 documents** in the `users` collection.
* It will check each document's `age` field.
* It will return only those where `age <= 30`.

---

### 📄 Expected `.explain()` Output (simplified):

```json
{
  "queryPlanner": {
    "plannerVersion": 1,
    "namespace": "yourdb.users",
    "indexFilterSet": false,
    "parsedQuery": {
      "age": { "$lte": 30 }
    },
    "winningPlan": {
      "stage": "COLLSCAN",   // 👈 FULL scan of collection
      "filter": { "age": { "$lte": 30 } },
      "direction": "forward"
    },
    ...
  },
  "executionStats": {
    "executionSuccess": true,
    "nReturned": X,                   // Number of matching documents (probably a few)
    "executionTimeMillis": Y,         // How long the query took
    "totalDocsExamined": 30           // All documents were scanned
  }
}
```

---

### ⚠️ Problem:

* **Performance is bad** for large collections because it has to check **every document**.

---

### ✅ Optimization Tip:

Create an index on `age`:

```js
db.users.createIndex({ age: 1 })
```

Then run the same `.find()` again. Now `.explain()` will show:

```json
"stage": "IXSCAN"  // ✅ Using index — much faster
```
---

When you run:

```js
db.users.find({ age: { $lte: 30 } }).explain("executionStats")
```

You're telling MongoDB:

> “Run the query and **give me detailed execution statistics**, not just the plan.”

---

### ✅ So What Will It Do?

It will:

1. Actually **execute** the query (not just plan it).
2. Return stats like:

   * 🧾 How many documents were **examined**
   * 📦 How many documents were **returned**
   * ⏱️ How much **time** it took
   * 🔍 Whether it used **COLLSCAN** or **IXSCAN**

---

### 🧪 Example Output (Simplified):

```json
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "COLLSCAN",   // Because no index
      "filter": { "age": { "$lte": 30 } },
      ...
    }
  },
  "executionStats": {
    "nReturned": 2,               // Number of matched documents
    "executionTimeMillis": 1,     // Time taken to run
    "totalKeysExamined": 0,       // 0 because no index
    "totalDocsExamined": 30       // Scanned all documents in collection
  }
}
```

---

### 📌 Meaning of Each Field:

| Field                 | Meaning                                                     |
| --------------------- | ----------------------------------------------------------- |
| `nReturned`           | How many documents matched the query                        |
| `executionTimeMillis` | How long the query took to run                              |
| `totalDocsExamined`   | Number of documents MongoDB looked at                       |
| `totalKeysExamined`   | Number of index keys scanned (will be `0` without an index) |
| `stage`               | Whether it used `"COLLSCAN"` or `"IXSCAN"` (index scan)     |

---

### 🧠 Why Use `"executionStats"`?

To:

* Measure **real performance**
* Check **how many documents** were scanned vs. returned
* Understand **if indexes are being used properly**

---

### ✅ Pro Tip:

If `totalDocsExamined` is much larger than `nReturned`, it means:

> You're **scanning too much** — maybe you need to add an **index**.



---
---
---


### ✅ 1. `db.users.createIndex({ age: 1 })`

This **creates an index** on the `age` field in **ascending order**.

#### 🔍 What it does:

* Builds a **B-tree** index on `age`.
* Speeds up queries that:

  * Use filters like `{ age: 25 }`, `{ age: { $lt: 40 } }`, etc.
  * Use sorting like `.sort({ age: 1 })`
* MongoDB now uses **IXSCAN** instead of **COLLSCAN** for queries on `age`.

```js
db.users.createIndex({ age: 1 })
```

the part `{ age: 1 }` means:

### 🔹 Create an **index** on the `age` field in **ascending order**.


### 🧠 Breakdown:

| Syntax | Meaning                           |
| ------ | --------------------------------- |
| `age`  | The field you want to index       |
| `1`    | Ascending order (small to large)  |
| `-1`   | Descending order (large to small) |


### 📘 Example:

```js
db.users.createIndex({ age: 1 })   // ascending
db.users.createIndex({ age: -1 })  // descending
```

Both are valid, and MongoDB will use them for filtering and sorting.


### 🔍 Use Cases:

| Query                   | Index to Use               |
| ----------------------- | -------------------------- |
| `{ age: { $lte: 30 } }` | `{ age: 1 }` ✅             |
| `{ age: { $gte: 40 } }` | `{ age: 1 }` ✅             |
| `.sort({ age: 1 })`     | `{ age: 1 }` ✅             |
| `.sort({ age: -1 })`    | `{ age: -1 }` is better ⚠️ |

So, the `1` or `-1` defines the **sort direction** of the index — it doesn't affect filtering but **matters for sorting**.

---

### ✅ 2. `db.users.getIndexes()`

This command **lists all indexes** that exist on the `users` collection.

#### 🔍 What it shows:

* `_id_` index (always there by default)
* Any other indexes you created, like `{ age: 1 }`

#### Example output:

```json
[
  { "v": 2, "key": { "_id": 1 }, "name": "_id_" },
  { "v": 2, "key": { "age": 1 }, "name": "age_1" }
]
```

---

### 📌 Summary

| Command                            | What It Does                        |
| ---------------------------------- | ----------------------------------- |
| `db.users.createIndex({ age: 1 })` | Creates index on `age` (ascending)  |
| `db.users.getIndexes()`            | Shows all indexes in the collection |

Now that you've created the index, re-run:

```js
db.users.find({ age: { $lte: 30 } }).explain("executionStats")
```

You should now see:

* `"stage": "IXSCAN"` ✅
* `totalDocsExamined` much smaller than before!


---
---
---


Now that you’ve run:

```js
db.users.createIndex({ age: 1 })
```

and then:

```js
db.users.find({ age: { $lte: 30 } }).explain("executionStats")
```

---

### 🔍 What Will Happen Now?

MongoDB will **use the index** you just created on `age` — instead of scanning every document.

So:

* The query will use **IXSCAN** (index scan), not **COLLSCAN**
* Fewer documents will be examined
* The query will be **faster and more efficient**

---

### ✅ Expected Output Highlights (simplified):

```json
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",
      "inputStage": {
        "stage": "IXSCAN",     // ✅ Index scan used now
        "keyPattern": { "age": 1 },
        "indexName": "age_1",
        "direction": "forward"
      }
    }
  },
  "executionStats": {
    "nReturned": 2,               // e.g., 2 documents where age <= 30
    "executionTimeMillis": 1,     // ✅ Faster
    "totalDocsExamined": 2,       // ✅ Only 2 docs actually fetched
    "totalKeysExamined": 2        // ✅ Only 2 index entries scanned
  }
}
```

---

### 📌 Comparison Before and After Index

| Feature               | Before (No Index) | After (`age` Index) |
| --------------------- | ----------------- | ------------------- |
| `stage`               | COLLSCAN          | IXSCAN              |
| `totalDocsExamined`   | 30                | 2 (only matching)   |
| `executionTimeMillis` | Higher            | Lower (faster)      |

---

### 💡 Conclusion:

By adding the index, MongoDB now:

* **Uses the index**
* **Skips unrelated documents**
* **Executes the query faster**


---
---
---



### ❌ **When You Do NOT Need Indexing in MongoDB**

---

### 1. **Small Collections**

* If your collection has **a small number of documents** (e.g., a few hundred or less), MongoDB can scan it quickly.
* Indexing adds overhead for no real performance gain.

> 🧠 MongoDB can perform COLLSCAN efficiently on small datasets.

---

### 2. **Write-Heavy Workloads**

* Indexes **slow down inserts, updates, and deletes**, because MongoDB must update the index too.
* In highly write-intensive systems, **too many indexes can reduce write performance**.

> 🔧 Example: Logging systems or real-time data streams.

---

### 3. **Fields Not Used in Queries**

* If you **never search**, **sort**, or **filter** by a field, there’s no benefit to indexing it.

> 🚫 Don’t index fields just because they exist.

---

### 4. **Fields with Low Selectivity**

* **Low selectivity** means most documents have the same value.
* Indexing such fields **doesn’t help filter** much and wastes space.

> ⚠️ Example: `gender` field with only "male" or "female" values.

---

### 5. **Too Many Indexes**

* Each index **uses RAM and disk**, and **adds write overhead**.
* Don’t create an index for every field — only for **fields often used in queries**.

> 📉 MongoDB might even choose a **bad index** if too many exist.

---

### 6. **When Using `$where` or JavaScript Expressions**

* These cannot use indexes.
* MongoDB must do a full collection scan anyway.

```js
db.users.find({ $where: "this.age + this.salary > 100" })
```

> ❗ This always triggers a COLLSCAN.

---

### 📌 Summary Table:

| Situation              | Index Needed? | Why                      |
| ---------------------- | ------------- | ------------------------ |
| Small collections      | ❌ No          | COLLSCAN is already fast |
| Write-heavy system     | ⚠️ Be careful | Indexes slow down writes |
| Not queried fields     | ❌ No          | No performance benefit   |
| Low-selectivity fields | ❌ No          | Index won't help filter  |
| Too many indexes       | ❌ No          | More harm than good      |
| `$where` or JS logic   | ❌ No          | Can't use indexes        |

---

### ✅ When Indexing Helps:

* Queries on **large collections**
* Frequent **search**, **filter**, **sort**
* **Unique** or **foreign key** fields