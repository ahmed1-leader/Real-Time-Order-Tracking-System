# PRD: Real-Time Order Tracking System

| Field | Details |
|---|---|
| **Document Version** | 1.0 |
| **Last Updated** | April 2026 |
| **Product Type** | Web Application (Backend + WebSocket) |
| **Target Audience** | For Me As Java Backend Developer portfolio project |
| **Tech Stack** | Java 21, Spring Boot 3.x, Spring Security + JWT, PostgreSQL + PostGIS, WebSocket (STOMP), Docker, JUnit + Mockito, Swagger/OpenAPI |

---

## 1. Executive Summary

A backend system that enables real-time order tracking for a delivery platform. Three user types interact:

- **Customer** – places orders, tracks driver live on a map
- **Driver** – accepts/rejects orders, updates location in real time
- **Admin** – manages users, views order history, resolves disputes

The system demonstrates expertise in: REST APIs, WebSocket (STOMP), JWT + RBAC, PostGIS geospatial queries, Docker, and testing.

---

## 2. User Stories

### Customer
- As a customer, I want to register/login using JWT.
- As a customer, I want to place an order with pickup/delivery addresses.
- As a customer, I want to see nearby available drivers (PostGIS).
- As a customer, I want to track my driver's live location on a mock map.
- As a customer, I want to receive push-style notifications (via WebSocket) when my order status changes.

### Driver
- As a driver, I want to toggle online/offline status.
- As a driver, I want to receive real-time order requests in my area.
- As a driver, I want to accept/reject an order.
- As a driver, I want to update my GPS location every few seconds (WebSocket).
- As a driver, I want to see my completed orders & earnings.

### Admin
- As an admin, I want to view all users, orders, and drivers.
- As an admin, I want to ban/unban a user or driver.
- As an admin, I want to see order history with location trails (PostGIS LINESTRING).

---

## 3. Functional Requirements

### 3.1 Authentication & Authorization (JWT + RBAC)

| Endpoint | Role |
|---|---|
| `POST /api/auth/register` | Public |
| `POST /api/auth/login` | Public |
| `GET /api/users/me` | Customer, Driver, Admin |
| `PUT /api/admin/users/{id}/ban` | Admin only |
| `PUT /api/drivers/online` | Driver only |

### 3.2 Order Management (REST)

| Endpoint | Description |
|---|---|
| `POST /api/orders` | Customer creates order (pickup + delivery lat/lng) |
| `GET /api/orders/{id}` | Get order details + current status |
| `GET /api/orders/my` | List customer's orders |
| `PUT /api/drivers/orders/{id}/accept` | Driver accepts order |
| `PUT /api/drivers/orders/{id}/reject` | Driver rejects order |
| `PUT /api/orders/{id}/status` | Admin updates status (e.g., `DELIVERED`) |

**Order status flow:**

```
PENDING → ACCEPTED → PICKED_UP → IN_TRANSIT → DELIVERED
(Also: REJECTED, CANCELLED)
```

### 3.3 Real-Time Driver Location (WebSocket STOMP)

**Driver publishes location:**

```
Destination: /app/drivers/location
Payload:
{
  "driverId": 123,
  "orderId": 456,
  "lat": 30.0444,
  "lng": 31.2357
}
```

**Customer subscribes to:**

```
/topic/orders/{orderId}/location
```

The server broadcasts location to that order's customer only.

### 3.4 Real-Time Order Notifications (WebSocket)

| Event | Subscribed User |
|---|---|
| New order available | Drivers within 3km (PostGIS) |
| Driver accepted | Customer |
| Driver arrived at pickup | Customer |
| Order status changed | Customer + Admin |
| Driver location update | Customer (via separate WebSocket topic) |

### 3.5 Geospatial (PostGIS)

- Store pickup/delivery locations as `POINT(lng lat)`
- Find nearby drivers: `ST_DWithin(location, driver_location, radius_in_meters)`
- Store full driver route as `LINESTRING` (optional, for history)

### 3.6 Testing Requirements

| Layer | Tools | Target |
|---|---|---|
| Unit tests | JUnit 5 + Mockito | ≥ 70% coverage |
| Repository tests | `@DataJpaTest` + TestContainers | CRUD + geospatial queries |
| Controller tests | MockMvc | All REST endpoints |
| WebSocket tests | `@SpringBootTest` + StompSession | Location broadcast |

