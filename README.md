# 🧱 InnoDB Cluster Deployment Guide

A step-by-step guide to deploying a **MySQL InnoDB Cluster** using **Docker swarm** and **MySQL Shell**.  
This guide ensures you create a clean, production-ready setup with proper cluster initialization and routing.

---

## 🪄 Prerequisites

Before you begin, make sure you have:

- 🐳 **Docker** and **Docker Swarm** initialized  
- 📦 A valid `stack.yml` file containing your MySQL services  
- ⚙️ Basic familiarity with `mysqlsh` (MySQL Shell)

---

## 🚀 Step 1: Deploy the Docker Stack

Deploy your stack with the following command:

```bash
docker stack deploy -c stack.yml innodb-cluster
````

> ✅ At this stage:
>
> * All **MySQL servers** should have **at least one replica running**
> * The **MySQL Router** must remain **stopped** until the MySQL InnoDB Cluster is created

---

## 💻 Step 2: Access the MySQL Shell

Open the MySQL Shell on one of your MySQL server containers:

```bash
mysqlsh
\js  # Switch to JavaScript mode
```

---

## 🔗 Step 3: Connect to the MySQL Instance

Use the **service name**, not `localhost`, when connecting:

```bash
shell.connect('root@mysql-server-1')
```

> ⚠️ **Do not use `localhost`!**
> The shell may connect, but cluster creation will fail .

---

## 🧩 Step 4: Create the InnoDB Cluster

Once connected, initialize your cluster:

```js
cluster = dba.createCluster('devCluster')
```

This creates your **InnoDB Cluster**.

---

## ➕ Step 5: Add Additional Nodes

Configure and add other MySQL servers (recommended: at least **three nodes**):

```js
dba.addinstance()
```

Repeat for each additional MySQL server you want in the cluster.

---

## 🔍 Step 6: Verify Cluster Status

Check the health and status of your cluster:

```js
var cluster = dba.getCluster()  // For subsequent checks
cluster.status()
```

This displays which node is **Primary (RW)** and which are **Secondary (RO)**.

---

## ⚙️ Step 7: Configure the MySQL Router

Once the cluster is fully functional, you can start your **MySQL Router** and point it to the **primary node**.

Scale the router service to one replica:

```bash
docker service scale <ROUTER_SERVICE_NAME>=1
```

⏱ Wait about **30–40 seconds** for the router to initialize and register with the cluster.

---

## 🎯 Step 8: Verify Everything

You now have a **fully functional InnoDB Cluster** with:

* One **Primary (Read-Write)** node
* two or more **Secondary (Read-Only)** nodes
* A **Router** managing connections automatically

Check again with:

```js
cluster.status()
```

---

## 💡 Pro Tips

* Use **persistent volumes** for all MySQL instances.
* Regularly run `cluster.status()` to monitor node health.
* Use `dba.rebootClusterFromCompleteOutage()` if the cluster goes down unexpectedly.
* Automate router scaling via Docker events for resilience.

---

## 🧾 Example Stack Deployment Command

```bash
docker stack deploy -c stack.yml innodb-cluster
```

To remove everything cleanly:

```bash
docker stack rm innodb-cluster
```
