```sql

-- for delete the whole database
db.dropDatabase()

-- for delete a perticular collections
db.students.drop()

db.books.insertMany([{_id:'A',name:'A',price:10},{_id:'B',name:'B',price:20},{_id:'C',name:'C',price:30},{_id:'D',name:'D',price:40}],{ ordered: false })

```


In MongoDB, the **`ordered`** option in the `insert()` or `insertMany()` command determines **how MongoDB handles multiple document inserts when an error occurs**.

---

### 🔹 Syntax (for `insertMany`)

```javascript
db.collection.insertMany(
  [ {doc1}, {doc2}, {doc3}, ... ],
  { ordered: true }  // or false
)
```

---

### 🔹 Behavior

| `ordered` Value  | Behavior                                                        |
| ---------------- | --------------------------------------------------------------- |
| `true` (default) | Stops inserting on the **first error** (e.g., duplicate `_id`). |
| `false`          | Continues inserting **remaining documents** even if one fails.  |

---

### 🔹 Example

#### Case 1: `ordered: true` (default)

```javascript
db.users.insertMany([
  { _id: 1, name: "Amit" },
  { _id: 2, name: "Neha" },
  { _id: 1, name: "Duplicate ID" } // This causes an error
], { ordered: true })
```

➡ Only documents with `_id: 1` and `_id: 2` are attempted.
Once the duplicate `_id: 1` is found, MongoDB **stops**, and `_id: 1` (duplicate) is **not inserted** — nor any following docs.

---

#### Case 2: `ordered: false`

```javascript
db.users.insertMany([
  { _id: 1, name: "Amit" },
  { _id: 2, name: "Neha" },
  { _id: 1, name: "Duplicate ID" },
  { _id: 3, name: "Kunal" }
], { ordered: false })
```

➡ MongoDB inserts `_id: 1` and `_id: 2`, **skips** the duplicate `_id: 1`, then **continues** and inserts `_id: 3`.

---

### 🔹 Why is it useful?

* Use `ordered: true` if you want to **guarantee sequence** and **avoid partial inserts**.
* Use `ordered: false` for **bulk inserts** where **speed and maximizing success** are more important than order.