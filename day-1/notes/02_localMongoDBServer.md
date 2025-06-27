## ✅ 1. Install MongoDB on Windows

### 📦 Step-by-step:

1. **Download MongoDB** (Community Edition):

   * [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
   * Choose MSI version for Windows.

2. **Run Installer**:

   * Choose “**Complete**” setup.
   * Check the box **"Install MongoDB as a Service"**.

     * This sets it up to run as `MongoDB` Windows service.
   * Also check “Install MongoDB Compass” if you want the GUI.

3. **Set Data and Log Directories (optional)**:

   * By default:

     * Data dir: `C:\Program Files\MongoDB\Server\6.0\data\db` *(or similar based on version)*
     * Log dir: `C:\Program Files\MongoDB\Server\6.0\log\mongod.log`

---

## ✅ 2. Start MongoDB Server

### ▶️ To start MongoDB service:

```bash
net start MongoDB
```

* This starts the MongoDB server in the background.
* By default, it listens at port **27017**.

You can now connect using:

```bash
mongo
```

or, for new versions:

```bash
mongosh
```

---

## 🛑 3. Stop MongoDB Server

### ⛔ To stop the MongoDB service:

```bash
net stop MongoDB
```

---

## 🔄 4. Restart MongoDB (optional)

```bash
net stop MongoDB
net start MongoDB
```


---
---
---

## For docker mongodb will it cause a conflict for same port , if yes then how can i resolve that?

Yes, if you’re **running MongoDB both locally (via Windows service)** and **in Docker**, and **both try to use the same port (27017)** on the **same host machine**, it **will cause a conflict**.

---

## 🔥 Problem: Port Conflict

Both the local MongoDB and Docker MongoDB try to use port `27017` on your computer (`localhost`). Only **one process can bind to a port at a time**, so you’ll get an error like:

```
Error starting userland proxy: listen tcp 0.0.0.0:27017: bind: address already in use.
```

---

## ✅ Solution: Change the Exposed Port in Docker

You can **map a different host port** to container's MongoDB port using the `-p` flag:

```bash
docker run -d --name mongo2 -p 27018:27017 mongo
```

* Here:

  * `27018` is the port on **your local machine**.
  * `27017` is the default port **inside the Docker container**.

Now, you can access this Docker MongoDB instance at:

```
mongodb://localhost:27018
```

---

## 🔁 Alternate Option: Stop Local MongoDB Temporarily

If you don’t need the local MongoDB running:

```bash
net stop MongoDB
```

Then run Docker MongoDB on the default port:

```bash
docker run -d --name mongo -p 27017:27017 mongo
```

---

## 🧠 Tip: Accessing MongoDB from Compass or Code

* Local MongoDB (Windows service):

  ```
  mongodb://localhost:27017
  ```

* Docker MongoDB (remapped to 27018):

  ```
  mongodb://localhost:27018
  ```

You can run both simultaneously if they use **different host ports**.