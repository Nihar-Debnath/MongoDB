### Dont worry this all queries looks like a javascript language, but it will smoothly run on mongDb shell

In MongoDB, **transactions** allow you to execute multiple operations atomically, just like in SQL databases. Transactions are available for **replica sets** and **sharded clusters** and are supported **only on WiredTiger storage engine** with **MongoDB 4.0+**.

Below is a complete **step-by-step process** for running a **multi-document transaction** in the **MongoDB shell** (`mongosh`).

---

## ✅ Step-by-Step: Transactions in Mongo Shell (`mongosh`)

---

### 🧱 Step 1: Ensure You Are Using a Replica Set

Transactions **require** a **replica set**, even if it has only one node.

To check:

```js
rs.status()
```

If you’re not in a replica set, you can initialize one for local testing:

```js
rs.initiate()
```

---

### 📘 Step 2: Create Sample Collections

Let’s create two collections to demonstrate a fund transfer:

* `accounts`
* `transactions_log`

```js
use bank

db.accounts.insertMany([
  { _id: 1, name: "Alice", balance: 1000 },
  { _id: 2, name: "Bob", balance: 500 }
])
```

---

### ⚙️ Step 3: Start a Session

```js
session = db.getMongo().startSession()
```

---

### 🧵 Step 4: Get the Session Database

```js
sessionDB = session.getDatabase("bank")
```

---

### 🔁 Step 5: Start a Transaction

```js
session.startTransaction()
```

---

### 💳 Step 6: Run Transactional Operations

Let’s say Alice sends ₹200 to Bob. You need to:

* Deduct ₹200 from Alice’s account
* Add ₹200 to Bob’s account
* Log the transaction

```js
sessionDB.accounts.updateOne(
  { _id: 1 },
  { $inc: { balance: -200 } }
)

sessionDB.accounts.updateOne(
  { _id: 2 },
  { $inc: { balance: 200 } }
)

sessionDB.transactions_log.insertOne({
  from: "Alice",
  to: "Bob",
  amount: 200,
  date: new Date()
})
```

---

### ✅ Step 7: Commit the Transaction

```js
session.commitTransaction()
```

---

### ❌ Optional: Abort the Transaction

If anything goes wrong and you want to cancel:

```js
session.abortTransaction()
```

---

### 🧹 Step 8: End the Session

```js
session.endSession()
```

---

## 📌 Full Transaction Example (Copy-Paste Ready)

```js
session = db.getMongo().startSession()
sessionDB = session.getDatabase("bank")

try {
  session.startTransaction()

  sessionDB.accounts.updateOne(
    { _id: 1 },
    { $inc: { balance: -200 } }
  )

  sessionDB.accounts.updateOne(
    { _id: 2 },
    { $inc: { balance: 200 } }
  )

  sessionDB.transactions_log.insertOne({
    from: "Alice",
    to: "Bob",
    amount: 200,
    date: new Date()
  })

  session.commitTransaction()
  print("✅ Transaction committed")
} catch (error) {
  session.abortTransaction()
  print("❌ Transaction aborted: " + error)
} finally {
  session.endSession()
}
```

---

## 🔐 Important Notes

* Transactions must **complete within 60 seconds** by default.
* Avoid long-running operations in a transaction.
* Transactions have **write conflicts**—if a document is locked, other writes may fail.

---
---
---



Sure! Let's break down **each and every part** of this MongoDB transaction code in **full detail**, so you understand exactly what is happening at every step.

---

## 🔁 FULL EXPLANATION OF THE TRANSACTION CODE:

```js
session = db.getMongo().startSession()
```

* **`db.getMongo()`**: gets the underlying connection to the MongoDB server.
* **`.startSession()`**: starts a new **client session**. Transactions require sessions.
* ✅ Now you have a session object stored in `session`.

---

```js
sessionDB = session.getDatabase("bank")
```

* This line connects your **session** to the **`bank` database**.
* `sessionDB` is just a variable name (like `db`) but tied to the session context.
* 🔗 All read/write operations during the transaction should use `sessionDB`, **not `db`**.

---

```js
try {
    session.startTransaction()
```

* This line begins the **multi-document transaction**.
* Anything between `startTransaction()` and `commitTransaction()` will be treated **atomically**—it either **all succeeds**, or **all fails**.

---

### 🔧 These are the actual transactional operations:

```js
sessionDB.accounts.updateOne(
  { _id: 1 },
  { $inc: { balance: -200 } }
)
```

* **Reduce ₹200 from Alice's account**
* `accounts` is the collection.
* `updateOne()` finds the document with `_id: 1` (Alice), and `$inc` subtracts 200 from the `balance`.

