Here is the break down each of the **MongoDB update operators** you've listed (`$inc`, `$min`, `$max`, `$mul`, `$unset`, `$rename`) and the concept of **Upsert**, with **clear examples**.

---

## 🔹 1. `$inc` — Increment a Field

**Increments the value of a field by a specified amount.**

### 📦 Example Collection:

```js
db.users.insertOne({ name: "Alice", age: 25, score: 80 })
```

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $inc: { age: 1, score: 10 } }
)
```

➡️ Result:

```js
{ name: "Alice", age: 26, score: 90 }
```

> If the field doesn't exist, `$inc` creates it.

---

## 🔹 2. `$min` — Set to Minimum Value

**Updates the field only if the specified value is **less than** the current value.**

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $min: { score: 85 } }
)
```

➡️ Result:

* Current `score` = 90 → No change
* If `score` = 80 → It would update to 85

---

## 🔹 3. `$max` — Set to Maximum Value

**Updates the field only if the specified value is **greater than** the current value.**

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $max: { score: 95 } }
)
```

➡️ Result:

* Current `score` = 90 → Updates to 95
* If `score` = 98 → No change

---

## 🔹 4. `$mul` — Multiply a Field

**Multiplies the value of a field by a number.**

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $mul: { score: 2 } }
)
```

➡️ Result:

* If `score` = 95 → Now `score` = 190

---

## 🔹 5. `$unset` — Remove a Field

**Removes a field from the document.**

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $unset: { score: "" } }
)
```

➡️ Result:

* `score` field is completely removed from the document.

---

## 🔹 6. `$rename` — Rename a Field

**Changes the name of a field.**

### ✅ Query:

```js
db.users.updateOne(
  { name: "Alice" },
  { $rename: { "age": "userAge" } }
)
```

➡️ Result:

```js
{ name: "Alice", userAge: 26 }
```

---

## 🔹 7. **Upsert** — Update or Insert

**`Upsert` = Update if document exists, or Insert a new one if it doesn’t.**

* Used with `updateOne()`, `updateMany()`, or `replaceOne()`

### ✅ Query:

```js
db.users.updateOne(
  { name: "Bob" },
  { $set: { age: 30, score: 85 } },
  { upsert: true }
)
```

➡️ If "Bob" exists: update the document
➡️ If "Bob" doesn't exist: insert this new document:

```js
{ name: "Bob", age: 30, score: 85 }
```

---

## ✅ Summary Table

| Operator  | Purpose                                 | Example                             |
| --------- | --------------------------------------- | ----------------------------------- |
| `$inc`    | Increments a number                     | `{ $inc: { age: 1 } }`              |
| `$min`    | Sets field only if new value is smaller | `{ $min: { score: 85 } }`           |
| `$max`    | Sets field only if new value is larger  | `{ $max: { score: 95 } }`           |
| `$mul`    | Multiplies field by value               | `{ $mul: { score: 2 } }`            |
| `$unset`  | Removes field                           | `{ $unset: { score: "" } }`         |
| `$rename` | Renames a field                         | `{ $rename: { "age": "userAge" } }` |
| `upsert`  | Update or insert                        | `{ upsert: true }`                  |
