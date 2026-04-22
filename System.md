# ERP – Niranjan Auto

A full-stack manufacturing ERP system designed to streamline production workflows, inventory management, and job scheduling for an automobile parts manufacturing company.

---

## 🚀 Overview

This system is built to digitize and optimize the end-to-end manufacturing lifecycle:

* Raw material intake
* Workflow-based production
* Machine-level task execution
* Quality inspection (scrap vs reusable)
* Finished goods inventory

The application is designed as a **modular monolith using Next.js**, with a strong emphasis on **traceability, configurability, and real-time production visibility**.

---

## 🧠 Core Concepts

### 1. Inventory-Centric Architecture

All operations revolve around:

* **Inventory**
* **Transformations (Workflows)**
* **Traceability (Transactions & Batches)**

---

### 2. Workflow-Driven Manufacturing

Production is modeled as a sequence of steps:

```
RAW MATERIAL → CUT → PROCESS → FINISH → OUTPUT
```

Each step:

* Consumes input
* Produces output
* Generates byproducts (scrap/reusable)

---

### 3. Template-Based Workflows

Workflows are reusable via templates:

* Templates define the process
* Instances execute per order
* Versioning ensures historical consistency

---

### 4. Event-Driven Execution

All actions (production, scrap, transfer) are recorded as **inventory transactions**, ensuring full auditability.

---

## 🏗️ System Architecture

### Monolithic Structure (Next.js)

```
Next.js App
│
├── UI (App Router)
├── API Routes (Controllers)
├── Service Layer (Business Logic)
└── Background Workers (Queue Processing)
```

---

### Key Components

#### 1. Frontend

* Built with Next.js (App Router)
* Zustand for state management
* Role-based dashboards

#### 2. Backend (Inside Next.js)

* API Routes as controllers
* Service layer for business logic
* MongoDB for persistence

#### 3. Queue System

* BullMQ + Redis
* Handles:

  * Workflow execution
  * Machine scheduling
  * Background processing

#### 4. Worker Process

Runs independently:

```
node worker.js
```

Responsible for:

* Processing machine jobs
* Updating workflow states
* Managing step transitions

---

## 📦 Core Modules

---

### 1. Inventory Management

Tracks:

* Raw Materials
* Work In Progress (WIP)
* Finished Goods
* Scrap
* Reusable Byproducts

#### Inventory Transactions

All changes are logged:

```
CONSUME | PRODUCE | SCRAP | TRANSFER
```

---

### 2. Product & BOM (Bill of Materials)

Defines:

* Product dimensions
* Required raw material
* Production rules

---

### 3. Workflow Engine

#### Workflow Template

Reusable blueprint:

```
Step 1 → Machine A → Horizontal Cut  
Step 2 → Machine B → Vertical Cut  
Step 3 → Machine C → Press  
Step 4 → Machine D → Finish  
```

---

#### Workflow Instance

Created per order:

* Independent execution
* Tracks real-time progress
* Stores actual outputs, scrap, and timestamps

---

### 4. Yield Calculation Engine

Calculates output based on material dimensions:

Example:

```
Sheet: 1000 × 1000  
Cut Size: 100 × 100  

→ 10 × 10 = 100 units per sheet
```

Used in:

* Workflow creation
* Inventory planning
* Production estimation

---

### 5. Work Orders

Represents customer demand:

* Product
* Quantity
* Deadline
* Workflow instance

---

### 6. Machine & Operator Management

Tracks:

* Machine capacity
* Status (available, busy, maintenance)
* Operator assignment

---

### 7. Scheduling System

* Queue-based execution per machine
* FIFO scheduling (initial)
* Supports concurrent workflows

---

### 8. Quality Inspection

After each step:

* Output is inspected
* Classified into:

  * Usable output
  * Reusable byproduct
  * Scrap

---

### 9. Batch Tracking

Each production run is tracked via batches:

* Enables traceability
* Supports defect analysis
* Allows recall if needed

---

## 🔄 Workflow Execution Flow

1. Create Work Order
2. Select Workflow Template
3. Generate Workflow Instance
4. Reserve Inventory
5. Push Jobs to Queue
6. Execute Step-by-Step via Worker
7. Perform Inspection
8. Update Inventory
9. Complete Production

---

## 🧩 Data Model (High-Level)

### Inventory

```
InventoryItem
- type (RAW | WIP | FINISHED | SCRAP | REUSABLE)
- quantity
- dimensions
- location
```

---

### Transactions

```
InventoryTransaction
- type
- quantity
- source
- destination
- reference_id
```

---

### Workflow

```
WorkflowTemplate
WorkflowTemplateStep

WorkflowInstance
WorkflowStepInstance
```

---

### Production

```
WorkOrder
Batch
Task / Step Execution
```

---

## ⚙️ Key Features

* Dynamic workflow builder
* Real-time production tracking
* Inventory-aware planning
* Scrap & reusable material handling
* Machine-level scheduling
* Template versioning
* Full audit trail via transactions

---

## 🔐 Roles & Permissions

* **Admin** → Full control
* **Operator** → Execute machine tasks
* **Quality Inspector** → Classify outputs
* **Manager** → Monitor operations

---

## 🚧 Development Phases

### Phase 1

* Products & Inventory
* Yield calculation

### Phase 2

* Workflow templates
* Work orders

### Phase 3

* Inventory transactions
* Scrap handling
* Inspection system

### Phase 4

* Scheduling engine
* Machine queues

### Phase 5

* Optimization & analytics

---

## ⚠️ Design Constraints

* No direct inventory mutation (use transactions only)
* Templates are immutable once used (versioning required)
* Workflow execution must be isolated per order
* Background jobs must not run in API request lifecycle

---

## 🛠️ Tech Stack

* **Frontend**: Next.js, Zustand
* **Backend**: Next.js API Routes (Service Layer)
* **Database**: MongoDB
* **Queue**: BullMQ + Redis

---

## 📌 Future Enhancements

* Advanced scheduling algorithms
* Material nesting optimization
* Multi-warehouse support
* Real-time WebSocket updates
* IoT integration with machines

---

## 🧾 Summary

This ERP system is designed as a **production-grade manufacturing solution**, focusing on:

* Accuracy
* Traceability
* Flexibility
* Scalability (within controlled scope)

It provides a strong foundation for managing complex manufacturing workflows while remaining maintainable and extensible.

---
