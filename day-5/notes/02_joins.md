## 🧩 What is a **Join**?

A **join** means **combining data** from **two different collections** (or tables in SQL) based on a **common field**.

> Example: You want to show a user and **their orders** — but user data is in `users` collection and orders are in `orders` collection. So, you "join" the two using `userId`.

---

## 📦 In MongoDB, joins are done using `$lookup`.

---

## 📚 Types of Joins in MongoDB

MongoDB mainly supports **4 types of joins** using `$lookup`.

Let’s take two collections:

### `users`:

```js
{ _id: 1, name: "Alice" }
{ _id: 2, name: "Bob" }
{ _id: 3, name: "Charlie" }
```

### `orders`:

```js
{ _id: 101, userId: 1, item: "Laptop" }
{ _id: 102, userId: 2, item: "Phone" }
{ _id: 103, userId: 1, item: "Mouse" }
```

---

### ✅ 1. **Left Outer Join (Default in MongoDB)**

🧠 **Keep all users**, and attach their orders (if any). If no orders, still show the user.

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

### ✅ Output:

```js
{
  _id: 1,
  name: "Alice",
  orders: [ { item: "Laptop" }, { item: "Mouse" } ]
}
{
  _id: 2,
  name: "Bob",
  orders: [ { item: "Phone" } ]
}
{
  _id: 3,
  name: "Charlie",
  orders: [] // No orders but still shown
}
```

---

### ✅ 2. **Inner Join (Manual)**

🧠 **Only show users who have at least one order**.

You do the same `$lookup`, but then **filter out** users with no orders using `$match`.

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
    $match: { orders: { $ne: [] } }
  }
])
```

### ✅ Output:

Only Alice and Bob are shown. Charlie is skipped.

---

### ✅ 3. **Right Outer Join** (Emulated)

🧠 MongoDB doesn’t support this directly, but you can **flip the collections**:

> Show **all orders** with their matching users (if any).

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

You’re joining from `orders` → `users`, so it's like a right join.

---

### ✅ 4. **Full Outer Join** (Emulated)

🧠 MongoDB doesn’t support this directly either. You simulate it by:

* Doing a **left join** from `users` → `orders`
* Doing a **left join** from `orders` → `users`
* Then **merging** both results (in code, not inside MongoDB).

So **not recommended unless truly needed**.

---

## ✅ Summary Table (Simple):

| Join Type        | MongoDB Support?      | Shows...                                 |
| ---------------- | --------------------- | ---------------------------------------- |
| Left Outer Join  | ✅ Yes (default)       | All users, with their orders if any      |
| Inner Join       | ✅ Yes (manual)        | Only users who have orders               |
| Right Outer Join | ⚠️ Emulated           | All orders, with user info if available  |
| Full Outer Join  | ❌ No (emulate in app) | All users and all orders, even unmatched |
