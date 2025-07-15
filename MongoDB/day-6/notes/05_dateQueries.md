## 🔥 WHY Date Queries Are Important?

Dates are essential in MongoDB for:

* Finding records by year/month/day
* Grouping by date/time
* Formatting timestamps
* Scheduling events, etc.

MongoDB stores dates in **ISODate format** (like `2024-07-09T12:34:56.789Z`).

---

## 📁 Sample Collection (`events`)

We'll use this collection:

```js
db.events.insertMany([
  { name: "Meeting",    startTime: ISODate("2024-07-09T10:00:00Z") },
  { name: "Workshop",   startTime: ISODate("2024-07-15T12:30:00Z") },
  { name: "Seminar",    startTime: ISODate("2024-08-02T09:45:00Z") },
  { name: "Conference", startTime: ISODate("2023-12-25T15:00:00Z") }
])
```

---

# 📅 1. `$year`, `$month`, `$dayOfMonth` — Extract Parts of Date

These are **aggregation operators** used inside `$project` or `$group`.

### 💡 Use in `$project` to extract parts:

```js
db.events.aggregate([
  {
    $project: {
      name: 1,
      year: { $year: "$startTime" },
      month: { $month: "$startTime" },
      day: { $dayOfMonth: "$startTime" }
    }
  }
])
```

### ✅ Output:

```json
{ name: "Meeting",    year: 2024, month: 7, day: 9 }
{ name: "Workshop",   year: 2024, month: 7, day: 15 }
{ name: "Seminar",    year: 2024, month: 8, day: 2 }
{ name: "Conference", year: 2023, month: 12, day: 25 }
```

---

# 📊 2. `$group` by Month or Year

### 💡 Count events by month:

```js
db.events.aggregate([
  {
    $group: {
      _id: {
        year: { $year: "$startTime" },
        month: { $month: "$startTime" }
      },
      count: { $sum: 1 }
    }
  }
])
```

### ✅ Output:

```json
{ _id: { year: 2024, month: 7 }, count: 2 }
{ _id: { year: 2024, month: 8 }, count: 1 }
{ _id: { year: 2023, month: 12 }, count: 1 }
```

---

# 📆 3. `$dateToString` — Format Date as a String

### 💡 Format `startTime` like "2024-07-09":

```js
db.events.aggregate([
  {
    $project: {
      name: 1,
      formattedDate: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$startTime"
        }
      }
    }
  }
])
```

### ✅ Output:

```json
{ name: "Meeting", formattedDate: "2024-07-09" }
```

### 🧠 Common Format Tokens:

| Format              | Output       |
| ------------------- | ------------ |
| `%Y`                | Year (2024)  |
| `%m`                | Month (07)   |
| `%d`                | Day (09)     |
| `%H:%M:%S`          | Time (24-hr) |
| `%Y-%m-%dT%H:%M:%S` | Full ISO     |

---

# ⌚ 4. Filter Using Dates (`$gte`, `$lt`, etc.)

### 💡 Find events after July 10, 2024:

```js
db.events.find({
  startTime: {
    $gte: ISODate("2024-07-10T00:00:00Z")
  }
})
```

---

# ⌛ 5. `$dateDiff`, `$subtract`, `$add` — Date Math

### 💡 Find days until each event:

```js
db.events.aggregate([
  {
    $project: {
      name: 1,
      daysLeft: {
        $dateDiff: {
          startDate: "$$NOW",     // Current date/time
          endDate: "$startTime",
          unit: "day"
        }
      }
    }
  }
])
```

> Requires MongoDB **5.0+**.

---

# ⏱ 6. `$dateAdd` / `$dateSubtract` — Add/Subtract Time

### 💡 Add 7 days to `startTime`:

```js
db.events.aggregate([
  {
    $project: {
      name: 1,
      newDate: {
        $dateAdd: {
          startDate: "$startTime",
          unit: "day",
          amount: 7
        }
      }
    }
  }
])
```

---

# 🔍 7. Use in `$match` With Ranges

### 💡 Events in July 2024:

