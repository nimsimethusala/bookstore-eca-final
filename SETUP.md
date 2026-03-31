# 🛠️ Bookstore ECA – Full Setup Guide

## Prerequisites

Before starting, ensure you have installed:

| Tool | Version | Download |
|---|---|---|
| Java JDK | 21 | https://adoptium.net |
| Apache Maven | 3.9+ | https://maven.apache.org |
| MySQL Server | 8.0+ | https://dev.mysql.com/downloads |
| MongoDB | 7.0+ | https://www.mongodb.com/try/download |
| Node.js + PM2 | 20+ | https://nodejs.org |
| Git | Latest | https://git-scm.com |

---

## Part 1 — Local Development Setup

### Step 1: Clone the Project

```bash
git clone https://github.com/<your-username>/bookstore-main.git
cd bookstore-main
git submodule update --init --recursive
```

### Step 2: Set Up Databases

**MySQL — create databases:**
```sql
CREATE DATABASE bookstore_books;
CREATE DATABASE bookstore_files;
CREATE USER 'bookstore'@'localhost' IDENTIFIED BY 'Root@1234';
GRANT ALL PRIVILEGES ON bookstore_books.* TO 'bookstore'@'localhost';
GRANT ALL PRIVILEGES ON bookstore_files.* TO 'bookstore'@'localhost';
FLUSH PRIVILEGES;
```

**MongoDB — no setup needed**, Spring Boot auto-creates the `bookstore_orders` database.

### Step 3: Configure Database Credentials

Edit `config-server/src/main/resources/configurations/book-service.yml`:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/bookstore_books?createDatabaseIfNotExist=true&useSSL=false&allowPublicKeyRetrieval=true
    username: root          # ← change to your MySQL username
    password: Root@1234     # ← change to your MySQL password
```

Edit `config-server/src/main/resources/configurations/file-service.yml`:
```yaml
spring:
  datasource:
    username: root          # ← change to your MySQL username
    password: Root@1234     # ← change to your MySQL password
gcp:
  storage:
    bucket-name: bookstore-files-bucket   # ← your GCS bucket name
    project-id: your-gcp-project-id       # ← your GCP project ID
    credentials-path: /etc/gcp/service-account.json   # ← path to your JSON key
```

### Step 4: Build All Services

Run the following from the project root. Build in this exact order:

```bash
# 1. Config Server
cd config-server
mvn clean package -DskipTests
cd ..

# 2. Eureka Server
cd eureka-server
mvn clean package -DskipTests
cd ..

# 3. API Gateway
cd api-gateway
mvn clean package -DskipTests
cd ..

# 4. Book Service
cd book-service
mvn clean package -DskipTests
cd ..

# 5. Order Service
cd order-service
mvn clean package -DskipTests
cd ..

# 6. File Service
cd file-service
mvn clean package -DskipTests
cd ..
```

### Step 5: Start Services — Correct Startup Order

**⚠️ IMPORTANT:** Always start in this order. Wait 15–20 seconds between each step.

```bash
# Terminal 1 — Config Server (MUST start first)
cd config-server
java -jar target/config-server-1.0.0.jar

# Terminal 2 — Eureka Server (wait 15s after config server)
cd eureka-server
java -jar target/eureka-server-1.0.0.jar

# Terminal 3 — API Gateway (wait for Eureka to be UP)
cd api-gateway
java -jar target/api-gateway-1.0.0.jar

# Terminal 4 — Book Service
cd book-service
java -jar target/book-service-1.0.0.jar

# Terminal 5 — Order Service
cd order-service
java -jar target/order-service-1.0.0.jar

# Terminal 6 — File Service
cd file-service
java -jar target/file-service-1.0.0.jar
```

### Step 6: Verify Everything is Running

- **Eureka Dashboard:** http://localhost:8761
- **Config Server:** http://localhost:8888/book-service/default
- **API Gateway:** http://localhost:8080/api/v1/books/health

Check that all 3 microservices appear registered in the Eureka Dashboard.

### Step 7: Open the Frontend

Open `frontend/index.html` in your browser.

Change the `API_BASE_URL` at the top of the file if needed:
```javascript
const API_BASE_URL = 'http://localhost:8080';  // local
// const API_BASE_URL = 'http://<GCP-GATEWAY-IP>:8080';  // GCP
```

---

## Part 2 — GCP Deployment Setup

### Step 1: Create GCP Resources

Log in to your GCP Console and create:

1. **VPC Network** — `bookstore-vpc` with custom subnet
2. **Cloud SQL (MySQL)** — `bookstore-mysql` instance (MySQL 8.0)
3. **MongoDB** — Deploy on a VM (or use Atlas)
4. **Cloud Storage Bucket** — `bookstore-files-bucket` (set to public read)
5. **Cloud Router + NAT** — for outbound internet from private VMs
6. **Firewall Rules** — allow TCP 8080, 8081, 8082, 8083, 8761, 8888

### Step 2: Create a VM for Platform Services (Config + Eureka)

```bash
# Create platform VM
gcloud compute instances create bookstore-platform \
  --machine-type=e2-medium \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --tags=bookstore-platform \
  --zone=asia-south1-a
