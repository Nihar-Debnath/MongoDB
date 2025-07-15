### 🔑 What is a **Multikey Index** in MongoDB?

A **multikey index** is a special type of index that **allows indexing fields that contain arrays**.

---

## 🔍 Why Is It Needed?

In MongoDB, a single field in a document can store an array:

```js
{
  name: "Emma",
  tags: ["student", "developer", "musician"]
}
```

If you query:

```js
db.users.find({ tags: "developer" })
```

MongoDB needs to **search inside an array** — a **regular index on `tags` won’t work**.

✅ So MongoDB automatically creates a **multikey index** when you index an array field.

---

## ✅ How to Create a Multikey Index

You **don’t have to do anything special**. Just create an index on the array field:

```js
db.users.createIndex({ tags: 1 })
```

If any document has an **array in the `tags` field**, MongoDB makes it a **multikey index**.

---

## 🔬 How It Works Internally

If a document is:

```js
{ _id: 1, tags: ["student", "developer"] }
```

Then MongoDB creates **multiple index entries** for each element:

```
"tags: student"
"tags: developer"
```

This is what makes it **multi**-key: 1 document generates **multiple keys** in the index.

---

## 🔎 Queries It Supports

```js
// Match any element
db.users.find({ tags: "developer" })

// Match multiple elements (AND-style)
db.users.find({ tags: { $all: ["student", "developer"] } })

// Range inside arrays (with numbers)
db.posts.find({ scores: { $gt: 80 } })
```

---

## ⚠️ Multikey Index Limitations

1. ❌ You **can’t create a compound multikey index** where **more than one field is an array**

   ```js
   // This is not allowed if both fields are arrays:
   db.coll.createIndex({ tags: 1, scores: 1 }) // ❌
   ```

2. ❌ Multikey indexes **cannot support covered queries** unless only one array field is involved and it's projected properly.

3. 🚫 Large arrays can **slow down index builds** and **increase index size**, since each element becomes a separate index entry.

---

## 📘 Check if an Index is Multikey

Use:

```js
db.users.getIndexes()
```

Or better:

```js
db.users.find({ tags: "developer" }).explain("executionStats")
```

Look for this in the output:

```json
"indexName": "tags_1",
"isMultiKey": true
```

---

## ✅ Summary

| Feature                    | Multikey Index Behavior                         |
| -------------------------- | ----------------------------------------------- |
| Indexes array elements     | Yes (each array element becomes a separate key) |
| Created automatically      | Yes, if indexed field is an array               |
| Compound index on 2 arrays | ❌ Not allowed                                   |
| Supports covered queries   | ❌ Limited support                               |
| Useful for                 | Tags, categories, lists, sets, scores, etc.     |



---
---
---


## 🛒 Sample Collection: `products`

```js
db.products.insertMany([
  {
    name: "Wireless Mouse",
    categories: ["electronics", "computer accessories", "gadgets"]
  },
  {
    name: "Cotton T-Shirt",
    categories: ["clothing", "men", "fashion"]
  },
  {
    name: "Yoga Mat",
    categories: ["fitness", "sports", "health"]
  }
])
```

Each `product` has a `categories` field, and it’s an **array**.

---

## 🔍 Use Case

You want to allow users to **search for products by category**, like:

```js
db.products.find({ categories: "electronics" })
```

Without an index, MongoDB would have to **scan every document**. Let’s fix that.

---

## ✅ Step 1: Create a Multikey Index

```js
db.products.createIndex({ categories: 1 })
```

Because `categories` is an **array**, MongoDB will automatically create a **multikey index**.

You can confirm it:

```js
db.products.find({ categories: "electronics" }).explain("executionStats")
```

Look for:

```json
"isMultiKey": true,
"indexName": "categories_1",
"stage": "IXSCAN"
```

---

## 📈 Now It’s Fast!

Let’s run a query:

```js
db.products.find({ categories: "sports" })
```

It will use the **multikey index** to quickly locate documents that have `"sports"` inside their array — no full scan!

---

## 🧠 How MongoDB Stores It Internally

Behind the scenes, for the first product:

```js
{
  name: "Wireless Mouse",
  categories: ["electronics", "computer accessories", "gadgets"]
}
```

MongoDB stores **three index keys**:

```
categories: electronics
categories: computer accessories
categories: gadgets
```

---

## ⚠️ Bonus: Compound Multikey Example (Invalid Case)

This **won’t work**:

```js
db.products.createIndex({ categories: 1, tags: 1 }) // ❌
```

If both `categories` and `tags` are arrays, MongoDB will **reject** this index with an error:

```
Cannot create a compound index with more than one array field.
```

---

## ✅ Summary

| Operation                                    | Result                             |
| -------------------------------------------- | ---------------------------------- |
| Create index on array field                  | ✅ MongoDB creates a multikey index |
| Query a value in the array                   | ✅ Index is used efficiently        |
| Combine two array fields in a compound index | ❌ Not allowed                      |



---
---
---





### ❗ What does this mean?

When you run this MongoDB query:

```js
db.users.find({}, { categories: 1, _id: 0 })
```

You're saying:

> “Return **all documents**, but show me **only the `categories` field**. Don't include `_id`.”

---

### 📦 What happens if a document **does not have a `categories` field**?

MongoDB will still return that document, because you **did not filter it out** — you're using an **empty filter** (`{}`), which matches **all documents**.

But in projection, you're only asking for:

```js
{ categories: 1, _id: 0 }
```

So if a document **doesn't have `categories`**, MongoDB doesn't know what else to show — and you'll get an **empty object**.

---

### 🧪 Example to Demonstrate

Suppose your collection looks like this:

```js
{
  _id: 1,
  name: "Alice",
  categories: ["admin", "editor"]
},
{
  _id: 2,
  name: "Bob"
}
```

Now run:

```js
db.users.find({}, { categories: 1, _id: 0 })
```

Result will be:

```js
{ categories: ["admin", "editor"] }
{}
```

➡️ The second document appears as an **empty object** (`{}`) because:

* It **exists**
* It was **not filtered out**
* But it has **no `categories` field to return**

---

### 🧠 Why This Matters

This is useful when you're only interested in a specific field, like:

* Showing a list of all categories in use
* Exporting a specific field
* Counting or aggregating only one field

But be aware: if some documents don’t have the field, they’ll still be there — just **empty** unless you **filter them out explicitly**.

---

### ✅ How to Avoid Empty Objects

If you want to return only documents that **have a `categories` field**, use a filter like:

```js
db.users.find({ categories: { $exists: true } }, { categories: 1, _id: 0 })
```

Now you will get **only those documents** that actually contain the `categories` field.