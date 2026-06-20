# Apache Cassandra Security (RBAC + Encryption)

## Overview

This project demonstrates Apache Cassandra security using:

- Authentication
- RBAC (Role-Based Access Control)
- Client-to-Node SSL Encryption
- Docker Deployment

## Server Details

```text
Public IP: Your IP
OS: Ubuntu Azure VM
````

## Setup

### 1. Install Dependencies

```bash
sudo apt update -y
sudo apt install -y docker.io docker-compose-v2 openjdk-17-jdk openssl git
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker azureuser
newgrp docker
```

### 2. Clone Repository

```bash
git clone https://github.com/<your-username>/06-Cassandra-RBAC-Encryption.git
cd 06-Cassandra-RBAC-Encryption
```

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

### 4. Start Cassandra

```bash
docker compose up -d --force-recreate
sleep 40
```

### 5. Export SSL Certificate

```bash
keytool -exportcert \
  -alias cassandra \
  -keystore certs/cassandra.keystore \
  -storepass cassandra123 \
  -rfc \
  -file certs/cassandra.crt
```

```bash
docker cp certs/cassandra.crt task9_cassandra_security:/certs/cassandra.crt
```

## Database Setup

### Login With SSL

```bash
docker exec -it \
  -e SSL_CERTFILE=/certs/cassandra.crt \
  task9_cassandra_security \
  cqlsh --ssl -u cassandra -p cassandra
```

### Create Keyspace and Table

```sql
CREATE KEYSPACE IF NOT EXISTS task9_demo
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE task9_demo;

CREATE TABLE IF NOT EXISTS employees (
  id int PRIMARY KEY,
  name text,
  department text,
  salary int
);

INSERT INTO employees (id, name, department, salary)
VALUES (1, 'Shubh', 'DevOps', 50000);
```

## RBAC Implementation

```sql
CREATE ROLE IF NOT EXISTS readonly_user WITH PASSWORD='Read@123' AND LOGIN=true;
CREATE ROLE IF NOT EXISTS writer_user WITH PASSWORD='Write@123' AND LOGIN=true;

GRANT SELECT ON KEYSPACE task9_demo TO readonly_user;

GRANT SELECT ON KEYSPACE task9_demo TO writer_user;
GRANT MODIFY ON KEYSPACE task9_demo TO writer_user;

Exit;
```

## Testing

### Readonly User

```bash
docker exec -it \
  -e SSL_CERTFILE=/certs/cassandra.crt \
  task9_cassandra_security \
  cqlsh --ssl -u readonly_user -p 'Read@123'
```

```sql
USE task9_demo;

SELECT * FROM employees;

INSERT INTO employees (id, name, department, salary)
VALUES (2, 'Fail', 'IT', 40000);

Exit;
```

Expected: insert fails because readonly user has no MODIFY permission.

### Writer User

```bash
docker exec -it \
  -e SSL_CERTFILE=/certs/cassandra.crt \
  task9_cassandra_security \
  cqlsh --ssl -u writer_user -p 'Write@123'
```

```sql
USE task9_demo;

INSERT INTO employees (id, name, department, salary)
VALUES (3, 'Writer Test', 'Backend', 70000);

SELECT * FROM employees;

Exit; 
```

Expected: insert works.

## Final Result

| Feature              | Status |
| -------------------- | ------ |
| Authentication       | ✅      |
| RBAC                 | ✅      |
| Readonly Restriction | ✅      |
| Writer Access        | ✅      |
| SSL Encryption       | ✅      |
| Docker Setup         | ✅      |

## Conclusion

Successfully implemented Cassandra Security with RBAC and SSL Encryption using Docker.
