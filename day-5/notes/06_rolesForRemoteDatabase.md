## ✅ GOAL: Create/manage users (authentication & authorization) for your cluster

---

## 🔐 Step-by-step to access Authentication & Authorization in MongoDB Atlas:

### 🧭 Step 1: Go to **Database Access** (on the left sidebar)

From your screenshot:

* Look on the left under **SECURITY**
* Click on **“Database Access”**

This is where you manage:

* **Database users** (authentication)
* **User roles** (authorization)

---

### 👤 Step 2: Add or Edit a User

Once you're on the **Database Access** page:

#### To **add a new user**:

1. Click **“+ ADD NEW DATABASE USER”**

2. Fill in:

   * **Username**: e.g., `appUser`
   * **Password**: e.g., `securePass123` (remember this)
   * **Database User Privileges**:

     * Select **Built-In Roles** like:

       * `readWrite` for full access to read/write a database
       * `read` for read-only access
     * Choose which database the role applies to (e.g., your main app database)

3. Leave authentication method as `Password`

4. Click **Add User**

> 🔐 This is **authentication**: creating users with usernames and passwords
> 🛡️ And **authorization**: assigning what roles those users have

---

### 🔄 Step 3: Use the new user in Compass

Now that the user is created:

1. Open **MongoDB Compass**

2. Click **"New Connection"**

3. Fill in connection info:

   * **URI Format**:

     ```bash
     mongodb+srv://appUser:securePass123@cluster0.xxxxxx.mongodb.net/
     ```
   * Or fill manually:

     * Host: your cluster hostname (e.g., `cluster0.xxxxxx.mongodb.net`)
     * Authentication: `Username/Password`
     * Username: `appUser`
     * Password: `securePass123`
     * Auth Source: your database name or `admin`

4. Click **Connect**

> ✅ You are now authenticated and authorized based on the role you gave the user.

---





---
---
---



## 🔎 How to Verify Role from Compass:

You can run this to check what roles are applied:

```js
db.runCommand({ connectionStatus: 1 })
```

What you're seeing is a **collapsed version** of the output. MongoDB Compass Shell (and some Node.js console tools) shorten nested objects like `[Object]` to keep output clean.

To fully see what's inside the objects, you can do one of the following:

---

## ✅ Option 1: Use `JSON.stringify()` to expand it

Run this instead:

```js
JSON.stringify(db.runCommand({ connectionStatus: 1 }), null, 2)
```

* `null, 2` means:

  * Don’t modify the data (`null`)
  * Format it nicely with 2-space indentation (`2`)

It will show full details like:

```json
{
  "authInfo": {
    "authenticatedUsers": [
      {
        "user": "yourUser",
        "db": "admin"
      }
    ],
    "authenticatedUserRoles": [
      {
        "role": "readWrite",
        "db": "yourDatabase"
      }
    ]
  },
  "ok": 1
}
```

---

## ✅ Option 2: Access only what you want

You can just extract what you care about:

```js
db.runCommand({ connectionStatus: 1 }).authInfo.authenticatedUserRoles
```

or:

```js
db.runCommand({ connectionStatus: 1 }).authInfo.authenticatedUsers
```

---

## ✅ Option 3: Assign to variable and inspect

```js
const result = db.runCommand({ connectionStatus: 1 });
console.log(JSON.stringify(result, null, 2));
```

This is especially useful when running inside Node.js or a script.
