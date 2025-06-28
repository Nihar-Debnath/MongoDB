
## 🔰 Common Array Query Operators

| Operator              | Description                                                                        |
| --------------------- | ---------------------------------------------------------------------------------- |
| `$all`                | Matches arrays that contain all specified elements.                                |
| `$elemMatch`          | Matches documents where at least one array element matches all specified criteria. |
| `$size`               | Matches arrays with a specific number of elements.                                 |
| Direct Array Matching | Matches when the query value equals an array element.                              |
| Dot Notation          | Access elements inside array of documents.                                         |

---

## 💡 Sample Collection

```js
db.students.insertMany([
  { name: "Alice", scores: [85, 92, 88] },
  { name: "Bob", scores: [75, 79, 85] },
  { name: "Charlie", scores: [90, 91] },
  { name: "David", scores: [88] }
])
```

---

## 🔹 1. **Direct Array Matching**

### 🎯 Match any document where `scores` array contains the value `88`.

```js
db.students.find({ scores: 88 })
```

### ✅ Output:

Matches Alice and David (because they both have `88` in their array).

---

## 🔹 2. **\$all**

### 🎯 Match arrays that contain **all specified elements**, in any order.

```js
db.students.find({ scores: { $all: [85, 88] } })
```

### ✅ Output:

Matches Alice (has both 85 and 88).

---

## 🔹 3. **\$elemMatch**

Used when you need to match multiple conditions on the **same array element**.

### 💾 New sample collection:

```js
db.products.insertMany([
  { name: "Phone", reviews: [{ rating: 5, user: "A" }, { rating: 3, user: "B" }] },
  { name: "Laptop", reviews: [{ rating: 4, user: "C" }, { rating: 5, user: "D" }] },
  { name: "Tablet", reviews: [{ rating: 3, user: "A" }] }
])
```

### 🎯 Find products where **a single review has rating >= 4 and user is "D"**

```js
db.products.find({
  reviews: {
    $elemMatch: { rating: { $gte: 4 }, user: "D" }
  }
})
```

### ✅ Output:

Matches Laptop (review by D with rating 5).

---

## 🔹 4. **\$size**

### 🎯 Find students who have exactly **2 scores**.

```js
db.students.find({ scores: { $size: 2 } })
```

### ✅ Output:

Matches Charlie (because his scores array has exactly 2 elements).

> 🔸 Note: `$size` **only matches exact array length**, and cannot be combined with `$gt`, `$lt`, etc.

---

## 🔹 5. **Dot Notation for Nested Arrays**

### 🎯 Find products where **any review has rating 5**

```js
db.products.find({ "reviews.rating": 5 })
```

### ✅ Output:

Matches Phone and Laptop (both have at least one review with rating 5).

---

## 🔹 6. **Match a Specific Element by Index**

### 🎯 Match documents where the **first score** is `85`

```js
db.students.find({ "scores.0": 85 })
```

### ✅ Output:

Matches Alice (first element of `scores` is 85).

---

## ✅ Summary Table

| Operator     | Use Case                                                 | Example                                                 |
| ------------ | -------------------------------------------------------- | ------------------------------------------------------- |
| `$all`       | Check array has **all** values                           | `{ scores: { $all: [85, 88] } }`                        |
| `$elemMatch` | Match **one element** that meets **multiple conditions** | `{ reviews: { $elemMatch: { rating: 5, user: "A" } } }` |
| `$size`      | Check exact **length** of array                          | `{ scores: { $size: 2 } }`                              |
| Direct Match | Match if **value exists** in array                       | `{ scores: 88 }`                                        |
| Dot Notation | Access array fields or specific indexes                  | `{ "scores.0": 85 }`, `{ "reviews.rating": 5 }`         |


---
---
---


## 🔰 Sample Collection: `students`

```js
db.students.insertMany([
  {
    name: "Alice",
    age: 21,
    scores: [85, 92, 88],
    hobbies: ["reading", "music", "coding"]
  },
  {
    name: "Bob",
    age: 22,
    scores: [75, 79, 85],
    hobbies: ["gaming", "music"]
  },
  {
    name: "Charlie",
    age: 23,
    scores: [90, 91],
    hobbies: ["travel", "photography"]
  },
  {
    name: "David",
    age: 20,
    scores: [88],
    hobbies: []
  },
  {
    name: "Eve",
    age: 24,
    scores: [95, 99, 100],
    hobbies: ["coding", "music"]
  }
])
```

---

## 🔹 1. `$all` Operator

### 🧠 Use: Match arrays containing **all** specified elements, in any order.

#### 🔍 Find students who like **both music and coding**:

