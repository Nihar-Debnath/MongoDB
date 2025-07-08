## 🧩 What is `$project`?

`$project` is an **aggregation stage** in MongoDB used to:

### ✅ Choose **which fields to include or exclude**

### ✅ Rename, reshape, or compute **new fields**

### ✅ Format the output of your query

Think of it like **customizing your result** to show **only what you need**.

---

## 🧠 Why is `$project` needed?

Because in real apps:

* You **don’t always want the full document**
* You may want to **hide sensitive info**
* Or show only the fields **you care about**
* Or even **compute a new field** (e.g., full name, total price, etc.)

---

## 📦 Real World Example: Users Collection

Let’s say you have a collection:

```js
{
  _id: 1,
  firstName: "Alice",
  lastName: "Johnson",
  email: "alice@example.com",
  password: "secret123",
  age: 28
}
```

---

## ✅ Example 1: Show only name and email (hide password)

```js
db.users.aggregate([
  {
    $project: {
      _id: 0,
      firstName: 1,
      lastName: 1,
      email: 1
    }
  }
])
```

### ✅ Output:

```js
{
  firstName: "Alice",
  lastName: "Johnson",
  email: "alice@example.com"
}
```

You **hid `_id` and password**, and **only showed the important parts**.

---

## ✅ Example 2: Create a full name field

```js
db.users.aggregate([
  {
    $project: {
      _id: 0,
      fullName: { $concat: ["$firstName", " ", "$lastName"] },
      email: 1
    }
  }
])
```

### ✅ Output:

```js
{
  fullName: "Alice Johnson",
  email: "alice@example.com"
}
```

You used `$project` to **combine fields into one**.

---

## ✅ Example 3: Calculate a discount price

Suppose you have:

```js
{
  item: "Laptop",
  price: 1000,
  discount: 0.1
}
```

Use `$project` to create a `finalPrice`:

```js
db.products.aggregate([
  {
    $project: {
      item: 1,
      finalPrice: { $subtract: ["$price", { $multiply: ["$price", "$discount"] }] }
    }
  }
])
```

### ✅ Output:

```js
{
  item: "Laptop",
  finalPrice: 900
}
```

---

## ✅ Summary (Very Simple):

| Feature        | What it does                           |
| -------------- | -------------------------------------- |
| Include fields | Show only what you need                |
| Exclude fields | Hide sensitive or unnecessary fields   |
| Rename/Combine | Make new fields like `fullName`        |
| Compute values | Calculate new fields like `finalPrice` |

---

## 🧠 Analogy:

> Think of `$project` like creating a **custom report** from a database. You choose what columns to include, what to hide, and even make new columns.



---
---
---


## ✅ 1. **Hiding Sensitive Information**

In many apps (like e-commerce or social media), you don’t want to expose sensitive fields like passwords or internal IDs.

### 🔹 Example: Hide `password` and `_id` from user info

```js
db.users.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      email: 1,
      password: 0
    }
  }
])
```

📌 **Use case:** Send user profile data to frontend **without leaking passwords**.

---

## ✅ 2. **Creating Computed Fields**

You can create new fields on the fly using math or string operations.

### 🔹 Example: Add a `totalPrice` field for orders

```js
db.orders.aggregate([
  {
    $project: {
      item: 1,
      quantity: 1,
      price: 1,
      totalPrice: { $multiply: ["$quantity", "$price"] }
    }
  }
])
```

📌 **Use case:** Show total amount in an invoice or cart.

---

## ✅ 3. **Combining Fields**

You can combine first name + last name, or address components.

### 🔹 Example: Create `fullName` field

```js
db.users.aggregate([
  {
    $project: {
      fullName: { $concat: ["$firstName", " ", "$lastName"] },
      email: 1
    }
  }
])
```

📌 **Use case:** Display nicely formatted names in dashboards or emails.

---

## ✅ 4. **Renaming Fields**

If your frontend expects different field names.

```js
db.products.aggregate([
  {
    $project: {
      productName: "$name",
      cost: "$price",
      _id: 0
    }
  }
])
```

📌 **Use case:** API formatting before sending data to client.

---

## ✅ 5. **Conditionally Showing Data**

With `$cond` (like if-else), you can show different values based on conditions.

### 🔹 Example: Show if user is adult or minor

```js
db.users.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      isAdult: {
        $cond: { if: { $gte: ["$age", 18] }, then: true, else: false }
      }
    }
  }
])
```

📌 **Use case:** Set access control, tagging, or UI behavior.

---

## ✅ 6. **Trimming / Cleaning Up Data**

You can remove unnecessary nested fields or flatten objects.

### 🔹 Example: Clean up nested address

```js
db.users.aggregate([
  {
    $project: {
      name: 1,
      city: "$address.city",
      country: "$address.country"
    }
  }
])
```

📌 **Use case:** Format data for analytics or mobile display.

---

## ✅ 7. **Show Only Date Part of Timestamp**

If you have a full timestamp and want to extract just the date.

```js
db.logs.aggregate([
  {
    $project: {
      message: 1,
      dateOnly: { $dateToString: { format: "%Y-%m-%d", date: "$timestamp" } }
    }
  }
])
```

📌 **Use case:** Show logs grouped by date (not full time).

---

## ✅ 8. **Masking Emails (for privacy)**

Only show part of an email (like for account recovery).

```js
db.users.aggregate([
  {
    $project: {
      email: {
        $concat: [
          { $substr: ["$email", 0, 3] },
          "*****@",
          { $substr: ["$email", { $indexOfBytes: ["$email", "@"] }, 100 ] }
        ]
      }
    }
  }
])
```

📌 **Use case:** Secure display in UI (like Gmail does).

---

## ✅ Summary Table of Real-World Uses of `$project`

| Use Case              | Purpose                                    |
| --------------------- | ------------------------------------------ |
| Hide fields           | Protect sensitive data like passwords      |
| Combine fields        | Full name, full address                    |
| Compute new fields    | totalPrice, isAdult                        |
| Rename fields         | API response formatting                    |
| Flatten nested fields | Easier output for frontend                 |
| Format dates          | Clean UI display                           |
| Mask user data        | Partial email, phone                       |
| Conditional values    | Based on logic (e.g., VIP customers, etc.) |
