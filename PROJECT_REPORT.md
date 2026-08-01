# Project Report — WareFlow
## Optimizing Material Flow in Industrial Warehousing: A Human-Centric Coordination Platform

**Project ID:** TCMAJA1352
**Department:** Computer Science & Engineering

---

## Chapter 1: Introduction

Modern warehouses coordinate four distinct groups of people — administrators, suppliers,
warehouse employees, and customers — around a constantly changing pool of physical stock.
When this coordination is manual or spreadsheet-based, it leads to delayed order fulfilment,
misplaced inventory, and workers walking further than necessary to pick items. WareFlow is a
full-stack platform that digitizes this coordination end-to-end, with a particular focus on
*where* inventory is stored, not just *how much* of it exists.

## Chapter 2: Objectives

(Official objective, as assigned)

The primary objective of the project is to enhance warehouse and supply chain operations for
improved efficiency. The objective is to facilitate precise and real-time tracking of
inventory, shipments and orders. The system is designed to enhance coordination among
administrators, suppliers, employees and customers. It is designed to optimize resource
utilization, storage space and workflow management. It also aims to reduce errors and delays
by automating routine tasks and providing continuous monitoring of operations. The project
also intends to provide actionable insights through analytics and reporting.

## Chapter 3: Literature Survey

- Traditional Warehouse Management Systems (WMS) focus on transaction logging (in/out) without
  addressing *where* items should physically live.
- ABC/Pareto analysis is a well-established inventory technique: a small percentage of SKUs
  account for the majority of order volume, and warehousing literature recommends storing
  those SKUs closest to the pack/dispatch area to minimize picker travel time.
- Task-assignment literature in logistics favors load-balancing (assigning work to the
  least-busy worker) over static assignment, which this project implements directly.

## Chapter 4: Problem Statement

Existing systems either (a) track inventory quantities without any placement intelligence, or
(b) provide placement optimization as a standalone academic model that is never connected to
an actual operational system that workers use day-to-day. There is a gap between "optimization
research" and "a system a warehouse employee can actually open and follow."

## Chapter 5: Proposed Solution — System Architecture

- **Backend**: Spring Boot 3 REST API (Java 17), Spring Security with JWT-based auth,
  role-based access control (ADMIN / SUPPLIER / EMPLOYEE / CUSTOMER)
- **Database**: MySQL, via Spring Data JPA / Hibernate
- **Frontend**: React 18 single-page app, role-aware navigation and dashboards
- **Slot Optimization Engine**: an ABC-classification heuristic (`SlotOptimizationService`)
  that ranks products by 30-day sales velocity and recommends storage bins by their distance
  to the dispatch dock — Class A (fastest movers) get the closest bins, Class C (slowest) the
  farthest, freeing prime space for what is actually picked most often.
- **Task Coordination Engine**: PUTAWAY tasks are generated automatically when a shipment is
  received; PICK tasks are generated automatically when an order is placed (allocated
  nearest-dispatch-bin-first across multiple bins if needed); every task is assigned to the
  least-loaded employee.

### Data flow

1. Supplier creates a Shipment (expected) → Admin/Employee marks it Received → system
   recommends a bin per line item → PUTAWAY tasks created → Employee completes them → stock
   increases in the recommended bins.
2. Customer places an Order → system checks total stock across bins → PICK tasks created
   (nearest-to-dispatch bin drawn from first) → Employee completes them → stock decreases.
3. Every sale increments the product's rolling `unitsSoldLast30Days` counter, which feeds back
   into the ABC classification for future putaway recommendations — the system adapts to
   changing demand over time.

## Chapter 6: Results

- End-to-end tested flow: registration for all 4 roles, warehouse/bin setup, shipment →
  putaway → order → pick, with live stock updates at every step.
- Dashboard analytics: low-stock alert count, pending/in-progress/completed task counts, task
  completion rate, and top-5 moving products with live ABC class badges.
- Slot optimization verified manually: products with high `unitsSoldLast30Days` are
  consistently recommended the lowest-`dispatchDistance` bins with available capacity.

## Chapter 7: Conclusion

WareFlow demonstrates that the "optimize resource utilization and storage space" objective
does not need to be a separate research exercise — a simple, explainable ABC-velocity
heuristic, wired directly into the putaway and picking workflows, is enough to make measurable
placement decisions inside a real, working multi-role application.

**Limitations & future scope**: velocity is currently computed from a rolling counter rather
than a true time-windowed query; a production version would use a scheduled job to decay old
sales data. A path-optimization module (multi-item pick-route sequencing within a single
order, e.g. via nearest-neighbour or 2-opt) would be a natural next step beyond single-item
bin recommendation.

## Chapter 8: References

1. Frazelle, E. *World-Class Warehousing and Material Handling*, McGraw-Hill.
2. Tompkins, J. A. et al., *Facilities Planning*, Wiley.
3. Spring Boot Reference Documentation — https://docs.spring.io/spring-boot/
4. React Documentation — https://react.dev
5. de Koster, R., Le-Duc, T., Roodbergen, K.J., "Design and control of warehouse order
   picking: A literature review," *European Journal of Operational Research*, 2007.