```js
db.events.aggregate([
  {
    $match: {
      startTime: {
        $gte: ISODate("2024-07-01T00:00:00Z"),
        $lt: ISODate("2024-08-01T00:00:00Z")
      }
    }
  }
])
```

---

# 🧠 Bonus: `$isoWeekYear`, `$isoDayOfWeek`

These are based on **ISO 8601 week definitions**.

```js
$isoWeekYear  // e.g., 2024
$isoWeek      // e.g., 27
$isoDayOfWeek // e.g., 1 (Monday) to 7 (Sunday)
```

---

# 🔚 Summary Table

| Operator        | Purpose                   | Example                  |
| --------------- | ------------------------- | ------------------------ |
| `$year`         | Extract year from date    | `$year: "$startTime"`    |
| `$month`        | Extract month from date   | `$month: "$startTime"`   |
| `$dateToString` | Format date as string     | `format: "%Y-%m-%d"`     |
| `$dateDiff`     | Time between two dates    | `unit: "day"`            |
| `$dateAdd`      | Add time to a date        | `amount: 7, unit: "day"` |
| `$dateSubtract` | Subtract time from a date |                          |
| `$isoWeek`      | ISO week number           | `$isoWeek: "$startTime"` |
| `$match`        | Filter by date ranges     | `$gte`, `$lt`, etc.      |

---
---
---


## 🧠 1. What is `ISODate` in MongoDB?

MongoDB stores all date values in **UTC (Coordinated Universal Time)** internally.

```js
ISODate("2024-07-09T10:00:00Z")
```

* The **`Z`** at the end stands for **Zulu Time**, which is UTC+00:00.
* Even if your system or client is in a different time zone, **MongoDB always stores dates in UTC**.

---

## 🕒 2. How Time Zones Work in `ISODate`

Here’s how time zones affect `ISODate`:

| Your Time Zone | Local Time (Your Clock) | Stored UTC (`ISODate`)            |
| -------------- | ----------------------- | --------------------------------- |
| IST (UTC+5:30) | 2024-07-09 15:30        | `ISODate("2024-07-09T10:00:00Z")` |
| PST (UTC-7:00) | 2024-07-09 03:00        | `ISODate("2024-07-09T10:00:00Z")` |

📌 Regardless of your timezone, MongoDB stores it as **UTC**, but you **view/format/convert** it to your local time when needed.

---

## 🔍 3. How to *View* and *Format* With Time Zone

### ✅ a. Use `$dateToString` with `timezone` option in Aggregation:

```js
db.events.aggregate([
  {
    $project: {
      localTime: {
        $dateToString: {
          date: "$startTime",
          format: "%Y-%m-%d %H:%M:%S",
          timezone: "Asia/Kolkata"  // IST (UTC+5:30)
        }
      }
    }
  }
])
```

### ✅ b. Other Example Time Zones

| City          | Timezone ID           |
| ------------- | --------------------- |
| New York      | `America/New_York`    |
| Los Angeles   | `America/Los_Angeles` |
| London        | `Europe/London`       |
| Kolkata       | `Asia/Kolkata`        |
| UTC (default) | `UTC`                 |

---

## 🛠️ 4. How to Insert Local Date as UTC

If you do:

```js
new Date("2024-07-09T10:00:00")
```

JavaScript interprets it as **local time**, and MongoDB **converts it to UTC** when storing.

Example (from India, IST):

```js
new Date("2024-07-09T10:00:00")  // => stored as "2024-07-09T04:30:00Z"
```

To insert UTC directly:

```js
ISODate("2024-07-09T10:00:00Z")  // exact UTC time
```

---

## 🔍 5. Check Your System’s Timezone in `mongosh`

```js
new Date()  // Shows current time in your local timezone
```

Try this:

```js
print(new Date().toString()) // e.g., "Tue Jul 09 2024 15:30:00 GMT+0530 (India Standard Time)"
```

But:

```js
print(ISODate()) // Always in UTC
```

---

## 🧪 6. Querying Based on Time Zone

To query events that happened on **2024-07-09 IST (00:00 to 23:59)**:

