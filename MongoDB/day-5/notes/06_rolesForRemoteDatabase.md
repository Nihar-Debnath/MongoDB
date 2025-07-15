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



---
---
---


In MongoDB, **roles** are central to **authorization** (what a user can do) after a user is **authenticated** (verified with username/password or other methods).
MongoDB uses a **Role-Based Access Control (RBAC)** system.
---

## 🔐 PART 1: Authentication — *“Who are you?”*

This just checks:

* **Username**
* **Password**
* Optional: x.509 certificate, LDAP, or Kerberos (in enterprise setups)

> After the user is authenticated, MongoDB checks the **roles** to decide **what they're allowed to do**.

---

## 🛡️ PART 2: Authorization — *“What can you do?”*

### 🔷 MongoDB has two kinds of roles:

1. **Built-in roles** (predefined by MongoDB)
2. **Custom roles** (you define them with custom privileges)

---

## ✅ 1. **Built-in Database User Roles**

These are the most commonly used roles when working with apps:

| Role Name              | Where it Applies  | What it Allows                                                  |
| ---------------------- | ----------------- | --------------------------------------------------------------- |
| `read`                 | Specific database | Can only read data (find, aggregate)                            |
| `readWrite`            | Specific database | Can read and write (insert, update, delete, create collections) |
| `dbAdmin`              | Specific database | Can manage indexes, stats, validate collections                 |
| `userAdmin`            | Specific database | Can create and modify roles/users on that DB                    |
| `dbOwner`              | Specific database | All of the above — full control of the DB                       |
| `readWriteAnyDatabase` | All databases     | Like `readWrite`, but for every database                        |
| `userAdminAnyDatabase` | All databases     | Can manage users across all databases                           |
| `dbAdminAnyDatabase`   | All databases     | Admin for all DBs                                               |
| `root`                 | All databases     | Superuser (does everything) — not in Atlas                      |

---

### 🎯 Example Use Case:

Let’s say you have a database called `myAppDB`.

#### 🔸 App User (for backend code):

```json
{
  "role": "readWrite",
  "db": "myAppDB"
}
```

This user can read and write documents — perfect for a Node.js app.

#### 🔸 Analytics User (for dashboard):

```json
{
  "role": "read",
  "db": "myAppDB"
}
```

This user can query data but **not modify** anything.

#### 🔸 Admin User:

```json
{
  "role": "dbOwner",
  "db": "myAppDB"
}
```

Can do everything inside `myAppDB`: manage users, roles, collections, etc.

---

## 🧠 2. **Cluster-Level Roles** (used in Atlas or advanced setups)

These roles give power **across databases** or affect the whole cluster.

| Role             | What it Does                                                   |
| ---------------- | -------------------------------------------------------------- |
| `clusterAdmin`   | Full access to cluster settings (indexes, replication, shards) |
| `clusterMonitor` | Can monitor but not modify cluster (used for dashboards)       |
| `backup`         | Can run backup/restore operations                              |
| `restore`        | Can restore data to cluster                                    |

---

## 🧰 3. **Atlas-Specific Roles** (if using MongoDB Atlas)

Atlas wraps many MongoDB roles into simpler roles, such as:

| Role                         | Permissions                          |
| ---------------------------- | ------------------------------------ |
| `atlasAdmin`                 | Full access to all cluster resources |
| `readWriteAnyDatabase@admin` | Same as the MongoDB built-in one     |
| `readAnyDatabase@admin`      | Can read any database                |
| `userAdmin@admin`            | Can manage users across all DBs      |

> In your earlier screenshot, your user has:
> `atlasAdmin@admin` → this includes full cluster and DB privileges.

---

## 🧱 4. **Custom Roles** (Advanced)

You can create a custom role like:

```json
{
  "role": "customReader",
  "privileges": [
    { "resource": { "db": "sales", "collection": "orders" }, "actions": ["find"] }
  ],
  "roles": []
}
```

This allows reading only from the `orders` collection in the `sales` DB.

---

## 🔍 Quick Summary Table

| Role                   | Scope         | Can Do                                   |
| ---------------------- | ------------- | ---------------------------------------- |
| `read`                 | One DB        | Only query (no writes)                   |
| `readWrite`            | One DB        | Query, insert, update, delete            |
| `dbAdmin`              | One DB        | Manage indexes, stats, validate          |
| `userAdmin`            | One DB        | Create/modify users on that DB           |
| `dbOwner`              | One DB        | All of the above                         |
| `readWriteAnyDatabase` | All DBs       | Read/write on all databases              |
| `userAdminAnyDatabase` | All DBs       | Manage users across all databases        |
| `clusterAdmin`         | Whole cluster | Manage replication, sharding, logs       |
| `atlasAdmin`           | Atlas-wide    | All of the above (full power on cluster) |

---

## 💡 Practical Tips

* 👨‍💻 App users → usually get `readWrite` on their working DB
* 🔒 Analytics users → `read` only
* 🧠 Admins → `dbOwner` or `userAdmin`/`dbAdmin`
* ❌ Avoid giving `root` or `atlasAdmin` to your app code unless truly needed

---

# 🔐 1. AUTHENTICATION (Who are you?)

### ✅ What is it?