---

```js
sessionDB.accounts.updateOne(
  { _id: 2 },
  { $inc: { balance: 200 } }
)
```

* **Add ₹200 to Bob's account**
* Finds document with `_id: 2` (Bob), and `$inc` increases balance by 200.

---

```js
sessionDB.transactions_log.insertOne({
  from: "Alice",
  to: "Bob",
  amount: 200,
  date: new Date()
})
```

* Logs the transaction into a **separate `transactions_log` collection**.
* Stores:

  * `from`: "Alice"
  * `to`: "Bob"
  * `amount`: 200
  * `date`: current date & time using `new Date()`

---

### ✅ COMMIT

```js
session.commitTransaction()
```

* This tells MongoDB to **finalize** and **apply all the operations**.
* If nothing goes wrong, all changes are committed atomically.

---

### ❌ CATCH (Error Handling)

```js
} catch (error) {
  session.abortTransaction()
  print("❌ Transaction aborted: " + error)
```

* If **any error** happens inside the `try` block (e.g., document not found, write conflict, connection error):

  * It will `abortTransaction()`.
  * This means **nothing gets written to the DB**—rollback happens.
  * Then it prints the error message.

---

### 🔚 FINALLY (Cleanup)

```js
} finally {
  session.endSession()
}
```

* This always runs at the end (whether success or failure).
* Ends the session and frees resources.
* **Important to call** this to avoid resource leaks on the server.

---

## 🧠 Summary of What Happened:

| Step | Action                                |
| ---- | ------------------------------------- |
| 1    | Start a session                       |
| 2    | Attach session to the "bank" database |
| 3    | Begin a transaction                   |
| 4    | Subtract ₹200 from Alice              |
| 5    | Add ₹200 to Bob                       |
| 6    | Log the transaction                   |
| 7    | Commit all changes atomically         |
| 8    | Rollback if any error occurs          |
| 9    | End the session                       |

---

## ⚠️ What Happens Internally

* MongoDB **locks** the documents involved during the transaction.
* It ensures **isolation** (no other user can interfere).
* The entire set of changes is either **fully applied** or **not at all**.
* It uses **write-ahead logging** to ensure durability.


---
---
---


Great! Let's now **simulate a transaction with an error**, so you can see how **MongoDB automatically rolls back** when something goes wrong.

---

## 🧪 Test Case: Simulate Error in a MongoDB Transaction

We’ll simulate:

* Deducting money from Alice
* Adding money to **a user that does not exist** (error)
* Logging the transaction
* Watch it **fail and rollback**

---

### 📝 Step-by-Step Code to Simulate Error (Run in `mongosh`):

```js
// 1. Start a session
session = db.getMongo().startSession()

// 2. Attach to database with the session
sessionDB = session.getDatabase("bank")

try {
    // 3. Start the transaction
    session.startTransaction()

    // 4. Deduct ₹200 from Alice (valid)
    sessionDB.accounts.updateOne(
        { _id: 1 },
        { $inc: { balance: -200 } }
    )

    // 5. Add ₹200 to a non-existent user (_id: 9999) — this will not match any document!
    const result = sessionDB.accounts.updateOne(
        { _id: 9999 },  // ERROR: No such user
        { $inc: { balance: 200 } }
    )

    // 6. Check if the user was not found
    if (result.matchedCount === 0) {
        throw new Error("Recipient not found!")
    }

    // 7. Log the transaction
    sessionDB.transactions_log.insertOne({
        from: "Alice",
        to: "Unknown",
        amount: 200,
        date: new Date()
    })

    // 8. Commit
    session.commitTransaction()
    print("✅ Transaction committed")

} catch (error) {
    // 9. Abort transaction
    session.abortTransaction()
    print("❌ Transaction aborted: " + error.message)

} finally {
    // 10. End session
    session.endSession()
}
```

---

### 🧾 Expected Output:

```
❌ Transaction aborted: Recipient not found!
```

### 🧼 Result:

* **No changes will be made** to `accounts`
* **No log will be inserted**
* It's like **nothing ever happened**

---

### 🔎 You can verify rollback:

After the failed transaction, check if Alice's balance was unchanged:

```js
db.accounts.find({ _id: 1 })
```

Also check if `transactions_log` is empty:

```js
db.transactions_log.find().pretty()
```

---
---
---

## ✅ 1. Successful Transaction (Everything Works)

This will:

* Deduct ₹300 from Alice (`_id: 1`)
* Add ₹300 to Bob (`_id: 2`)
* Log it in `transactions_log`

