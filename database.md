# ✅ Microservices & Database
- In **microservices architecture**, each service should have its own **independent database**.
- For local development:
  ```bash
  docker run -p 3306:3306 --name bank-account-db -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=bank-account-db -d mysql
  ```
- Use [SQLECTRON](https://sqlectron.github.io/) with `localhost:3306` to connect.
- Assign **unique ports per service** locally.

---

# 📦 AWS RDS (Relational Database Service)

## 🔍 Overview
- **Fully managed** relational database service (continuous backup & restore, monitoring).
- Supports **MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora**.
- No **SSH access** to DB instances.
- Billing based on:
    - Storage capacity used
    - I/O requests
    - Backup storage
    - Data transfer
- Instance types: **On-Demand** (pay-as-you-go)
- Part of **VPC** for security and network configuration

---

## ⚙️ Key Features

### 📊 Read Replicas
- Improves **read performance**.
- **Async replication**, eventually consistent.
- Up to **15 replicas** across:
    - Same AZ
    - Cross-AZ
    - Cross-region
- Read replica can be **promoted** to master.
- To leverage read replicas, `applicaiton` must be updated
- Read client in a different region can use that region's read replica
- Use case: **analytics, reporting** without impacting production DB.
- Replication is **one-way**, used for **read scalability**.

![aws-rds-read-replica.png](media/aws-rds-read-replica.png)
- Complete Multi-AZ Setup

![VPC-AWS-RDS.png](media/VPC-RDS-setup.png)

**Network Cost and Read Replicas**
- Usually traffic between AZs incur additional costs
- However, for RDS read replicas within the same region, you don't pay that fee.
- i.e `Master (M)` in `us-east-1a` replicates the data to `read replica (R)` in `us-east-1b` without cross AZ costs.
- Cross-region will incur costs (replication fee)

### 🛡️ Multi-AZ (High Availability & DR)
- **Synchronous replication** for HA.
- Automatic **failover** in case of:
    - AZ failure
    - Network issues
    - Storage or DB instance issues
- **No performance boost**, purely for **availability**.
- No manual intervention in app (automatic)
  - AWS RDS simply flips the CNAME for the DB to point to the standby, which in turm promoted to become primary
- Note: A read replica can be setup as multi-AZ for DR
- Going from Single-AZ to Multi-AZ
  - zero downtime, just modify
  - an standby will be brought up
    - a snapshot is taken
    - a new DB is restored from snapshot in the new AZ
    - synchronization is established between the two DBs

![multi-az-dr-availability.png](media/multi-az-dr-availability.png)

- Data sync between master and standby each 5 minutes
- Failover in 1 to 2 minutes
- Upgrade: first on standby, reroute traffic to standby and so on
- Backups can be taken from standby to avoid impacts

---

### 🔐 Security & Connectivity
- Encrypted **at-rest and in-transit (TLS)** using **KMS**.
- Should be deployed in **private subnet** (not public internet).
- Security via:
    - Security groups (stateful firewall)
    - Network ACL (optional)
    - IAM for authentication
    - **Secrets Manager** for secure password rotation

---

### 🔁 Backups & Snapshots
- **Automated backups (snapshots)**:
    - Daily full backup during backup window
    - Transaction logs every 5 mins
      - so you can restore to any point in time (till 5 minutes ago)
    - Retention: 1–30 days
- **Manual Snapshots**:
    - manually triggered by the user
    - Long-term storage
    - Cheaper than running instance
    - Can be copied/shared across regions/accounts
- To **encrypt unencrypted DB**:
    - Take snapshot → copy with encryption → restore

---

### 🧰 RDS Resource Creation (Steps)
1. Create VPC
2. Create Subnet Group using VPC and Subnets
3. Create DB:
    - Choose engine, storage (e.g. gp3, autoscaling), instance type
    - Set up security group
4. Configure connectivity (EC2 connection, VPC options)

---

### 🔄 RDS Proxy
- Manages DB connections:
    - Reduces load, minimizes timeouts
    - Pools connections
- Fully managed, **multi-AZ**, **autoscaling**
- **Reduces failover time by up to 66%**
- Works with:
    - RDS (MySQL, PostgreSQL, etc.)
    - Aurora (MySQL & PostgreSQL)
- Integrates with IAM & Secrets Manager

![aws-rds-proxy.png](media/aws-rds-proxy.png)

---

### 🧪 RDS vs RDS Custom
| Feature          | RDS                         | RDS Custom                         |
|------------------|------------------------------|-------------------------------------|
| OS-level access | ❌ No                         | ✅ Yes (SSH, root access)           |
| Use Cases       | General apps                 | Custom configurations, legacy apps |
| Supported Engines | All major engines incl. Aurora | Only Oracle & SQL Server           |

---
### RDS Demos
- Configuration

![rds-postgres-config.gif](media/rds-postgres-config.gif)

- Connection to the DB

![rds-postgres-connection.gif](media/rds-postgres-connection.gif)

- Additional features

![rds-additional-features.gif](media/rds-additional-features.gif)

# 🚀 Amazon Aurora

## 🔍 Overview
- **Proprietary, cloud-native** DB engine by AWS.
- **Compatible with MySQL & PostgreSQL**
- Offers **up to 5x performance** over MySQL and **3x over PostgreSQL**
- Comes with both **Provisioned** and **Serverless** options.

---

## ⚙️ Core Features

### 💡 Performance & Scalability
- **Auto-scaling storage**: 10GB–128TB
- **Sub-10s autoscaling** of compute (Serverless v2)
- Supports **up to 15 read replicas**
- Separate **cluster endpoint** for reads and writes
- **Aurora I/O-Optimized** and **Aurora Standard** storage types:
    - I/O-Optimized: better for intensive workloads
    - Standard: cost-effective

![aurora-auto-scaling-endpoints.png](media/aurora-auto-scaling-endpoints.png)
---

### 🔁 High Availability & Fault Tolerance
- Data replicated **6 times across 3 AZs**
- Only **4/6 copies** needed for writes, 3/6 for reads
- **Self-healing** and **peer-to-peer** replication
- Failover:
    - Replica priorities determine promotion
    - Failover time <30 seconds
- Write-only **one master instance**, all others are **read-only**

![aurora-availability-scaling.png](media/aurora-availability-scaling.png)


![aurora-db-w-r-endpoint.png](media/aurora-db-w-r-endpoint.png)

---

### 🔄 Aurora Cloning
- **Fast, cost-effective alternative** to snapshot/restore
- Uses **copy-on-write** model
- Ideal for **staging environments**, testing

---

### 🌎 Aurora Global Database
- **Global, cross-region** read replication
- Up to **5 secondary regions**
- Less than **1-second replication lag**
- Region failover <1 minute
- Up to **16 replicas** per region

![aurora-global.png](media/aurora-global.png)
---

### ☁️ Aurora Serverless
- **Automatically scales** based on usage
- Ideal for:
    - Infrequent workloads
    - Spiky traffic
- **Pay-per-second**
- Available for both **MySQL & PostgreSQL**
- **Serverless clusters** can have:
    - Writer provisioned, readers serverless (or vice versa)

![aurora-serverless.png](media/aurora-serverless.png)
---

### 🔐 Aurora Security
- **KMS encryption**, at-rest and in-transit
- Integrated with **IAM**, **Secrets Manager**
- Only **VPC-accessible**, not internet-exposed

---

### 🔧 Aurora Advanced
- **Custom Endpoints**: define endpoints for specific replicas

![aurora-custom-endpoints.png](media/aurora-custom-endpoints.png)

- **ML Integration**:
    - Use SageMaker, Comprehend
    - Real-time predictions in DB (fraud detection, recommendations)

![aurora-ml-integration.png](media/aurora-ml-integration.png)
---

### 📋 Aurora Backups
- **Automated Backups**:
    - 1 to 35 days (mandatory)
- **Manual Snapshots**
- **Point-in-time restore** support

---

## 🛠️ Aurora Resource Creation
1. Choose **Aurora engine (MySQL/Postgres)**
2. Select:
    - Writer/Reader endpoints
    - Instance classes (Graviton, Intel, etc.)
    - Storage type (Standard / I/O-Optimized)
    - Enable Serverless / Global DB
3. Configure:
    - Failover priority
    - VPC + security groups
    - IAM roles and secrets
4. Connect via EC2 using private endpoint

![aurora-db.png](media/aurora-db.png)

---

### 📈 Aurora vs RDS Comparison (Quick Glance)

| Feature                  | RDS                            | Aurora                                |
|--------------------------|---------------------------------|----------------------------------------|
| Engine Support           | MySQL, Postgres, Oracle, etc.  | Aurora MySQL & Aurora Postgres         |
| Performance              | Standard                       | 3x Postgres, 5x MySQL                  |
| Replicas                 | Up to 5                        | Up to 15                               |
| Failover Time            | 1–2 mins (Multi-AZ)            | <30 seconds                            |
| Storage Scaling          | Manual or autoscale            | Automatic 10GB–128TB                   |
| Global DB                | Cross-region replica           | Native global support (Aurora Global)  |
| Serverless Option        | ❌                             | ✅ Aurora Serverless                   |
| Pricing                  | Lower                          | ~20% higher but better efficiency      |
| Backup Retention         | 1–30 days                      | 1–35 days                              |

---

## ⚙️ What Are RDS Parameter Groups?

An **RDS Parameter Group** is like a **configuration profile** for your RDS or Aurora DB instance or cluster.

It contains settings that define the behavior of the database engine — kind of like editing your `my.cnf` or `postgresql.conf` file in a traditional setup.

---

### 🧩 Types of Parameter Groups

There are **two main types**, depending on the engine:

| Type                    | Used For                              |
|-------------------------|----------------------------------------|
| **DB Parameter Group**  | RDS instance-level parameters (MySQL, PostgreSQL, etc.) |
| **DB Cluster Parameter Group** | Aurora cluster-level settings (Aurora MySQL, Aurora PostgreSQL) |

> For **Aurora**, some settings go in the **cluster parameter group**, while others (like memory or log settings) go in the **instance-level parameter group**.

---

### 🔧 Examples of Things You Can Configure

- `max_connections`
- `log_statement`
- `innodb_buffer_pool_size`
- `autovacuum`
- `slow_query_log`
- `timezone`
- `log_min_duration_statement`

---

### 🛠️ How to Use a Parameter Group

1. **Create a custom parameter group** from the RDS Console or CLI.
2. **Modify the parameters** you need.
3. **Assign it to your DB instance or cluster.**
4. **Reboot** the DB instance if the parameters require it.

> Some changes are **dynamic** (apply immediately), others are **static** (require reboot).

---

### ⚠️ Important Notes

- If you don’t explicitly assign one, your DB uses the **default parameter group**, which is **read-only**.
- Always create and use a **custom parameter group** when you need to tweak engine behavior.

---

### 🧠 Tip for Aurora Users:
- **Aurora MySQL / PostgreSQL** has both **DB cluster parameter group** and **DB parameter group** — you'll usually need to modify both.

---

## ✅ TL;DR on Parameter Groups

| Feature              | RDS                                  | Aurora                                 |
|----------------------|---------------------------------------|-----------------------------------------|
| Config Scope         | Per instance                         | Per cluster + per instance              |
| Default Group        | Read-only                            | Read-only                               |
| Custom Group         | Editable, required for tuning        | Editable, required for tuning           |
| Apply Type           | Dynamic or Static (reboot needed)    | Same                                    |

---

## Aurora DB Demo
- Aurora Setup Demo

![aurora-db-setup.gif](media/aurora-db-setup.gif)

- Aurora DB Interaction Demo

![aurora-db-interaction.gif](media/aurora-db-interaction.gif)