```js
db.students.find({ hobbies: { $all: ["music", "coding"] } })
```

✅ **Matches**: Alice and Eve.

---

## 🔹 2. `$elemMatch` Operator

### 🧠 Use: Match **a single element** in the array that satisfies **multiple conditions**.

Now let’s expand the document:

```js
db.students.updateOne(
  { name: "Alice" },
  { $set: {
      exams: [
        { subject: "math", score: 92 },
        { subject: "physics", score: 88 }
      ]
    }
  }
)

db.students.updateOne(
  { name: "Bob" },
  { $set: {
      exams: [
        { subject: "math", score: 78 },
        { subject: "physics", score: 75 }
      ]
    }
  }
)
```

#### 🔍 Find students who have **at least one exam** where subject is `"math"` and score is greater than 80:

```js
db.students.find({
  exams: {
    $elemMatch: { subject: "math", score: { $gt: 80 } }
  }
})
```

✅ **Matches**: Alice (math: 92)

---

## 🔹 3. `$size` Operator

### 🧠 Use: Match arrays with **exact number of elements**.

#### 🔍 Find students with exactly **3 scores**:

```js
db.students.find({ scores: { $size: 3 } })
```

✅ **Matches**: Alice, Bob, Eve

---

## 🔹 4. Direct Value Match in Array

### 🧠 Use: Checks if the **value exists in the array**.

#### 🔍 Find students who have scored `88` in any exam:

```js
db.students.find({ scores: 88 })
```

✅ **Matches**: Alice and David

---

## 🔹 5. Dot Notation for Nested Arrays

#### 🔍 Find students whose **first score** is 75:

```js
db.students.find({ "scores.0": 75 })
```

✅ **Matches**: Bob

---

## 🔹 6. Combine Conditions on Arrays

#### 🔍 Find students who have a score > 90 and like "music":

```js
db.students.find({
  scores: { $elemMatch: { $gt: 90 } },
  hobbies: "music"
})
```

✅ **Matches**: Alice and Eve

---

## 🔹 7. Querying Arrays of Embedded Documents (More Examples)

### 📁 New Collection: `projects`

```js
db.projects.insertMany([
  {
    title: "AI Research",
    members: [
      { name: "Alice", role: "lead", tasks: ["planning", "coding"] },
      { name: "Bob", role: "engineer", tasks: ["coding"] }
    ]
  },
  {
    title: "Web Development",
    members: [
      { name: "Charlie", role: "designer", tasks: ["UI", "UX"] },
      { name: "Eve", role: "engineer", tasks: ["backend", "frontend"] }
    ]
  },
  {
    title: "Mobile App",
    members: [
      { name: "David", role: "engineer", tasks: ["frontend", "testing"] }
    ]
  }
])
```

#### 🔍 Find projects that have a member named "Eve":

```js
db.projects.find({ "members.name": "Eve" })
```

#### 🔍 Find projects where **someone is an engineer AND has 'frontend' as a task**:

```js
db.projects.find({
  members: {
    $elemMatch: {
      role: "engineer",
      tasks: "frontend"
    }
  }
})
```

✅ **Matches**: Web Development, Mobile App

---

## 🔹 8. Match Embedded Documents Exactly

#### 🔍 Match a member document that is **exactly** `{ name: "Bob", role: "engineer", tasks: ["coding"] }`

```js
db.projects.find({
  members: {
    $eq: { name: "Bob", role: "engineer", tasks: ["coding"] }
  }
})
```

✅ Requires **exact match** (fields, order, values)

---

## ✅ Summary Table (Extended)

| Operator / Feature | Works On           | Description                                                  | Example                                                              |
| ------------------ | ------------------ | ------------------------------------------------------------ | -------------------------------------------------------------------- |
| `$all`             | Arrays             | Match arrays that contain **all** values                     | `{ hobbies: { $all: ["music", "coding"] } }`                         |
| `$elemMatch`       | Arrays             | Match a **single element** in array with multiple conditions | `{ exams: { $elemMatch: { subject: "math", score: { $gt: 90 } } } }` |
| `$size`            | Arrays             | Match arrays with **exact size**                             | `{ scores: { $size: 3 } }`                                           |
| Direct Match       | Arrays             | Match array containing a value                               | `{ scores: 88 }`                                                     |
| Dot Notation       | Nested             | Access array elements or nested fields                       | `{ "scores.0": 85 }`, `{ "members.name": "Eve" }`                    |
| Exact Object Match | Embedded Documents | Match **exact** subdocument                                  | `{ members: { name: ..., role: ..., tasks: [...] } }`                |