```js
db.events.find({
  startTime: {
    $gte: ISODate("2024-07-08T18:30:00Z"),  // 00:00 IST
    $lt: ISODate("2024-07-09T18:30:00Z")    // 00:00 next day IST
  }
})
```

> Because IST is **UTC+5:30**, subtract 5.5 hours to get UTC time range.

---

## 💡 7. Handy Trick: Visualize Timezones

Use this small function in `mongosh`:

```js
function showTimeZones() {
  const zones = ["UTC", "Asia/Kolkata", "America/New_York", "Europe/London"]
  zones.forEach(zone => {
    print(zone + ": " + new Date().toLocaleString("en-US", { timeZone: zone }))
  })
}
showTimeZones()
```

This will show the current time in different time zones side by side.

---

## 🧠 Summary

| Topic                   | Key Point                                                      |
| ----------------------- | -------------------------------------------------------------- |
| `ISODate()`             | Stores time in UTC                                             |
| Timezone conversion     | Use `$dateToString` with `timezone` option                     |
| Local to UTC conversion | MongoDB auto-converts your local date to UTC                   |
| Query in timezone       | Manually adjust UTC offsets for `$gte`/`$lt` queries           |
| View local time         | Use `new Date().toString()` or `$dateToString` with `timezone` |



---
---
---


Let's fully and properly understand the **date extraction operators** in MongoDB aggregation — like `$year`, `$month`, `$week`, `$hour`, `$minute`, `$second`, and `$millisecond`.

These operators are used to **extract specific parts** of a date (like "just the year" or "just the hour") during aggregation queries.

---

## 📦 Sample Document (Reference)

Let's assume this document:

```js
{
  name: "Test Event",
  date: ISODate("2024-07-09T14:25:36.123Z")
}
```

We’ll use this to explain what each operator returns.

---

## 📌 Syntax (Common for All)

All of these are used **inside aggregation pipelines**, in stages like `$project` or `$group`.

```js
{
  $project: {
    valueYouWant: { $operatorName: "$dateField" }
  }
}
```

---

# 📅 `$year`

### 🔹 Purpose: Extracts **year** from the date.

```js
{ $year: "$date" }
```

### 🧾 Output (from above sample):

```
2024
```

---

# 📆 `$month`

### 🔹 Purpose: Extracts **month number** (1–12).

```js
{ $month: "$date" }
```

### 🧾 Output:

```
7
```

✅ Note: July = 7

---

# 🗓️ `$week`

### 🔹 Purpose: Extracts **ISO week number** (1–53).

```js
{ $week: "$date" }
```

### 🧾 Output:

```
28
```

✅ Week 28 of 2024.

⚠️ **This is deprecated** in newer MongoDB versions. Use `$isoWeek` instead for ISO-compliant week numbers.

---

# 🕐 `$hour`

### 🔹 Purpose: Extracts **hour** (0–23) from time (UTC by default).

```js
{ $hour: "$date" }
```

### 🧾 Output:

```
14
```

✅ Means 2:00 PM UTC.

---

# ⏱️ `$minute`

### 🔹 Purpose: Extracts **minutes** (0–59).

```js
{ $minute: "$date" }
```

### 🧾 Output:

```
25
```

---

# ⏲️ `$second`

### 🔹 Purpose: Extracts **seconds** (0–59).

```js
{ $second: "$date" }
```

### 🧾 Output:

```
36
```

---

# ⚡ `$millisecond`

### 🔹 Purpose: Extracts **milliseconds** (0–999).

```js
{ $millisecond: "$date" }
```

### 🧾 Output:

```
123
```

✅ From `.123` in `14:25:36.123`

---

## 🧪 Real Aggregation Example

```js
db.events.aggregate([
  {
    $project: {
      name: 1,
      year: { $year: "$date" },
      month: { $month: "$date" },
      week: { $week: "$date" },
      hour: { $hour: "$date" },
      minute: { $minute: "$date" },
      second: { $second: "$date" },
      millisecond: { $millisecond: "$date" }
    }
  }
])
```

