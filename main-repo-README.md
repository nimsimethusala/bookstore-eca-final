# 📚 Bookstore Cloud – Main Repository

> **ITS 2130 Enterprise Cloud Architecture | Final Project | HDSE @ IJSE**

This is the **main submission repository** that links all individual microservice repositories via Git submodules.

---

## 🔗 Live Eureka Dashboard

> **http://\<YOUR-EUREKA-VM-EXTERNAL-IP\>:8761**
>
> *(Replace with your actual GCP VM external IP after deployment)*

---

## 📦 Submodule Repositories

| Repository | Description |
|---|---|
| [config-server](./config-server) | Spring Cloud Config Server – centralized config |
| [eureka-server](./eureka-server) | Netflix Eureka – service registry & discovery |
| [api-gateway](./api-gateway) | Spring Cloud Gateway – single entry point |
| [book-service](./book-service) | Book management microservice (MySQL) |
| [order-service](./order-service) | Order management microservice (MongoDB) |
| [file-service](./file-service) | File upload microservice (MySQL + GCS) |
| [frontend](./frontend) | HTML/JS frontend application |

---

## 🚀 Clone with Submodules

```bash
git clone --recurse-submodules https://github.com/<your-username>/bookstore-main.git
```

Or if already cloned:
```bash
git submodule update --init --recursive
```

---

## 📖 Documentation

- **[README.md](./README.md)** – Full project overview, architecture, API docs
- **[SETUP.md](./SETUP.md)** – Step-by-step local & GCP deployment guide
- **[Bookstore-ECA.postman_collection.json](./Bookstore-ECA.postman_collection.json)** – Postman API collection
- **[ecosystem.config.js](./ecosystem.config.js)** – PM2 process management config

---

## 🏗️ Architecture

```
                   ┌─────────────────────┐
                   │   Config Server      │  :8888
                   │  (Spring Cloud)      │
                   └─────────┬───────────┘
                             │ provides config
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌──────────────┐ ┌───────────┐ ┌──────────────┐
     │ Eureka Server│ │API Gateway│ │ Microservices│
     │   :8761      │ │  :8080    │ │ :8081/:8082  │
     └──────────────┘ └─────┬─────┘ │ /:8083       │
                             │       └──────────────┘
                      routes to:
                   ┌──────┬──────┬──────┐
                   ▼      ▼      ▼
              Book Svc  Order  File Svc
              (MySQL)  (MongoDB) (MySQL+GCS)
```

---

## ☁️ GCP Infrastructure

- VM Instance Groups (auto-scaling, multi-zone)
- VM Instance Templates + Disk Images
- Health Checks + HTTP Load Balancer
- Cloud SQL (MySQL 8.0)
- Cloud Firestore / MongoDB VM
- Cloud Storage Bucket (`bookstore-files-bucket`)
- Cloud NAT Gateway + Cloud Router
- VPC Network + Firewall Rules
- Cloud DNS
