## 🎯 Why Use Weights?

When MongoDB performs a `$text` search, by default:

* All fields in the text index are treated **equally**.
* But in real life, some fields (like `title`) are **more important** than others (like `content` or `tags`).

👉 Weights allow you to say:

> “If the search word is in the **title**, that’s way more important than if it’s in the **content**.”

---

## 📘 Basic Syntax: Assigning Weights

```js
db.articles.createIndex(
  { title: "text", content: "text", tags: "text" },
  {
    weights: {
      title: 10,
      content: 5,
      tags: 1
    },
    name: "TextIndex"
  }
)
```

This means:

| Field     | Weight |
| --------- | ------ |
| `title`   | 10     |
| `content` | 5      |
| `tags`    | 1      |

> So if "MongoDB" is found in the `title`, it contributes **10x more** to the **relevance score** than if it's in `tags`.

---

## 🧪 Real Example

```js
// Insert three documents
db.articles.insertMany([
  {
    title: "MongoDB Guide",
    content: "Learn how to use MongoDB",
    tags: ["database", "NoSQL"]
  },
  {
    title: "Databases Explained",
    content: "MongoDB is a NoSQL database",
    tags: ["MongoDB"]
  },
  {
    title: "NoSQL Trends",
    content: "MongoDB MongoDB MongoDB",
    tags: []
  }
])

// Create index with weights
db.articles.createIndex(
  { title: "text", content: "text", tags: "text" },
  {
    weights: {
      title: 10,
      content: 5,
      tags: 1
    },
    name: "weightedTextIndex"
  }
)
```

Now run:

```js
db.articles.find(
  { $text: { $search: "MongoDB" } },
  { score: { $meta: "textScore" }, title: 1 }
).sort({ score: { $meta: "textScore" } })
```

---

## 📈 MongoDB Calculates the Score Like:

* doc#1:

  * `"MongoDB"` in `title` → 10
  * `"MongoDB"` in `content` → 5
  * Total = 15 (high score ✅)

* doc#2:

  * `"MongoDB"` in `content` → 5
  * `"MongoDB"` in `tags` → 1
  * Total = 6

* doc#3:

  * `"MongoDB"` appears 3 times in `content` → 3 × 5 = 15
  * Total = 15

So depending on where and **how many times** a word appears, the score changes.

---

## 🧠 Key Rules About Weights

| Rule                               | Explanation                           |
| ---------------------------------- | ------------------------------------- |
| Only works on `"text"` index       | You must use `{ field: "text" }` type |
| Only one text index per collection | MongoDB allows just one text index    |
| Default weight = 1                 | If you don’t set it, each field = 1   |
| Higher weight = higher score       | That document ranks higher in results |
| Affects `textScore`                | Doesn't affect storage, only ranking  |

---

## 🔎 Query Without Score Sorting?

If you don’t use `textScore`, weights **still influence** which documents appear **first**, but you won’t see the actual number.

So always include:

```js
{ score: { $meta: "textScore" } }
```

---

## ✅ Summary

| Feature       | Purpose                                |
| ------------- | -------------------------------------- |
| Text index    | Enables full-text search               |
| Weights       | Tells MongoDB which fields matter more |
| `textScore`   | Shows how relevant a document is       |
| `sort(score)` | Ranks best results at the top          |

---

### Example Analogy:

* Searching `"iPhone"` on Flipkart
* If it appears in the **product title**, it’s a strong match 🔥
* If it’s only in a review or tag, it’s a weak match

Weights give you this kind of power in your own app.

