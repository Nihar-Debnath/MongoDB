## 🔷 What is `mongoimport`?

`mongoimport` is a **command-line tool** provided by MongoDB that allows you to **import data from files** (like `.json`, `.csv`, `.tsv`) into a MongoDB database.

It's mainly used for:

* Importing large datasets
* Migrating data between databases
* Testing with mock data

---

## ✅ File Formats Supported

You can import:

* **JSON** (as an array or individual documents)
* **CSV** (comma-separated values)
* **TSV** (tab-separated values)

---

## 🛠️ How to Use `mongoimport` from the Command Line (Local or Cloud)

### ✅ 1. **For Local MongoDB**

If MongoDB is running locally on port `27017` (default), use:

### ◾ JSON File

```bash
mongoimport --db myDatabase --collection myCollection --file data.json --jsonArray
```

Let's break down the `mongoimport` command:

## 🔍 Explanation of Each Part:

| Part                        | Meaning                                                                                     |
| --------------------------- | ------------------------------------------------------------------------------------------- |
| `mongoimport`               | Command-line tool to import data into MongoDB                                               |
| `--db myDatabase`           | Specifies the **database name** where the data should be imported (`myDatabase`)            |
| `--collection myCollection` | Specifies the **collection name** inside the database (`myCollection`)                      |
| `--file data.json`          | The **file path** to the data you want to import (`data.json`)                              |
| `--jsonArray`               | Indicates that the file contains a **JSON array** of documents, not individual JSON objects |

---

### ◾ CSV File

```bash
mongoimport --db myDatabase --collection myCollection --type csv --file data.csv --headerline
```

> ✅ `--headerline` tells MongoDB to use the first row as field names.

---

### ✅ 2. **For Cloud MongoDB (MongoDB Atlas)**

You must use your **MongoDB Atlas connection URI**:

```bash
mongoimport --uri "mongodb+srv://<username>:<password>@cluster0.mongodb.net/myDatabase" \
  --collection myCollection \
  --file data.json \
  --jsonArray
```

> 🟡 Make sure your IP address is **allowed in Atlas**, and you **URL-encode** your password if it contains symbols (like `@`, `#`, etc.).

### 🧠 Example With Quotes:

```bash
mongoimport --db "my Database" --collection "test data" --file "my file.json" --jsonArray
```

Or if you're passing a URI (which has special characters like `@`, `:`), you **must use quotes**:

---

### 🧪 Example

**You have a `students.json` file:**

```json
[
  { "name": "Alice", "age": 22 },
  { "name": "Bob", "age": 24 }
]
```

Import it into `school` database, `students` collection:

```bash
mongoimport --db school --collection students --file students.json --jsonArray
```

---

## 🧭 How to Import Files in **MongoDB Compass**

MongoDB Compass provides a **GUI (graphical way)** to import data without using the command line.

### ✅ Steps:

1. Open **MongoDB Compass**.
2. Connect to your **local MongoDB** or **MongoDB Atlas** cluster.
3. Choose the **Database** and **Collection** you want to import data into.
4. Click **“Add Data” > “Import File”**.
5. Choose:

   * **File Type**: JSON or CSV
   * **Select File**: Pick from your computer
   * If CSV: Toggle **headerline** and delimiter
6. Click **“Import”**.

🎉 Your data will now appear in the collection!

---

## 🧩 Summary Table

| Feature           | `mongoimport` CLI      | Compass GUI Import           |
| ----------------- | ---------------------- | ---------------------------- |
| Format support    | JSON, CSV, TSV         | JSON, CSV                    |
| Easy to use?      | ⚠️ Needs terminal      | ✅ Very beginner-friendly     |
| Local DB support  | ✅ Yes                  | ✅ Yes                        |
| Atlas support     | ✅ Yes with URI         | ✅ Yes                        |
| Batch data import | ✅ Best for large files | 🚫 Slower for large datasets |

---

## 🧰 Tip: Where to Get `mongoimport`

* It's part of the **MongoDB Database Tools**:
  [🔗 Download here](https://www.mongodb.com/try/download/database-tools)
