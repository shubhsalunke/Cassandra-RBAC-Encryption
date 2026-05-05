# Task 9: Apache Cassandra Security (RBAC + Encryption)

## Overview

This project demonstrates how to implement **security in Apache Cassandra** using:

* Authentication (Password-based)
* RBAC (Role-Based Access Control)
* Client-to-Node SSL Encryption
* Docker Deployment

---

## Server Details

```

Public IP: Your Ip
OS: Ubuntu (Azure VM)

````

---

## Tech Stack

* Apache Cassandra 4.1
* Docker & Docker Compose
* OpenJDK 17
* OpenSSL
* cqlsh

---

## Setup (Start to End)

### 1. Install Dependencies

```bash
sudo apt update -y
sudo apt install -y docker.io docker-compose-v2 openjdk-17-jdk openssl
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker azureuser
newgrp docker
````

---

### 2. Clone Repository

```bash
git clone https://github.com/your-username/cassandra-security-rbac-encryption-demo
cd cassandra-security-rbac-encryption-demo
mkdir certs
```

---

### 3. Generate SSL Keystore

```bash
keytool -genkeypair \
  -alias cassandra \
  -keyalg RSA \
  -keysize 2048 \
  -validity 365 \
  -keystore certs/cassandra.keystore \
  -storepass cassandra123 \
  -keypass cassandra123 \
  -dname "CN=40.75.89.78, OU=DevOps, O=Demo, L=Pune, ST=MH, C=IN"
```

---

### 4. Start Cassandra

```bash
docker compose up -d --force-recreate
sleep 40
```

---

## Database Setup

### Login

```bash
docker exec -it task9_cassandra_security cqlsh -u cassandra -p cassandra
```

---

### Create Keyspace & Table

```sql
CREATE KEYSPACE task9_demo
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE task9_demo;

CREATE TABLE employees (
  id int PRIMARY KEY,
  name text,
  department text,
  salary int
);

INSERT INTO employees VALUES (1, 'Shubham', 'DevOps', 50000);
```

---

## RBAC Implementation

### Create Roles

```sql
CREATE ROLE readonly_user WITH PASSWORD='Read@123' AND LOGIN=true;
CREATE ROLE writer_user WITH PASSWORD='Write@123' AND LOGIN=true;
```

---

### Grant Permissions

```sql
GRANT SELECT ON KEYSPACE task9_demo TO readonly_user;

GRANT SELECT ON KEYSPACE task9_demo TO writer_user;
GRANT MODIFY ON KEYSPACE task9_demo TO writer_user;
```

---

## Testing

### Readonly User

```bash
docker exec -it task9_cassandra_security cqlsh -u readonly_user -p Read@123
```

```sql
SELECT * FROM employees;   -- ✅ Works

INSERT INTO employees VALUES (2, 'Fail', 'IT', 40000);
-- ❌ Fails (No MODIFY permission)
```

---

### 🔹 Writer User

```bash
docker exec -it task9_cassandra_security cqlsh -u writer_user -p Write@123
```

```sql
INSERT INTO employees VALUES (3, 'Writer Test', 'Backend', 70000);
-- ✅ Works
```

---

## SSL Encryption

### Export Certificate

```bash
keytool -exportcert \
  -alias cassandra \
  -keystore certs/cassandra.keystore \
  -storepass cassandra123 \
  -rfc \
  -file certs/cassandra.crt
```

---

### Copy to Container

```bash
docker cp certs/cassandra.crt task9_cassandra_security:/certs/cassandra.crt
```

---

### Test Secure Connection

```bash
docker exec -it \
  -e SSL_CERTFILE=/certs/cassandra.crt \
  task9_cassandra_security \
  cqlsh --ssl -u cassandra -p cassandra
```

---

## Final Result

| Feature              | Status |
| -------------------- | ------ |
| Authentication       | ✅      |
| RBAC                 | ✅      |
| Readonly Restriction | ✅      |
| Writer Access        | ✅      |
| SSL Encryption       | ✅      |
| Docker Setup         | ✅      |

---

## Conclusion

Successfully implemented **Cassandra Security with RBAC and SSL Encryption** using Docker.
