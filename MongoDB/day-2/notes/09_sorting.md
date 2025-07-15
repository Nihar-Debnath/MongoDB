## 🔹 Syntax of `sort()`

```js
db.collection.find(query).sort({ field1: 1, field2: -1 })
```

| Value | Meaning    |
| ----- | ---------- |
| `1`   | Ascending  |
| `-1`  | Descending |

---

## 🔰 Sample Collection

```js
db.students.insertMany([
  { name: "Alice", age: 21, score: 88 },
  { name: "Bob", age: 22, score: 75 },
  { name: "Charlie", age: 23, score: 91 },
  { name: "David", age: 20, score: 85 },
  { name: "Eve", age: 24, score: 91 }
])
```

---

## 🔹 1. Sort by One Field

### 🔍 Sort students by `score` in **ascending** order:

```js
db.students.find().sort({ score: 1 })
```

✅ Output (lowest score first):

```
Bob     - 75  
David   - 85  
Alice   - 88  
Charlie - 91  
Eve     - 91  
```

---

### 🔍 Sort by `score` in **descending** order:

```js
db.students.find().sort({ score: -1 })
```

✅ Output (highest score first):

```
Charlie - 91  
Eve     - 91  
Alice   - 88  
David   - 85  
Bob     - 75  
```

---

## 🔹 2. Sort by Multiple Fields

When two or more documents have the **same value** for the first field, MongoDB uses the **next field** to sort.

### 🔍 Sort by `score` (descending), then by `age` (ascending):

```js
db.students.find().sort({ score: -1, age: 1 })
```

✅ Output:

```
Charlie - 91 (age 23)
Eve     - 91 (age 24)
Alice   - 88
David   - 85
Bob     - 75
```

---

## 🔹 3. Sort with Filter

You can use `.sort()` after `.find()` with a filter.

### 🔍 Find students with score >= 85 and sort by age descending:

```js
db.students.find({ score: { $gte: 85 } }).sort({ age: -1 })
```

✅ Output:

```
Eve     - age 24  
Charlie - age 23  
Alice   - age 21  
David   - age 20  
```

---

## 🔹 4. Sort + Limit

To show top `N` results (like Top 3 scores), use `.limit()` with `.sort()`.

### 🔍 Top 3 highest scores:

```js
db.students.find().sort({ score: -1 }).limit(3)
```

✅ Output:

```
Charlie - 91  
Eve     - 91  
Alice   - 88  
```

---

## 🔹 5. Sort on Embedded/Nested Fields

### Sample:

```js
db.books.insertMany([
  { title: "Book A", reviews: { stars: 4, count: 120 } },
  { title: "Book B", reviews: { stars: 5, count: 50 } },
  { title: "Book C", reviews: { stars: 4, count: 200 } }
])
```

### 🔍 Sort books by `reviews.stars` (descending), then `reviews.count` (descending):

```js
db.books.find().sort({ "reviews.stars": -1, "reviews.count": -1 })
```

✅ Output:

```
Book B - 5 stars  
Book C - 4 stars, 200 reviews  
Book A - 4 stars, 120 reviews  
```

---

## ⚠️ Notes

* If you sort large datasets without indexes on sorted fields, it may impact performance.
* You can use `.hint()` to manually suggest index usage.
* If your sort result exceeds memory, MongoDB may throw an error unless you allow disk use via aggregation.

---

## ✅ Summary Table

| Goal                     | Query                                            |
| ------------------------ | ------------------------------------------------ |
| Sort by field ascending  | `.sort({ field: 1 })`                            |
| Sort by field descending | `.sort({ field: -1 })`                           |
| Sort by multiple fields  | `.sort({ field1: -1, field2: 1 })`               |
| Filter + sort            | `.find({ age: { $gt: 20 } }).sort({ score: 1 })` |
| Top N results            | `.sort({ score: -1 }).limit(3)`                  |
| Sort by nested field     | `.sort({ "reviews.stars": -1 })`                 |