```

### Step 3: Install Java and PM2 on VMs

SSH into each VM and run:

```bash
# Install Java 21
sudo apt update
sudo apt install -y openjdk-21-jdk

# Install Node.js and PM2
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2

# Create log directory
sudo mkdir -p /var/log/bookstore
sudo chmod 777 /var/log/bookstore

# Create app directory
sudo mkdir -p /opt/bookstore
```

### Step 4: Upload JAR Files to VMs

```bash
# From your local machine, SCP each JAR
gcloud compute scp config-server/target/config-server-1.0.0.jar bookstore-platform:/opt/bookstore/config-server/ --zone=asia-south1-a
gcloud compute scp eureka-server/target/eureka-server-1.0.0.jar bookstore-platform:/opt/bookstore/eureka-server/ --zone=asia-south1-a
gcloud compute scp api-gateway/target/api-gateway-1.0.0.jar bookstore-platform:/opt/bookstore/api-gateway/ --zone=asia-south1-a
```

### Step 5: Update Config for GCP

In `config-server/src/main/resources/configurations/`, update the database URLs to use Cloud SQL private IP and your MongoDB VM IP.

Rebuild and re-upload config-server JAR after changes.

### Step 6: Set Up GCP Service Account for Cloud Storage

```bash
# On the file-service VM
sudo mkdir -p /etc/gcp
# Upload your service account JSON key
gcloud compute scp service-account.json file-service-vm:/etc/gcp/service-account.json --zone=asia-south1-a
```

### Step 7: Start All Services with PM2

Upload `ecosystem.config.js` to each VM, then:

```bash
# On the platform VM (config + eureka + gateway)
pm2 start ecosystem.config.js --only config-server
sleep 20
pm2 start ecosystem.config.js --only eureka-server
sleep 20
pm2 start ecosystem.config.js --only api-gateway

# Save PM2 state so it survives VM restart
pm2 save
pm2 startup   # follow the printed command to enable auto-start

# On microservice VMs
pm2 start ecosystem.config.js --only book-service
pm2 start ecosystem.config.js --only order-service
pm2 start ecosystem.config.js --only file-service
pm2 save
pm2 startup
```

### Step 8: Verify PM2 Processes

```bash
pm2 list
pm2 monit
```

All services should show status `online`.

### Step 9: Create Instance Templates and Instance Groups

For each microservice, create an Instance Template based on the configured disk image, then create a Managed Instance Group with:
- **Min instances:** 2
- **Max instances:** 5
- **Autoscaling policy:** CPU utilization > 60%

### Step 10: Set Up Load Balancer

Create an HTTP(S) Load Balancer pointing to the API Gateway instance group's named port `8080`.

Create Health Check targeting `/api/v1/books/health`.

### Step 11: Configure Cloud DNS

Map your domain or a static IP to the Load Balancer frontend.

---

## Part 3 — Verifying the Deployment

### Check Eureka Dashboard

Open: `http://<EUREKA-VM-EXTERNAL-IP>:8761`

You should see all 3 microservices registered:
- `BOOK-SERVICE`
- `ORDER-SERVICE`
- `FILE-SERVICE`

### Test API via Gateway

```bash
curl http://<GATEWAY-IP>:8080/api/v1/books/health
curl http://<GATEWAY-IP>:8080/api/v1/orders/health
curl http://<GATEWAY-IP>:8080/api/v1/files/health
```

Each should return `{"status":200,"message":"... Service is running","data":"UP"}`

### Import Postman Collection

1. Open Postman
2. Click **Import** → select `Bookstore-ECA.postman_collection.json`
3. Set the collection variable `base_url` to `http://<GATEWAY-IP>:8080`
4. Run requests in order

---

## Troubleshooting

| Problem | Solution |
|---|---|
| Service not registering with Eureka | Ensure Config Server is up first; check `bootstrap.yml` config URI |
| MySQL connection refused | Check firewall rules allow port 3306; verify credentials in config |
| MongoDB connection refused | Ensure MongoDB is running: `sudo systemctl status mongod` |
| File upload fails | Verify GCP service account has Storage Admin role; check credentials path |
| PM2 service stops | Run `pm2 logs <service-name>` to see the error |
| Port not accessible | Check GCP Firewall Rules allow the required ports |

---

## Port Reference

| Service | Port |
|---|---|
| Config Server | 8888 |
| Eureka Server | 8761 |
| API Gateway | 8080 |
| Book Service | 8081 |
| Order Service | 8082 |
| File Service | 8083 |
| MySQL | 3306 |
| MongoDB | 27017 |
