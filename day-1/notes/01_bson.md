**BSON** stands for **Binary JSON**. It's a **binary-encoded** serialization format that is used to store and transmit data structures similar to JSON (JavaScript Object Notation), but in a way that is more efficient for computers to parse and generate.

---

### 🔹 Key Features of BSON:

1. **Binary Format**:

   * Unlike plain text JSON, BSON is stored in binary form, which is faster to parse and more compact.

2. **Supports More Data Types**:

   * BSON supports additional data types not present in standard JSON, such as:

     * `int32`, `int64`
     * `float`, `double`
     * `binary data`
     * `ObjectId` (used in MongoDB)
     * `Date`, `Timestamp`
     * `Null`, `Boolean`, `Array`, `Embedded documents`

3. **Efficient Encoding**:

   * It includes **length prefixes** for strings and arrays, making traversal faster and allowing for **efficient skipping** over fields.

4. **Used by MongoDB**:

   * BSON is the primary data storage and network transfer format used by **MongoDB**, a NoSQL database.

---

### 🔸 Example

**JSON Example:**

```json
{
  "name": "Harsh",
  "age": 20
}
```

**BSON Equivalent (human-readable):**

```bson
\x16\x00\x00\x00           // total document size (22 bytes)
\x02                       // field type = string
name\x00                   // field name
\x06\x00\x00\x00Harsh\x00  // string length + string
\x10                       // field type = int32
age\x00\x14\x00\x00\x00    // field name + value (20 in int32)
\x00                       // end of document
```

---

### 🔹 Use Cases

* **MongoDB** storage and wire protocol
* Applications needing **faster data encoding/decoding** than JSON
* Data interchange where **type precision** is important (like 64-bit integers)
