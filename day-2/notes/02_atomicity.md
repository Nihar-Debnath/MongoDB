## 🧨 What is **Atomicity**?

**Atomicity** means **"all-or-nothing"**.

> Either the whole operation **completes successfully**, or **nothing happens at all** — there is **no partial change**.

It ensures that data is **not left in an inconsistent or broken state** if something fails midway.

---

## ⚙️ **Atomicity in MongoDB**

MongoDB **guarantees atomicity at the document level**.

> ✅ **Single-document operations are always atomic** — regardless of the number of fields or updates made inside that one document.

---

### ✅ Example 1: Atomic Update on a Single Document

```js
db.accounts.updateOne(
  { _id: 1 },
  {
    $inc: { balance: -500 },
    $push: { transactions: { type: "withdraw", amount: 500 } }
  }
)
```

Both `balance` update and the `transactions` push happen **together or not at all**.
If the server crashes during this, MongoDB **rolls back** the changes — no partial update.

---

### ❌ Example 2: Non-Atomic Across Multiple Documents

Suppose you're transferring ₹500 from **User A to User B** (two documents):

```js
// Decrease from A
db.accounts.updateOne({ _id: "A" }, { $inc: { balance: -500 } })

// Increase in B
db.accounts.updateOne({ _id: "B" }, { $inc: { balance: +500 } })
```

If a crash happens **after step 1 but before step 2**, User A loses ₹500 and User B doesn’t get it.

⚠️ This is a **partial update** — **not atomic**.

---

## 🧩 Why is Atomicity Needed?

| ✅ Reason                | 🔍 Explanation                                                                   |
| ----------------------- | -------------------------------------------------------------------------------- |
| **Data consistency**    | Prevents half-done writes that leave data in a broken or corrupt state.          |
| **Error recovery**      | If a crash or error happens, MongoDB automatically rolls back uncommitted parts. |
| **Transaction safety**  | Ensures reliability of updates, especially in financial or critical systems.     |
| **Concurrency control** | Multiple users updating data won’t step over each other’s incomplete operations. |

---

## 🧪 What About Multi-Document Atomicity?

Before MongoDB 4.0: ❌ Only **single-document atomicity**.

After MongoDB 4.0: ✅ Supports **multi-document transactions** (like RDBMS).

```js
const session = db.getMongo().startSession();
session.startTransaction();

try {
  const accounts = session.getDatabase("bank").accounts;

  accounts.updateOne({ _id: "A" }, { $inc: { balance: -500 } });
  accounts.updateOne({ _id: "B" }, { $inc: { balance: +500 } });

  session.commitTransaction();
} catch (e) {
  session.abortTransaction();
}
```

Now, **both updates are atomic**: either both happen, or none.

---

## 🚀 Real-World Analogy

Imagine you’re transferring money between two wallets:

* If only ₹500 is deducted from Wallet A, but not added to Wallet B — you’re in trouble.
* **Atomicity = the bank ensures the transfer happens fully or not at all.**

---

## 📌 Summary

| 🔑 Feature                | ✅ In MongoDB                                               |
| ------------------------- | ------------------------------------------------------------ |
| Single-document atomicity | ✅ Always guaranteed                                         |
| Multi-document atomicity  | ✅ From v4.0 via transactions                                |
| Why it’s needed           | Prevents inconsistent data, especially in failure situations |