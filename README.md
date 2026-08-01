# WareFlow — Human-Centric Warehouse Coordination Platform

Optimizing material flow in industrial warehousing: a multi-role platform (Admin, Supplier,
Employee, Customer) for real-time inventory tracking, shipment/order coordination, and
data-driven storage-slot optimization.

**Project code:** TCMAJA1352 — Java / Web Applications

## Architecture

- **Backend**: Spring Boot 3 (Java 17), Spring Security + JWT, Spring Data JPA, MySQL
- **Frontend**: React 18, React Router, plain CSS (industrial design system)
- Single service (no separate AI microservice) — the slot-optimization logic lives directly
  in the Java backend as an ABC-classification heuristic (see `SlotOptimizationService.java`)

## Core features

- **Multi-role auth**: Admin, Supplier, Employee, Customer — each with a tailored dashboard/nav
- **Inventory tracking**: products, bins, live stock levels, low-stock alerts
- **Slot optimization**: fast-moving (Class A) products are automatically recommended bins
  closest to the dispatch dock; slow movers (Class C) go to the farthest bins — freeing up
  prime warehouse real-estate for what actually gets picked most
- **Shipment coordination**: suppliers send shipments → receiving auto-generates PUTAWAY
  tasks with AI-recommended bins → employees complete them, updating live stock
- **Order coordination**: customers place orders → system checks stock → auto-generates PICK
  tasks (nearest-to-dispatch bin first) → employees complete them → stock decrements
- **Task load-balancing**: every new task is auto-assigned to the least-loaded employee
- **Analytics dashboard**: low-stock count, task completion rate, top-moving products, orders
  in progress, shipments expected

## Running locally

### 1. Backend (port 8081)
Edit `backend/src/main/resources/application.properties` if your MySQL username/password
differ from `root`/`root`.

```bash
cd backend
mvn spring-boot:run
```

### 2. Frontend (port 3000, or 3001 if Vaultwise is also running)

```bash
cd frontend
npm install
npm start
```

## Demo flow

1. Register 4 accounts, one per role (Admin, Supplier, Employee, Customer)
2. As **Admin**: create a Warehouse, then a couple of Storage Bins (vary the "dispatch
   distance" — e.g. 2, 8, 20 — to see optimization in action). Add a few Products.
3. As **Supplier**: send a Shipment with some product quantities
4. As **Admin/Employee**: mark the shipment "Received" — watch PUTAWAY tasks appear with
   bin recommendations
5. As **Employee**: Start → Complete the putaway tasks — stock now shows in Products
6. As **Customer**: place an Order for one of those products — watch PICK tasks generate
7. As **Employee**: complete the pick tasks — order stock decrements
8. Check the **Dashboard** (any role) for live analytics

## Notes for the project report

- The optimization module (`SlotOptimizationService`) implements ABC/Pareto velocity
  classification — a standard, real-world warehousing technique — as the "optimize resource
  utilization, storage space" objective from the assigned brief.
- Ports: backend runs on **8081** (not 8080) so it can run alongside other Java projects
  (e.g. Vaultwise) without a port clash.
