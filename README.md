# 📚 Bookstore Cloud – ITS 2130 Enterprise Cloud Architecture

> **Student Project | Higher Diploma in Software Engineering | IJSE**

---

## 🔗 Live URLs (update after GCP deployment)

| Service | URL |
|---|---|
| **Eureka Dashboard** | `http://<YOUR-EUREKA-VM-EXTERNAL-IP>:8761` |
| **API Gateway** | `http://<YOUR-GATEWAY-EXTERNAL-IP>:8080` |
| **Frontend** | `http://<YOUR-FRONTEND-IP>` |

---

## 🏗️ Architecture Overview

This is a **cloud-native microservice-based Bookstore application** deployed on **Google Cloud Platform (GCP)**.

```
Frontend (HTML/JS)
       │
       ▼
  API Gateway  (:8080)          ← Spring Cloud Gateway + Eureka Client
       │
  ┌────┴────────────────┐
  ▼        ▼            ▼
Book      Order        File
Service   Service      Service
(:8081)   (:8082)     (:8083)
MySQL     MongoDB     MySQL + GCS
```

**Platform Services:**
- **Config Server** (:8888) — Centralized configuration for all microservices
- **Eureka Server** (:8761) — Service registry and discovery
- **API Gateway** (:8080) — Single entry point, load-balanced routing

---

## 📦 Repository Structure (Polyrepo + Git Submodules)

```
bookstore-main/                   ← This repository (submission repo)
├── config-server/                ← Git submodule
├── eureka-server/                ← Git submodule
├── api-gateway/                  ← Git submodule
├── book-service/                 ← Git submodule
├── order-service/                ← Git submodule
├── file-service/                 ← Git submodule
├── frontend/                     ← Git submodule
└── .gitmodules
```

---

## 🛠️ Technology Stack

| Component | Technology |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3.4.1 |
| Service Discovery | Spring Cloud Netflix Eureka |
| Config Management | Spring Cloud Config |
| API Gateway | Spring Cloud Gateway |
| Relational DB | MySQL (book-service, file-service) |
| NoSQL DB | MongoDB (order-service) |
| File Storage | Google Cloud Storage |
| Process Manager | PM2 |
| Cloud Platform | Google Cloud Platform (GCP) |

---

## 🌐 GCP Infrastructure Used

- VM Instance Groups (auto-scaling for microservices)
- VM Instance Templates
- Virtual Machines (VMs)
- Disk Images
- Health Checks
- Cloud DNS
- Load Balancing (HTTP/S)
- Cloud NAT Gateway
- Cloud SQL (MySQL)
- Firestore (MongoDB alternative)
- Cloud Storage Buckets
- Cloud Router
- VPC Network + Firewall Rules

---

## 🔌 API Endpoints Summary

All requests go through the **API Gateway** at port `8080`.

### Book Service — `/api/v1/books`
| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/books` | Create a new book |
| GET | `/api/v1/books` | Get all books |
| GET | `/api/v1/books/{id}` | Get book by ID |
| GET | `/api/v1/books/isbn/{isbn}` | Get book by ISBN |
| GET | `/api/v1/books/category/{category}` | Get books by category |
| GET | `/api/v1/books/search?keyword=` | Search books |
| GET | `/api/v1/books/available` | Get in-stock books |
| PUT | `/api/v1/books/{id}` | Update book |
| PATCH | `/api/v1/books/{id}/stock?quantity=` | Update stock |
| DELETE | `/api/v1/books/{id}` | Delete book |

### Order Service — `/api/v1/orders`
| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/orders` | Create a new order |
| GET | `/api/v1/orders` | Get all orders |
| GET | `/api/v1/orders/{id}` | Get order by ID |
| GET | `/api/v1/orders/customer?email=` | Get orders by customer |
| GET | `/api/v1/orders/status/{status}` | Get orders by status |
| PATCH | `/api/v1/orders/{id}/status?status=` | Update order status |
| PATCH | `/api/v1/orders/{id}/cancel` | Cancel an order |
| DELETE | `/api/v1/orders/{id}` | Delete order |

### File Service — `/api/v1/files`
| Method | Path | Description |
|---|---|---|
| POST | `/api/v1/files/upload` | Upload file to GCS |
| GET | `/api/v1/files` | Get all file records |
| GET | `/api/v1/files/{id}` | Get file record by ID |
| GET | `/api/v1/files/category/{category}` | Get files by category |
| GET | `/api/v1/files/reference?refId=&refType=` | Get files by reference |
| DELETE | `/api/v1/files/{id}` | Delete file |

---

## 🚀 Quick Setup (Local Development)

See **[SETUP.md](./SETUP.md)** for full step-by-step local and GCP deployment instructions.

---

## 📮 Postman Collection

Import `Bookstore-ECA.postman_collection.json` into Postman.

Set the `base_url` variable to:
- **Local:** `http://localhost:8080`
- **GCP:** `http://<GATEWAY-IP>:8080`

---

## 👤 Author

**[Your Name]**  
HDSE Batch [Your Batch]  
IJSE – Institute of Java and Software Engineering
