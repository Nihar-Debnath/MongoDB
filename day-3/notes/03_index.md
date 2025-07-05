### 📋 **1. Single Field Index**

* **Index on one field only**
* Most common type
* Use when you often query a single field

```js
db.users.createIndex({ age: 1 })  // 1 = ascending
```

✅ Use when:

```js
db.users.find({ age: 25 })
```

---

### 📋 **2. Compound Index**

* Index on **two or more fields**
* Useful when your queries filter by **multiple fields**

```js
db.users.createIndex({ age: 1, name: 1 })
```

✅ Use when:

```js
db.users.find({ age: 25, name: "Alice" })
```

⚠️ Order matters:
This index supports queries on `{ age }` and `{ age, name }` — **not just `{ name }`**

---

### 📋 **3. Multikey Index**

* Used when indexing **array fields**
* MongoDB creates index entries for each array element

```js
db.products.createIndex({ tags: 1 })
```

✅ Use when:

```js
db.products.find({ tags: "electronics" })
```

---

### 📋 **4. Text Index**

* Used for **full-text search** in string fields

```js
db.articles.createIndex({ content: "text" })
```

✅ Use when:

```js
db.articles.find({ $text: { $search: "mongodb" } })
```

---

### 📋 **5. Hashed Index**

* Index based on **hash of the field’s value**
* Used for **sharding** (not for range queries)

```js
db.users.createIndex({ user_id: "hashed" })
```

✅ Use when:
You want to distribute data evenly in a sharded cluster.

---

### 📋 **6. Geospatial Index**

* Used for location-based data (latitude, longitude)

```js
db.places.createIndex({ location: "2dsphere" })
```

✅ Use when:

```js
db.places.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [longitude, latitude] },
      $maxDistance: 1000
    }
  }
})
```

---

### 📋 **7. Wildcard Index (`$**`)**

* Indexes **any or all fields**, even dynamic/unpredictable fields

```js
db.logs.createIndex({ "$**": 1 })
```

✅ Use when:
You don’t know the field names in advance, or documents have **different structures**.

---

### ✅ Summary Table:

| Index Type   | Use For                                |
| ------------ | -------------------------------------- |
| Single Field | Searching one field                    |
| Compound     | Searching multiple fields together     |
| Multikey     | Searching inside arrays                |
| Text         | Full-text search                       |
| Hashed       | Equal queries + Sharding               |
| Geospatial   | Location-based queries                 |
| Wildcard     | Flexible documents with unknown fields |
