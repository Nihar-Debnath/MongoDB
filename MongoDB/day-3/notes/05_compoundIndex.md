```js
db.users.dropIndex("age_1")

db.users.createIndex({ age: 1, gender:1 })

db.users.find({ age: { $gte: 27 }, gender:"male" }).explain("executionStats")

db.users.find({ age: { $lte: 30 } }).explain("executionStats")

db.users.find({  gender:"male" }).explain("executionStats")

```

### 🔧 Step-by-Step Breakdown:

---

### 🔹 `db.users.dropIndex("age_1")`

**What it does:**

* Drops the **single-field index** on `age` that you previously created with:

  ```js
  db.users.createIndex({ age: 1 })
  ```
* MongoDB will now only use **compound indexes** (if available) for queries on `age`.

---

### 🔹 `db.users.createIndex({ age: 1, gender: 1 })`

**What it does:**

* Creates a **compound index** where:

  * `age` is the **first field** (main sort/filter key)
  * `gender` is the **second field**

**MongoDB can now use this index for queries that:**

* Use `age` alone ✅
* Use `age` and `gender` together ✅
* Use `age` for filtering and `gender` for sorting ✅
* **Cannot use this index efficiently if you query only on `gender`** ❌ (unless it scans the whole index)

---

### 🔹 `db.users.find({ age: { $gte: 27 }, gender: "male" }).explain("executionStats")`

✅ **This WILL use the compound index!**

**Why?**

* The query uses both `age` and `gender` in the **same order** as the index `{ age: 1, gender: 1 }`
* You will see:

  ```json
  "stage": "IXSCAN"
  "indexName": "age_1_gender_1"
  "totalDocsExamined": (only matching documents)
  ```

✔️ Efficient and fast.

---

### 🔹 `db.users.find({ age: { $lte: 30 } }).explain("executionStats")`

✅ **This WILL ALSO use the compound index!**

**Why?**

* Even though the query only uses `age`, it's the **first field in the index**, so it’s fully usable.

You will likely see:

```json
"stage": "IXSCAN"
"totalDocsExamined": (only those with age ≤ 30)
```

✔️ Still efficient.

---

### 🔹 `db.users.find({ gender: "male" }).explain("executionStats")`

⚠️ **This will NOT efficiently use the compound index `{ age: 1, gender: 1 }`**

**Why?**

* In a compound index, **the first field must be used** for MongoDB to efficiently use the index.
* Since `age` is not included in the query, MongoDB **cannot jump into the index**.
* So this will likely fall back to **COLLSCAN** or scan the **entire index**.

You will likely see:

```json
"stage": "COLLSCAN" or "IXSCAN" + full scan
"totalDocsExamined": 30
```

⚠️ **Not efficient**.

---

### 📌 Summary Table

| Query                                   | Index Used? | Efficient? | Notes                                            |
| --------------------------------------- | ----------- | ---------- | ------------------------------------------------ |
| `{ age: { $gte: 27 }, gender: "male" }` | ✅ Yes       | ✅ Fast     | Perfect match for compound index                 |
| `{ age: { $lte: 30 } }`                 | ✅ Yes       | ✅ Fast     | Uses first field of index                        |
| `{ gender: "male" }`                    | ❌ No        | ❌ Slow     | Skips `age` — cannot use compound index properly |

---

### ✅ Pro Tip:

> Compound indexes must be used **starting from the leftmost field**. If your query skips the first field, MongoDB can’t use the index efficiently.

if you want to test a compound index with **gender first** instead (`{ gender: 1, age: 1 }`) and see the behavior flip!