### 3.7 API Documentation

- OpenAPI 3.0 / Swagger UI available at `/swagger-ui.html`
- Include example request/response for all endpoints

---

## 4. Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Performance** | REST API response < 200ms (p95); WebSocket broadcast < 50ms |
| **Scalability** | Can handle 100 concurrent WebSocket connections locally |
| **Security** | Passwords hashed with BCrypt; JWT expiry = 24h; HTTPS (optional for portfolio) |
| **Portability** | Docker multi-stage build + docker-compose for Spring Boot + PostGIS |
| **Code Quality** | Follows SOLID + package-by-feature structure |

---

## 5. Database Schema

### `users`

| Column | Type |
|---|---|
| `id` | `BIGINT` PK |
| `email` | `VARCHAR` UNIQUE |
| `password_hash` | `VARCHAR` |
| `role` | `ENUM (CUSTOMER, DRIVER, ADMIN)` |
| `is_banned` | `BOOLEAN` |

### `drivers` (extends users)

| Column | Type |
|---|---|
| `user_id` | `BIGINT` PK, FK → `users(id)` |
| `is_online` | `BOOLEAN` |
| `current_lat` | `DOUBLE` |
| `current_lng` | `DOUBLE` |
| `total_earnings` | `DECIMAL` |

### `orders`

| Column | Type |
|---|---|
| `id` | `BIGINT` PK |
| `customer_id` | `BIGINT` FK → `users(id)` |
| `driver_id` | `BIGINT` FK → `drivers(user_id)` |
| `pickup_location` | `GEOGRAPHY(POINT)` – PostGIS |
| `delivery_location` | `GEOGRAPHY(POINT)` – PostGIS |
| `status` | `VARCHAR` |
| `created_at` | `TIMESTAMP` |

### `driver_location_history` _(optional)_

| Column | Type |
|---|---|
| `id` | `BIGINT` PK |
| `driver_id` | `BIGINT` |
| `order_id` | `BIGINT` |
| `location` | `GEOGRAPHY(POINT)` |
| `recorded_at` | `TIMESTAMP` |

---

## 6. API Response Examples

### Place Order Response

```json
{
  "orderId": 101,
  "status": "PENDING",
  "nearbyDriversCount": 3,
  "estimatedWaitSeconds": 120
}
```

### WebSocket Location Broadcast (to customer)

```json
{
  "orderId": 101,
  "driverId": 5,
  "lat": 30.047,
  "lng": 31.233,
  "timestamp": "2026-04-28T10:30:00Z"
}
```

---

## 7. Deliverables for Portfolio

### GitHub Repository
- [ ] Clean, commented code
- [ ] `README.md` with setup instructions
- [ ] `docker-compose.yml`
- [ ] Postman collection (or OpenAPI JSON)
- [ ] Screenshot of Swagger UI

### Video Demo (2–3 min) showing:
- [ ] Customer places order
- [ ] Driver receives request (via WebSocket)
- [ ] Driver accepts and updates location
- [ ] Customer sees live location

### Test Report
- [ ] Generated from `mvn test`

---

## 8. Skill Mapping

| Your Skill | Where It Appears in This PRD |
|---|---|
| Spring Boot REST APIs | All `/api/*` endpoints |
| JWT + Spring Security | Authentication + RBAC (Customer/Driver/Admin) |
| WebSocket (STOMP) | Live location + real-time notifications |
| PostgreSQL + PostGIS | Driver proximity query, location storage |
| JUnit + Mockito | Unit & integration tests |
| Docker multistage | `Dockerfile` + `docker-compose` |
| OpenAPI/Swagger | API documentation |
| Design patterns | Strategy (e.g., different order pricing), DTO mapping |

---

## 9. Optional Stretch Goals

- [ ] Add Stripe `PaymentIntent` (like your e-commerce project)
- [ ] Store full driver route as `LINESTRING` and replay trip on request
- [ ] Implement exponential backoff for WebSocket reconnection
- [ ] Deploy on Render.com or Railway with free-tier PostgreSQL
