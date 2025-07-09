Let’s build a **real-world app-like example** using MongoDB's date operators. We'll simulate a **hotel booking system** and show how to use:

* `$year`, `$month`, `$dateToString`
* `$dateAdd`, `$dateDiff`, `$hour`, `$minute`
* Filtering, formatting, and calculating durations
* Time zone conversion

---

## 🏨 SCENARIO: Hotel Booking System

### Collection: `bookings`

```js
db.bookings.insertMany([
  {
    guestName: "Alice",
    room: "101",
    checkIn: ISODate("2024-07-01T14:00:00Z"),
    checkOut: ISODate("2024-07-05T11:00:00Z")
  },
  {
    guestName: "Bob",
    room: "102",
    checkIn: ISODate("2024-07-03T15:30:00Z"),
    checkOut: ISODate("2024-07-04T10:00:00Z")
  },
  {
    guestName: "Charlie",
    room: "103",
    checkIn: ISODate("2024-07-05T12:00:00Z"),
    checkOut: ISODate("2024-07-10T09:00:00Z")
  }
])
```

---

## 🎯 1. SHOW ALL BOOKINGS with Duration in Days and Local Time

### 🔍 Goal:

* Show guest name
* Show `checkIn`, `checkOut` in IST
* Show number of **nights stayed**

### ✅ Query:

```js
db.bookings.aggregate([
  {
    $project: {
      guestName: 1,
      room: 1,
      checkInIST: {
        $dateToString: {
          date: "$checkIn",
          format: "%Y-%m-%d %H:%M:%S",
          timezone: "Asia/Kolkata"
        }
      },
      checkOutIST: {
        $dateToString: {
          date: "$checkOut",
          format: "%Y-%m-%d %H:%M:%S",
          timezone: "Asia/Kolkata"
        }
      },
      nightsStayed: {
        $dateDiff: {
          startDate: "$checkIn",
          endDate: "$checkOut",
          unit: "day"
        }
      }
    }
  }
])
```

### 🧾 Output:

```json
{
  guestName: "Alice",
  room: "101",
  checkInIST: "2024-07-01 19:30:00",
  checkOutIST: "2024-07-05 16:30:00",
  nightsStayed: 4
}
```

---

## 📊 2. GET BOOKINGS IN JULY 2024

### ✅ Query:

```js
db.bookings.aggregate([
  {
    $match: {
      checkIn: {
        $gte: ISODate("2024-07-01T00:00:00Z"),
        $lt: ISODate("2024-08-01T00:00:00Z")
      }
    }
  },
  {
    $project: {
      guestName: 1,
      checkIn: 1,
      checkOut: 1
    }
  }
])
```

> Useful for filtering by **month**.

---

## 📆 3. GROUP BOOKINGS BY DATE (`%Y-%m-%d`) — for Daily Report

```js
db.bookings.aggregate([
  {
    $group: {
      _id: {
        date: {
          $dateToString: {
            date: "$checkIn",
            format: "%Y-%m-%d",
            timezone: "Asia/Kolkata"
          }
        }
      },
      count: { $sum: 1 },
      guests: { $push: "$guestName" }
    }
  },
  {
    $sort: { "_id.date": 1 }
  }
])
```

### 🧾 Output:

```json
{
  _id: { date: "2024-07-01" },
  count: 1,
  guests: ["Alice"]
}
```

---

## ⌛ 4. UPCOMING BOOKINGS (Next 3 Days from Today)

```js
db.bookings.aggregate([
  {
    $match: {
      checkIn: {
        $gte: "$$NOW",
        $lt: {
          $dateAdd: {
            startDate: "$$NOW",
            unit: "day",
            amount: 3
          }
        }
      }
    }
  },
  {
    $project: {
      guestName: 1,
      room: 1,
      checkInLocal: {
        $dateToString: {
          date: "$checkIn",
          format: "%Y-%m-%d %H:%M",
          timezone: "Asia/Kolkata"
        }
      }
    }
  }
])
```

---

## 🧠 BONUS: Calculate Check-Out Time in Local Time

If you store check-in and want to **auto-calculate check-out after 2 nights**:

```js
db.bookings.aggregate([
  {
    $project: {
      guestName: 1,
      checkIn: 1,
      estimatedCheckout: {
        $dateAdd: {
          startDate: "$checkIn",
          unit: "day",
          amount: 2
        }
      },
      estimatedCheckoutLocal: {
        $dateToString: {
          date: {
            $dateAdd: {
              startDate: "$checkIn",
              unit: "day",
              amount: 2
            }
          },
          format: "%Y-%m-%d %H:%M:%S",
          timezone: "Asia/Kolkata"
        }
      }
    }
  }
])
```

---

## 🧾 Summary — Real Use of Operators

| Operator        | What it did in app                  |
| --------------- | ----------------------------------- |
| `$dateToString` | Show human-readable local times     |
| `$dateDiff`     | Count nights stayed                 |
| `$dateAdd`      | Calculate check-out or future dates |
| `$year/$month`  | Group or filter bookings by month   |
| `$match`        | Find bookings in date range         |
| `$group`        | Aggregate bookings by day           |
| `$project`      | Customize fields to return          |
