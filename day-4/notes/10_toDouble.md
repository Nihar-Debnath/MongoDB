### 📌 What is `$toDouble`?

`$toDouble` is an **aggregation operator** in MongoDB used to **convert values to the `double` (floating-point) data type** during aggregation operations.

It’s commonly used inside a **`$project`**, **`$addFields`**, or **expression** to convert a value (like a string or int) into a `double`.

---

### 🔧 Syntax:

```js
{ $toDouble: <expression> }
```

* `<expression>` can be a field (`"$price"`), a computed expression, or a literal value (`"45.67"`).

---

### ✅ Example 1: Convert a string field to double

Suppose you have this document:

```js
{ _id: 1, price: "45.67" }
```

And you run:

```js
db.items.aggregate([
  {
    $project: {
      originalPrice: "$price",
      priceAsDouble: { $toDouble: "$price" }
    }
  }
])
```

📦 Output:

```js
{ _id: 1, originalPrice: "45.67", priceAsDouble: 45.67 }
```

* `"45.67"` (a **string**) is converted to `45.67` (a **double**).

---

### ✅ Example 2: Arithmetic after conversion

Let's say your price field is stored as a string:

```js
{ price: "120.50", discount: 0.2 }
```

And you want to calculate discounted price. You must first convert it:

```js
db.products.aggregate([
  {
    $project: {
      price: 1,
      discount: 1,
      numericPrice: { $toDouble: "$price" },
      finalPrice: {
        $multiply: [
          { $toDouble: "$price" },
          { $subtract: [1, "$discount"] }
        ]
      }
    }
  }
])
```

This calculates:
`finalPrice = (price as double) * (1 - discount)`

---

### 🔁 What kinds of values can `$toDouble` convert?

| From Type          | Convertible? | Notes                                   |
| ------------------ | ------------ | --------------------------------------- |
| String             | ✅ Yes        | Must be a numeric string like `"12.34"` |
| Integer            | ✅ Yes        | Will be converted to float              |
| Boolean            | ✅ Yes        | `true → 1.0`, `false → 0.0`             |
| Date/ObjectId      | ❌ No         | Not allowed                             |
| Null               | ✅ Yes        | Converts to `null`                      |
| Non-numeric string | ❌ No         | Will throw error                        |

---

### ⚠️ Important Notes:

* If the value **cannot** be converted (like `"abc"` or a complex object), it will **fail with an error**.
* You can use `$convert` for **safe conversion with error handling**, like this:

```js
{ 
  $convert: {
    input: "$price",
    to: "double",
    onError: 0.0,
    onNull: 0.0
  }
}
```

---

### 🧠 When to use `$toDouble`?

* When numeric values are stored as strings (e.g., from imported CSV/JSON data)
* When performing arithmetic operations in aggregation pipelines
* When ensuring consistent types for sorting or comparison

---

### ✅ Summary:

| Feature       | Description                                     |
| ------------- | ----------------------------------------------- |
| What it does  | Converts a value to double (floating point)     |
| Usage context | Inside `$project`, `$addFields`, or expressions |
| Input types   | Strings, integers, booleans, etc.               |
| Errors        | Happens if input is non-convertible             |
