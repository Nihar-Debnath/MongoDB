## 🔒 What is a Capped Collection?

A **capped collection** is a **fixed-size** collection in MongoDB that:

* Stores documents in **insertion order**.
* **Automatically overwrites** the **oldest** documents when the size limit is reached.
* **Pre-allocates** disk space for performance.

> 💡 Think of it like a **circular queue or log file**.

---

## 🧠 Key Features

| Feature                 | Description                                                                     |
| ----------------------- | ------------------------------------------------------------------------------- |
| 🔁 **Fixed Size**       | You define a max size (in bytes), and MongoDB does not let it grow beyond that. |
| 🧹 **Auto-deletion**    | Oldest documents are deleted when the limit is reached.                         |
| 📥 **Insert-Only**      | You cannot delete or update the size of documents (only update in-place).       |
| 🏃 **High Performance** | Very fast inserts and reads — no index updates or fragmentation.                |
| 🧾 **Insertion Order**  | Documents are always read in the same order as inserted.                        |

---

## ✅ Why Use Capped Collections?

They are **ideal for logs or real-time data** where:

* You only care about the **latest N records**.
* You want **fast performance**.
* You don't need to **query old records** forever.

---

## 📦 Real-World Examples

| Use Case          | Description                                      |
| ----------------- | ------------------------------------------------ |
| **Server logs**   | Store the latest logs (error, access logs, etc.) |
| **Chat messages** | Show only recent messages (like the last 1000)   |
| **Sensor data**   | Keep only the last 24 hours of IoT readings      |
| **Audit trails**  | Show latest user activities (login, logout)      |

---

## 🛠 How to Create a Capped Collection

```js
db.createCollection("logs", {
  capped: true,
  size: 1048576, // 1MB max size
  max: 1000       // (optional) max 1000 documents
})
```

You can specify:

* `size`: the maximum total size in **bytes**.
* `max`: (optional) max number of documents.

---

## 📥 Insert Example

```js
db.logs.insertOne({ time: new Date(), message: "Server started" })
```

Keep inserting logs... Once the 1MB limit or 1000 doc limit is hit, **old logs will be auto-deleted**.

---

## 🔍 Query Example

```js
db.logs.find().sort({ $natural: -1 }).limit(10)
```

> `$natural` means order of **insertion** — fastest way to get recent data.

---

## ⚠️ Limitations

| Limitation                            | Why?                                                             |
| ------------------------------------- | ---------------------------------------------------------------- |
| ❌ Cannot delete docs manually         | It’s managed by MongoDB automatically                            |
| ❌ Cannot increase size after creation | You must create a new capped collection                          |
| ❌ No document growth                  | You can't update a doc to be larger than original size           |
| ❌ Limited indexes                     | You can use indexes, but usually they’re avoided for performance |

---

## ✅ Summary

| Property     | Value                             |
| ------------ | --------------------------------- |
| Type         | Fixed-size collection             |
| Deletes data | Yes (oldest auto-deleted)         |
| Use case     | Logs, real-time feeds, chat, etc. |
| Performance  | Very high for writes and reads    |
| Insert order | Always maintained                 |

---

### 🧠 Analogy:

> A capped collection is like a **CCTV DVR**: it only keeps the **latest footage**, and **automatically deletes** the oldest when space runs out.

---
---
---
---



## 🔁 **Capped Collection Deep Dive**

A **capped collection** is like a **fixed-size bucket**:

* It can hold **only a specific number of documents** or **a fixed amount of bytes**.
* When it’s full, it **automatically removes the oldest** documents to make room for new ones.
* You **cannot delete** documents manually.
* Documents are **always stored in insertion order**.

---

## ❓ Can an existing collection be capped later?

**No**, you **cannot convert** a normal collection into capped **directly**.

### But there’s a workaround:

You can **copy** an existing collection into a new capped collection:

```js
// Step 1: Create a capped collection
db.createCollection("logs_capped", { capped: true, size: 1024, max: 5 })

// Step 2: Copy existing data into it
db.original_collection.find().forEach(function(doc) {
  db.logs_capped.insert(doc);
});
```

---

## 📦 Let's Do Your Example (Max 5 Documents)

### Step 1: Create a capped collection that holds **only 5 documents**:

```js
db.createCollection("test_logs", {
  capped: true,
  size: 1024, // small size to simulate limit
  max: 5      // max 5 documents allowed
})
```

---

### Step 2: Insert 6 documents (1 by 1)

```js
db.test_logs.insertOne({ msg: "Log 1" });
db.test_logs.insertOne({ msg: "Log 2" });
db.test_logs.insertOne({ msg: "Log 3" });
db.test_logs.insertOne({ msg: "Log 4" });
db.test_logs.insertOne({ msg: "Log 5" });
```

### Now check the collection:

```js
db.test_logs.find();
```

✅ Output:

```js
{ msg: "Log 1" }
{ msg: "Log 2" }
{ msg: "Log 3" }
{ msg: "Log 4" }
{ msg: "Log 5" }
```

---

### Step 3: Insert the 6th document

```js
db.test_logs.insertOne({ msg: "Log 6" });
```

Now again check:

```js
db.test_logs.find();
```

✅ Output:

```js
{ msg: "Log 2" }
{ msg: "Log 3" }
{ msg: "Log 4" }
{ msg: "Log 5" }
{ msg: "Log 6" }
```

> 🚨 **Log 1 is gone!**
> The oldest doc was **automatically removed** to make room for the new one.

---

## 🔍 Visual Summary

| Insertion Order | Collection Contents                          |
| --------------- | -------------------------------------------- |
| Insert Log 1    | Log 1                                        |
| Insert Log 2    | Log 1, Log 2                                 |
| Insert Log 3    | Log 1, Log 2, Log 3                          |
| Insert Log 4    | Log 1, Log 2, Log 3, Log 4                   |
| Insert Log 5    | Log 1, Log 2, Log 3, Log 4, Log 5            |
| Insert Log 6    | 🚫 Log 1 is removed → Log 2 to Log 6 remains |

---

## ✅ Summary

| Feature                    | Details                           |
| -------------------------- | --------------------------------- |
| Fixed size                 | Max documents or bytes            |
| Auto-delete                | Oldest docs removed automatically |
| Convert existing to capped | ❌ Not directly (but can copy)     |
| Use case                   | Logs, chat, activity feed         |
| Limitations                | No manual delete, fixed doc size  |
