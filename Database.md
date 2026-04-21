# 🗄️ Database Design (PostgreSQL)

This document defines the database schema for the **Manufacturing Workflow Management System**.

The system is designed for:

* Multi-workflow manufacturing environments
* Real-time tracking of machines and operators
* Inventory and storage management
* Quality inspection processes
---
ER-Diagram
```mermaid
    MACHINES {
        int id PK
        text name
        text type
        numeric cost_per_hour
        numeric power_consumption_kw
        text status
        date purchase_date
        text location
    }

    MACHINE_MAINTENANCE {
        int id PK
        int machine_id FK
        text schedule_type
        date last_maintenance_date
        date next_due_date
        text notes
    }

    MACHINE_LOGS {
        int id PK
        int machine_id FK
        int operator_id FK
        int workflow_id FK
        timestamp start_time
        timestamp end_time
        text status
    }

    OPERATORS {
        int id PK
        text name
        numeric hourly_rate
        text skill_level
        text status
    }

    OPERATOR_ACTIVITY {
        int id PK
        int operator_id FK
        int machine_id FK
        int workflow_id FK
        timestamp start_time
        timestamp end_time
        int units_produced
        int strokes
    }

    USERS {
        int id PK
        text name
        text role
        numeric hourly_rate
    }

    QUALITY_CHECKS {
        int id PK
        int inspector_id FK
        int workflow_id FK
        int item_id FK
        text status
        text defects
        timestamp timestamp
    }

    STORAGE_UNITS {
        int id PK
        text uid
        numeric height
        numeric width
        numeric thickness
        numeric max_weight
        text location
    }

    INVENTORY_ITEMS {
        int id PK
        text name
        text type
        numeric weight
        int customer_id
    }

    STORAGE_ALLOCATION {
        int id PK
        int storage_id FK
        int item_id FK
        int quantity
        timestamp timestamp
    }

    WORKFLOWS {
        int id PK
        text name
        int customer_id
        text status
        timestamp start_date
        timestamp end_date
    }

    WORKFLOW_STEPS {
        int id PK
        int workflow_id FK
        int machine_id FK
        int step_order
        numeric expected_time
    }

    WORKFLOW_EXECUTION {
        int id PK
        int workflow_id FK
        int step_id FK
        int operator_id FK
        int machine_id FK
        timestamp start_time
        timestamp end_time
        int output_units
        text status
    }

    WORKFLOW_TEMPLATES {
        int id PK
        text name
        text description
    }

    TEMPLATE_STEPS {
        int id PK
        int template_id FK
        text machine_type
        int step_order
        numeric expected_time
    }

    %% Relationships

    MACHINES ||--o{ MACHINE_MAINTENANCE : has
    MACHINES ||--o{ MACHINE_LOGS : logs
    MACHINES ||--o{ WORKFLOW_STEPS : used_in
    MACHINES ||--o{ WORKFLOW_EXECUTION : executes
    MACHINES ||--o{ OPERATOR_ACTIVITY : used_by

    OPERATORS ||--o{ OPERATOR_ACTIVITY : performs
    OPERATORS ||--o{ WORKFLOW_EXECUTION : assigned_to

    USERS ||--o{ QUALITY_CHECKS : performs

    WORKFLOWS ||--o{ WORKFLOW_STEPS : contains
    WORKFLOWS ||--o{ WORKFLOW_EXECUTION : tracks
    WORKFLOWS ||--o{ MACHINE_LOGS : linked_to
    WORKFLOWS ||--o{ OPERATOR_ACTIVITY : linked_to
    WORKFLOWS ||--o{ QUALITY_CHECKS : evaluated_in

    WORKFLOW_STEPS ||--o{ WORKFLOW_EXECUTION : executed_as

    STORAGE_UNITS ||--o{ STORAGE_ALLOCATION : holds
    INVENTORY_ITEMS ||--o{ STORAGE_ALLOCATION : stored_in

    INVENTORY_ITEMS ||--o{ QUALITY_CHECKS : inspected

    WORKFLOW_TEMPLATES ||--o{ TEMPLATE_STEPS : defines
```

---

# 📌 Design Principles

## 1. Normalization

The schema avoids redundancy and ensures data consistency using proper relational modeling.

## 2. Derived Data is NOT Stored

Metrics such as:

* Efficiency
* Machine utilization
* Cost per unit

are computed dynamically using queries.

## 3. Time-Based Tracking

All operational tables include:

* `start_time`
* `end_time`

This enables analytics, reporting, and optimization.

## 4. Event Logging

Separate log tables are used for:

* Machine activity
* Operator activity

---

# 🏭 Core Tables

## Machines