---

## 🧠 Important Notes

| Operator       | Range | Notes                       |
| -------------- | ----- | --------------------------- |
| `$year`        | 0–∞   | Full year (e.g., 2024)      |
| `$month`       | 1–12  | Jan = 1, Dec = 12           |
| `$week`        | 1–53  | Based on ISO week           |
| `$hour`        | 0–23  | 24-hour format              |
| `$minute`      | 0–59  |                             |
| `$second`      | 0–59  |                             |
| `$millisecond` | 0–999 | Fractional part of a second |

---

## 🔄 Timezone Handling in These Operators

All these operators extract based on **UTC by default**.

To use **local time**, use the `timezone` option inside `$dateToParts`:

```js
{
  $dateToParts: {
    date: "$date",
    timezone: "Asia/Kolkata"
  }
}
```

This gives **all** parts (year, month, day, hour, etc.) in local time.

---

## ✅ Recommended: `$dateToParts` (MongoDB 4.4+)

Instead of using each one separately:

```js
db.events.aggregate([
  {
    $project: {
      parts: {
        $dateToParts: {
          date: "$date",
          timezone: "Asia/Kolkata"
        }
      }
    }
  }
])
```

### 🧾 Output:

```json
{
  parts: {
    year: 2024,
    month: 7,
    day: 9,
    hour: 19,
    minute: 55,
    second: 36,
    millisecond: 123
  }
}
```

---

## 🧠 Summary Table

| Operator       | Purpose             | Output for "2024-07-09T14:25:36.123Z" |
| -------------- | ------------------- | ------------------------------------- |
| `$year`        | Extract year        | `2024`                                |
| `$month`       | Extract month       | `7`                                   |
| `$week`        | Extract week number | `28`                                  |
| `$hour`        | Extract hour (UTC)  | `14`                                  |
| `$minute`      | Extract minute      | `25`                                  |
| `$second`      | Extract second      | `36`                                  |
| `$millisecond` | Extract millisecond | `123`                                 |




---
---
---




Awesome! You're on a mission to **master MongoDB's date operators** — let's go beyond just `$year`, `$month`, etc., and explore **many more powerful `$date`-related operators** used in aggregation pipelines. These include:

---

## 🔥 FULL LIST OF DATE OPERATORS IN MONGODB (Aggregation)

We’ll organize them into 3 categories:

### 1. 🧠 **Date Component Extractors** (like `$year`)

### 2. ⚙️ **Date Construction and Formatting** (like `$dateToString`)

### 3. 🔁 **Date Arithmetic** (like `$dateAdd`, `$dateDiff`)

---

## 🧠 1. Date Component Extractors

Used to **extract specific parts** from a date.

| Operator        | Description                              | Example Output |
| --------------- | ---------------------------------------- | -------------- |
| `$year`         | Extract year                             | `2024`         |
| `$month`        | Extract month (1–12)                     | `7`            |
| `$dayOfMonth`   | Day in month (1–31)                      | `9`            |
| `$dayOfWeek`    | Day in week (1 = Sunday)                 | `3`            |
| `$dayOfYear`    | Day in year (1–366)                      | `191`          |
| `$week`         | **Deprecated** ISO week (use `$isoWeek`) | `28`           |
| `$isoWeek`      | ISO 8601 week number                     | `28`           |
| `$isoWeekYear`  | ISO week year                            | `2024`         |
| `$isoDayOfWeek` | ISO weekday (1 = Monday to 7 = Sunday)   | `2`            |
| `$hour`         | Hour of day (0–23)                       | `14`           |
| `$minute`       | Minute (0–59)                            | `25`           |
| `$second`       | Seconds (0–59)                           | `36`           |
| `$millisecond`  | Milliseconds (0–999)                     | `123`          |

---

## ⚙️ 2. Date Construction & Formatting

Used to **format**, **parse**, or **decompose** date values.

---

### 🧾 `$dateToString`

Formats a date into a **custom string**.

```js
{
  $dateToString: {
    date: "$createdAt",
    format: "%Y-%m-%d %H:%M:%S",
    timezone: "Asia/Kolkata"
  }
}
```

