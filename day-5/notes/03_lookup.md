## 🔍 What is `$lookup` in MongoDB?

### `$lookup` is used to **join data from two collections** — just like **JOIN** in SQL.

MongoDB is a **NoSQL database**, so by default it stores data in **separate collections** (like separate tables). But sometimes you still need to **combine related data** from different collections — that’s where `$lookup` comes in.

---

## 🧠 Why is `$lookup` needed?

Because:

* MongoDB stores **related data separately** for flexibility and scalability.
* But in **real-world applications**, you often need to **show combined data**.
* `$lookup` lets you **join related data** **without changing the schema**.

---

## 📦 Real-World Example: **Users and Orders**

Let’s say you have two collections:

### ➤ `users` collection:

```js
{
  _id: 1,
  name: "Alice",
  email: "alice@example.com"
}
```

### ➤ `orders` collection:

```js
{
  _id: 101,
  userId: 1,
  item: "Laptop",
  price: 1200
}
```

---

## ✅ Goal:

> Show the user's name **with their order details**.

---

## 🛠 `$lookup` Query:

```js
db.users.aggregate([
  {
    $lookup: {
      from: "orders",           // the other collection
      localField: "_id",        // field in users collection
      foreignField: "userId",   // field in orders collection
      as: "user_orders"         // output array field name
    }
  }
])
```

---

## ✅ Result:

```js
{
  _id: 1,
  name: "Alice",
  email: "alice@example.com",
  user_orders: [
    {
      _id: 101,
      userId: 1,
      item: "Laptop",
      price: 1200
    }
  ]
}
```

---

### 🧾 Real-World Use Case Example:

In an **e-commerce website**:

* `users` contains customer info.
* `orders` contains their purchases.
* To show **order history on a profile page**, you use `$lookup` to **join orders with the user**.

---

## ✅ Summary:

| Feature       | Explanation                                                         |
| ------------- | ------------------------------------------------------------------- |
| `$lookup`     | Joins documents from two collections.                               |
| Why needed    | To combine related data (like in SQL JOIN).                         |
| Real use case | Showing user info along with their orders, comments, messages, etc. |



---
---
---



## 🧩 Step 1: Insert Sample Data

### ✅ Insert into `users` collection:

```js
db.users.insertMany([
  { _id: 1, name: "Alice" },
  { _id: 2, name: "Bob" },
  { _id: 3, name: "Charlie" },
  { _id: 4, name: "David" }
])
```

### ✅ Insert into `orders` collection:

```js
db.orders.insertMany([
  { _id: 101, userId: 1, item: "Laptop" },
  { _id: 102, userId: 1, item: "Mouse" },
  { _id: 103, userId: 2, item: "Phone" },
  { _id: 104, userId: 5, item: "Tablet" } // no matching user
])
```

---

Now, let’s run each type of **join** using aggregation and `$lookup`.

---

## 🔶 1. **Left Outer Join (default in MongoDB)**

> Show **all users** and their **orders if any**.

```js
db.users.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  }
])
```

### ✅ Output (Simplified):

```js
{
  name: "Alice",
  orders: [ { item: "Laptop" }, { item: "Mouse" } ]
}
{
  name: "Bob",
  orders: [ { item: "Phone" } ]
}
{
  name: "Charlie",
  orders: []
}
{
  name: "David",
  orders: []
}
```

---

## 🔶 2. **Inner Join**

> Show only users **who have orders** (no empty `orders` array).

```js
db.users.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  },
  {
    $match: {
      orders: { $ne: [] }
    }
  }
])
```

### ✅ Output:

```js
{
  name: "Alice",
  orders: [ { item: "Laptop" }, { item: "Mouse" } ]
}
{
  name: "Bob",
  orders: [ { item: "Phone" } ]
}
```

---

## 🔶 3. **Right Outer Join (emulated)**

> Show all **orders**, with user details if found.

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
])
```

### ✅ Output:

```js
{
  item: "Laptop",
  user: [ { name: "Alice" } ]
}
{
  item: "Mouse",
  user: [ { name: "Alice" } ]
}
{
  item: "Phone",
  user: [ { name: "Bob" } ]
}
{
  item: "Tablet",
  user: [] // No user with userId: 5
}
```

---

## 🔶 4. **Full Outer Join (emulated in 2 steps)**

> Combine **all users** and **all orders**, even if they don’t match.

⚠️ MongoDB doesn’t support full outer join directly. You need to do it like this:

### ✅ A. Get all users with orders (left join):

```js
const usersWithOrders = db.users.aggregate([
  {
    $lookup: {
      from: "orders",
      localField: "_id",
      foreignField: "userId",
      as: "orders"
    }
  }
]).toArray()
```

### ✅ B. Get all orders with users (right join style):

```js
const ordersWithUsers = db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "userId",
      foreignField: "_id",
      as: "user"
    }
  }
]).toArray()
```

### ✅ C. Merge `usersWithOrders` and orders from `ordersWithUsers` where user is missing (in app code, like in JavaScript/Python).

---

## ✅ Summary Table of Joins

| Type            | Supported?  | Example Output                       |
| --------------- | ----------- | ------------------------------------ |
| Left Join       | ✅ Yes       | All users + their orders if any      |
| Inner Join      | ✅ Manual    | Only users with at least 1 order     |
| Right Join      | ⚠️ Emulated | All orders + matched users if any    |
| Full Outer Join | ❌ Manual    | All users and orders, matched or not |