### 🔧 Code:

```js
session = db.getMongo().startSession()
sessionDB = session.getDatabase("bank")

try {
    session.startTransaction()

    sessionDB.accounts.updateOne(
        { _id: 1 },
        { $inc: { balance: -300 } }
    )

    sessionDB.accounts.updateOne(
        { _id: 2 },
        { $inc: { balance: 300 } }
    )

    sessionDB.transactions_log.insertOne({
        from: "Alice",
        to: "Bob",
        amount: 300,
        date: new Date()
    })

    session.commitTransaction()
    print("✅ Transaction committed successfully")
} catch (error) {
    session.abortTransaction()
    print("❌ Transaction aborted: " + error.message)
} finally {
    session.endSession()
}
```

---

## ❌ 2. Duplicate Key Error (Simulated Failure)

We’ll try to insert a document into `accounts` with an `_id` that already exists (e.g., `_id: 1`).

### 🔧 Code:

```js
session = db.getMongo().startSession()
sessionDB = session.getDatabase("bank")

try {
    session.startTransaction()

    // Valid operation
    sessionDB.accounts.updateOne(
        { _id: 2 },
        { $inc: { balance: 100 } }
    )

    // Invalid: Duplicate _id will cause error
    sessionDB.accounts.insertOne({
        _id: 1,  // Already exists
        name: "Duplicate Alice",
        balance: 500
    })

    session.commitTransaction()
    print("✅ Transaction committed")
} catch (error) {
    session.abortTransaction()
    print("❌ Transaction aborted due to duplicate key: " + error.message)
} finally {
    session.endSession()
}
```

### 🧾 Output:

```
❌ Transaction aborted due to duplicate key: E11000 duplicate key error...
```

---

## 🌐 3. Simulate Network Error (Simulated manually)

MongoDB doesn't let you *fake* a network issue directly via shell, but here’s how you **simulate** the effect:

### 🧪 How to test:

1. Start a transaction with a **long sleep** (e.g., `sleep(60000)`).
2. While sleeping, **disconnect the shell** (e.g., close the terminal).
3. MongoDB **automatically aborts the transaction** after \~60 seconds due to lost session.

---

#### 🧪 Code to Simulate Network Drop:

```js
session = db.getMongo().startSession()
sessionDB = session.getDatabase("bank")

session.startTransaction()
sessionDB.accounts.updateOne({ _id: 1 }, { $inc: { balance: -100 } })
sessionDB.accounts.updateOne({ _id: 2 }, { $inc: { balance: 100 } })

// Now sleep 60 seconds (simulate network issue)
sleep(60000) // <-- or kill shell manually

session.commitTransaction() // likely to fail due to timeout
```

---

## ⏱️ 4. Simulate Timeout Inside a Transaction

MongoDB transactions **must complete within 60 seconds** by default.

Let’s simulate this by **sleeping for too long** in the middle of the transaction.

> 🔔 Use `sleep(ms)` in `mongosh` (where `1000 ms = 1 second`)

### 🔧 Code:

```js
session = db.getMongo().startSession()
sessionDB = session.getDatabase("bank")

try {
    session.startTransaction()

    sessionDB.accounts.updateOne(
        { _id: 1 },
        { $inc: { balance: -100 } }
    )

    print("⏱️ Sleeping for 65 seconds to force timeout...")
    sleep(65000)

    sessionDB.accounts.updateOne(
        { _id: 2 },
        { $inc: { balance: 100 } }
    )

    session.commitTransaction()
    print("✅ Transaction committed")
} catch (error) {
    session.abortTransaction()
    print("❌ Transaction aborted due to timeout: " + error.message)
} finally {
    session.endSession()
}
```

### 🧾 Expected Output:

```
❌ Transaction aborted due to timeout: Transaction  exceeded time limit
```

---

## 🧼 Cleanup Tip (Reset Collections):

To reset your collections before testing again:

```js
use bank
db.accounts.drop()
db.transactions_log.drop()

db.accounts.insertMany([
  { _id: 1, name: "Alice", balance: 1000 },
  { _id: 2, name: "Bob", balance: 500 }
])
```

---

## ✅ Summary Table:

| Case                     | Description                        | Result                    |
| ------------------------ | ---------------------------------- | ------------------------- |
| ✅ Successful Transaction | Valid operations                   | Changes are committed     |
| ❌ Duplicate Key Error    | Insert with duplicate `_id`        | Rolled back               |
| 🌐 Network Simulation    | Drop connection during transaction | Rolled back automatically |
| ⏱️ Timeout               | Sleep too long inside transaction  | Rolled back               |