🧾 Output:

```
"2024-07-09 19:55:36"
```

---

### 🧾 `$dateFromString`

Converts a formatted string **back into a date**.

```js
{
  $dateFromString: {
    dateString: "2024-07-09 14:25:36",
    format: "%Y-%m-%d %H:%M:%S",
    timezone: "UTC"
  }
}
```

🧾 Output:

```
ISODate("2024-07-09T14:25:36Z")
```

---

### 🧾 `$dateToParts`

Breaks a date into **parts (year, month, day, etc.)**.

```js
{
  $dateToParts: {
    date: "$createdAt",
    timezone: "Asia/Kolkata"
  }
}
```

🧾 Output:

```json
{
  year: 2024,
  month: 7,
  day: 9,
  hour: 19,
  minute: 55,
  second: 36,
  millisecond: 123,
  isoWeek: 28
}
```

---

### 🧾 `$dateFromParts`

Creates a date **from components**.

```js
{
  $dateFromParts: {
    year: 2024,
    month: 7,
    day: 9,
    hour: 14,
    minute: 25,
    second: 36
  }
}
```

🧾 Output:

```
ISODate("2024-07-09T14:25:36Z")
```

---

## 🔁 3. Date Arithmetic Operators

Used to **compare**, **add**, or **subtract** dates and times.

---

### 🧾 `$dateAdd`

Adds time to a date.

```js
{
  $dateAdd: {
    startDate: "$startTime",
    unit: "day",
    amount: 7,
    timezone: "UTC"
  }
}
```

🧾 Output: Adds 7 days

---

### 🧾 `$dateSubtract`

Subtracts time from a date.

```js
{
  $dateSubtract: {
    startDate: "$startTime",
    unit: "hour",
    amount: 3
  }
}
```

🧾 Output: Subtracts 3 hours

---

### 🧾 `$dateDiff`

Finds difference between 2 dates.

```js
{
  $dateDiff: {
    startDate: "$startTime",
    endDate: "$endTime",
    unit: "hour"
  }
}
```

🧾 Output:

```
5  // hours between the two
```

---

### 🧾 `$subtract` / `$add`

These are **general math operators**, but they work on dates too!

```js
{ $subtract: [ "$endTime", "$startTime" ] }
```

🧾 Output:

```
1800000  // milliseconds (30 minutes)
```

---

## 🛠 Example: Combine Everything

```js
db.events.aggregate([
  {
    $project: {
      original: "$startTime",
      year: { $year: "$startTime" },
      isoWeek: { $isoWeek: "$startTime" },
      localTimeString: {
        $dateToString: {
          date: "$startTime",
          format: "%Y-%m-%d %H:%M:%S",
          timezone: "Asia/Kolkata"
        }
      },
      added7Days: {
        $dateAdd: {
          startDate: "$startTime",
          unit: "day",
          amount: 7
        }
      },
      daysLeft: {
        $dateDiff: {
          startDate: "$$NOW",
          endDate: "$startTime",
          unit: "day"
        }
      }
    }
  }
])
```

---

## ✅ Summary Cheatsheet (All Operators)

| Category     | Operator                                                     | Purpose            |
| ------------ | ------------------------------------------------------------ | ------------------ |
| Extract      | `$year`, `$month`                                            | Get date parts     |
|              | `$dayOfMonth`, `$hour`, `$minute`, `$second`, `$millisecond` |                    |
| ISO Extract  | `$isoWeek`, `$isoWeekYear`, `$isoDayOfWeek`                  | ISO-standard parts |
| Format       | `$dateToString`                                              | Format to string   |
| Parse        | `$dateFromString`                                            | String to date     |
| Build Date   | `$dateFromParts`                                             | Build from fields  |
| Split Date   | `$dateToParts`                                               | Break into fields  |
| Add/Subtract | `$dateAdd`, `$dateSubtract`                                  | Add/subtract time  |
| Compare      | `$dateDiff`, `$subtract`                                     | Date differences   |