### `machines`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
type TEXT,
cost_per_hour NUMERIC,
power_consumption_kw NUMERIC,
status TEXT,
purchase_date DATE,
location TEXT
```

---

### `machine_maintenance`

```sql
id SERIAL PRIMARY KEY,
machine_id INT REFERENCES machines(id),
schedule_type TEXT,
last_maintenance_date DATE,
next_due_date DATE,
notes TEXT
```

---

### `machine_logs`

```sql
id SERIAL PRIMARY KEY,
machine_id INT REFERENCES machines(id),
operator_id INT,
workflow_id INT,
start_time TIMESTAMP,
end_time TIMESTAMP,
status TEXT
```

---

## Operators

### `operators`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
hourly_rate NUMERIC,
skill_level TEXT,
status TEXT
```

---

### `operator_activity`

```sql
id SERIAL PRIMARY KEY,
operator_id INT REFERENCES operators(id),
machine_id INT REFERENCES machines(id),
workflow_id INT,
start_time TIMESTAMP,
end_time TIMESTAMP,
units_produced INT,
strokes INT
```

---

## Users & Roles

### `users`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
role TEXT CHECK (role IN ('operator', 'inspector', 'admin')),
hourly_rate NUMERIC
```

---

## 🔍 Quality Control

### `quality_checks`

```sql
id SERIAL PRIMARY KEY,
inspector_id INT REFERENCES users(id),
workflow_id INT,
item_id INT,
status TEXT CHECK (status IN ('approved', 'rejected')),
defects TEXT,
timestamp TIMESTAMP
```

---

## 📦 Storage & Inventory

### `storage_units`

```sql
id SERIAL PRIMARY KEY,
uid TEXT UNIQUE,
height NUMERIC,
width NUMERIC,
thickness NUMERIC,
max_weight NUMERIC,
location TEXT
```

---

### `inventory_items`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
type TEXT CHECK (type IN ('raw_material', 'finished_good')),
weight NUMERIC,
customer_id INT
```

---

### `storage_allocation`

```sql
id SERIAL PRIMARY KEY,
storage_id INT REFERENCES storage_units(id),
item_id INT REFERENCES inventory_items(id),
quantity INT,
timestamp TIMESTAMP
```

---

## 🔄 Workflows (Core System)

### `workflows`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
customer_id INT,
status TEXT CHECK (status IN ('active', 'completed')),
start_date TIMESTAMP,
end_date TIMESTAMP
```

---

### `workflow_steps`

```sql
id SERIAL PRIMARY KEY,
workflow_id INT REFERENCES workflows(id),
machine_id INT REFERENCES machines(id),
step_order INT,
expected_time NUMERIC
```

---

### `workflow_execution`

```sql
id SERIAL PRIMARY KEY,
workflow_id INT REFERENCES workflows(id),
step_id INT REFERENCES workflow_steps(id),
operator_id INT REFERENCES operators(id),
machine_id INT REFERENCES machines(id),
start_time TIMESTAMP,
end_time TIMESTAMP,
output_units INT,
status TEXT
```

---

## 📄 Templates

### `workflow_templates`

```sql
id SERIAL PRIMARY KEY,
name TEXT NOT NULL,
description TEXT
```

---

### `template_steps`

```sql
id SERIAL PRIMARY KEY,
template_id INT REFERENCES workflow_templates(id),
machine_type TEXT,
step_order INT,
expected_time NUMERIC
```

---

# 🔗 Relationships Overview

* Machines → Machine Logs (1:N)
* Machines → Workflow Steps (1:N)
* Operators → Activity Logs (1:N)
* Workflows → Steps (1:N)
* Workflows → Execution (1:N)
* Storage Units → Allocation (1:N)
* Inventory Items → Allocation (1:N)
* Users → Quality Checks (1:N)

---

# 📊 Key Computations (Query-Level)

## Operator Efficiency

```sql
units_produced / EXTRACT(EPOCH FROM (end_time - start_time)) * 3600
```

## Machine Utilization

```sql
SUM(runtime) / total_available_time
```

## Cost Per Unit

```sql
(machine_cost_per_hour * runtime) / output_units
```

---

# ⚠️ Engineering Notes

* Index all foreign keys
* Use composite indexes for time-based queries
* Consider table partitioning for logs (`machine_logs`, `operator_activity`)
* Avoid storing computed metrics

---

# 🚀 Future Improvements

* Audit logging system
* Soft deletes (`is_deleted`, `deleted_at`)
* JSONB fields for flexible metadata
* Partitioning large log tables
* Triggers for automation (alerts, updates)

---

# ✅ Summary

This schema is designed to support:

* Concurrent manufacturing workflows
* Real-time machine and operator tracking
* Scalable analytics and reporting

It forms a **robust foundation for industrial-grade workflow management systems**.