Authentication is the process of verifying a user’s identity using:

* Username & password
* Or certificate (e.g., x.509)
* Or external system (LDAP, Kerberos)

### 🛠️ How it works:

1. MongoDB checks your credentials (username & password)
2. If they match, you’re “authenticated”
3. Now MongoDB checks what **roles** you're allowed to use (authorization)

---

# 🛡️ 2. AUTHORIZATION (What can you do?)

After successful authentication, MongoDB uses **roles** to check **permissions**.

> 🔑 Roles define what actions a user can perform.

---

# 📋 3. TYPES OF ROLES IN MONGODB

---

## 🔹 A. DATABASE-SPECIFIC ROLES

These roles apply only to a **single database** (e.g., `myAppDB`)

| **Role Name** | **Can Do**                                                                    | **Cannot Do**             | **Use Case**                   |
| ------------- | ----------------------------------------------------------------------------- | ------------------------- | ------------------------------ |
| `read`        | Read documents, run queries (`find`, `aggregate`)                             | Write, update, delete     | Analyst or dashboard user      |
| `readWrite`   | Everything `read` can do + `insert`, `update`, `delete`, `create collections` | Manage users or indexes   | App backend that modifies data |
| `dbAdmin`     | Create indexes, view stats, run validation                                    | Insert/update/delete data | DevOps managing DB performance |
| `userAdmin`   | Create, delete, or modify users in the database                               | Read or write documents   | Admin team setting up users    |
| `dbOwner`     | All of the above (read, write, manage users, indexes)                         | Cluster-level operations  | Full control over one DB only  |

### Example:

```json
{
  "user": "editor",
  "roles": [ { "role": "readWrite", "db": "blogDB" } ]
}
```

---

## 🔹 B. ALL-DATABASE (GLOBAL) ROLES

These roles apply to **all databases** in MongoDB.

| **Role Name**          | **Can Do**               | **Use Case**            |
| ---------------------- | ------------------------ | ----------------------- |
| `readAnyDatabase`      | Read from any DB         | Super analyst           |
| `readWriteAnyDatabase` | Read/write all DBs       | Multi-tenant apps       |
| `dbAdminAnyDatabase`   | Admin actions on all DBs | Global DB administrator |
| `userAdminAnyDatabase` | Manage users in all DBs  | Global user manager     |

### Example:

```json
{
  "user": "globalAdmin",
  "roles": [ { "role": "readWriteAnyDatabase", "db": "admin" } ]
}
```

> These roles are assigned in the **`admin`** database and apply globally.

---

## 🔹 C. CLUSTER ROLES (For managing entire MongoDB clusters)

These are **for operations beyond data**, like replication, backup, and monitoring.

| **Role Name**    | **Can Do**                                   | **Use Case**             |
| ---------------- | -------------------------------------------- | ------------------------ |
| `clusterAdmin`   | Control sharding, replication, server config | Cluster manager          |
| `clusterMonitor` | View logs, stats, replication info           | Monitoring dashboards    |
| `backup`         | Run backups                                  | Automated backup scripts |
| `restore`        | Restore from backup                          | Disaster recovery        |

---

## 🔹 D. SUPERUSER ROLE

| **Role Name** | **What it does**          | **Notes**                                                    |
| ------------- | ------------------------- | ------------------------------------------------------------ |
| `root`        | Full access to everything | Only available in local MongoDB setups, not in MongoDB Atlas |

---

## 🔹 E. MONGODB ATLAS-SPECIFIC ROLES (Cloud Only)

Atlas wraps MongoDB roles for easy access control.

| **Role**           | **Meaning**                                                          |
| ------------------ | -------------------------------------------------------------------- |
| `atlasAdmin@admin` | Superuser of the Atlas project (create clusters, manage users, etc.) |
| `readWrite@yourDB` | Read & write a specific database                                     |
| `read@yourDB`      | Read-only access to a specific DB                                    |
| `userAdmin@admin`  | Manage users via Atlas dashboard                                     |

---

# 💡 Example: Assigning a Role in Atlas UI

To give an app read/write access to a specific database:

1. Go to **Database Access** in Atlas
2. Click **Add New Database User**
3. Set:

   * Username: `appUser`
   * Password: `secure123`
   * Role: `readWrite` on database `appDB`

Now use this in Compass or your app URI:

```
mongodb+srv://appUser:secure123@cluster0.xxxx.mongodb.net/appDB
```

---

# ✅ Summary Table

| **Role**               | **Scope** | **Permission Level**           |
| ---------------------- | --------- | ------------------------------ |
| `read`                 | One DB    | Read only                      |
| `readWrite`            | One DB    | Read and write                 |
| `dbAdmin`              | One DB    | Admin (indexes, stats)         |
| `userAdmin`            | One DB    | Manage users                   |
| `dbOwner`              | One DB    | Full DB control                |
| `readAnyDatabase`      | All DBs   | Read                           |
| `readWriteAnyDatabase` | All DBs   | Read/write                     |
| `clusterAdmin`         | Cluster   | Full system admin              |
| `clusterMonitor`       | Cluster   | Monitor only                   |
| `atlasAdmin`           | Atlas     | Full Atlas control             |
| `root`                 | All       | Superuser (local MongoDB only) |

