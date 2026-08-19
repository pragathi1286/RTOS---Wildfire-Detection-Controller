## A Real-Time Scheduling Framework for Wildfire Detection and Emergency Response

**RTOS Project – Review 1**
**Department of Computer Science and Engineering**
**KLH / KL University**

---

## 📌 Overview

Wildfires can spread rapidly, making timely detection and emergency response critical. Conventional sensor-monitoring systems often rely on periodic polling and simple task execution strategies, which may delay emergency alerts when the system is under heavy workload.

It is a real-time scheduling framework designed to prioritize wildfire emergency events over routine sensor-monitoring tasks.

The system models routine sensor monitoring as **periodic real-time tasks** and fire-detection events as **high-priority aperiodic tasks with strict deadlines**. It evaluates classical real-time scheduling algorithms such as **Rate-Monotonic Scheduling (RMS)** and **Earliest-Deadline-First (EDF)** against a simple **First-Come-First-Served (FCFS)** baseline.

The framework also demonstrates **priority inversion** and evaluates **Priority Inheritance (PI)** as a mechanism for reducing delays caused by shared-resource contention.

A simulated wildfire environment using a **Cellular Automaton model** and a low-cost **Arduino sensor layer** provides realistic task-generation scenarios for evaluating scheduler performance.

> **Core focus:** It focuses on **response timing and deadline compliance**, rather than wildfire detection accuracy.

---

# 🎯 Problem Statement

Wildfire monitoring systems may generate multiple sensor-processing tasks simultaneously. When these tasks are handled using simple scheduling approaches, an emergency fire alert can be delayed behind routine monitoring operations.

This creates a critical real-time systems problem:

> **How can an emergency wildfire alert be guaranteed or evaluated for timely execution when the system is already processing multiple sensor tasks?**

Addresses this problem by applying real-time scheduling theory to wildfire-response tasks and measuring their ability to meet predefined deadlines under varying system load conditions.

---

# 🔬 Research Question

**How effectively can real-time scheduling algorithms such as RMS and EDF reduce emergency-response latency and deadline misses compared with FCFS under increasing sensor workload?**

---

# 🎯 Objectives

The primary objectives of BLAZE-RT are:

1. Design a real-time task scheduling framework for wildfire-response scenarios.
2. Model routine sensor monitoring as periodic real-time tasks.
3. Model fire-detection events as high-priority aperiodic emergency tasks.
4. Implement and evaluate **Rate-Monotonic Scheduling (RMS)**.
5. Implement and evaluate **Earliest-Deadline-First (EDF)**.
6. Implement **FCFS** as a baseline scheduling strategy.
7. Demonstrate the problem of **priority inversion**.
8. Implement **Priority Inheritance (PI)** to reduce priority inversion.
9. Simulate wildfire propagation using a Cellular Automaton model.
10. Integrate Arduino-based sensors for real-world sensor input.
11. Measure response latency, deadline misses, and processor utilization.
12. Compare scheduler performance under increasing system workload.

---

# 🧠 Key Concepts

| Concept                  | Description                                                                                         |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| **Periodic Task**        | A task released at regular time intervals, such as temperature or humidity monitoring.              |
| **Aperiodic Task**       | A task triggered by an external event, such as a detected fire.                                     |
| **Hard Deadline**        | A time limit by which an emergency task must be completed.                                        |
| **RMS**                  | Assigns higher priority to tasks with shorter periods.                                              |
| **EDF**                  | Gives priority to the task with the earliest deadline.                                              |
| **FCFS**                 | Executes tasks according to their arrival order and serves as the baseline.                         |
| **Priority Inversion**   | A high-priority task is indirectly delayed because a lower-priority task holds a required resource. |
| **Priority Inheritance** | Temporarily raises the priority of the blocking task so it can release the resource sooner.         |
| **System Utilization**   | Percentage of processor capacity consumed by the task set.                                          |
| **Response Latency**     | Time between fire detection and execution of the emergency response.                                |
| **Deadline Miss**        | Occurs when a task finishes after its specified deadline.                                           |

---

# 🏗️ System Architecture

```text
                  ┌──────────────────────┐
                  │   Arduino Sensors    │
                  │                      │
                  │ Flame Sensor         │
                  │ DHT11 Temperature    │
                  │ DHT11 Humidity       │
                  └──────────┬───────────┘
                             │
                       Serial Data
                             │
                             ▼
                  ┌──────────────────────┐
                  │   Sensor Interface   │
                  │     pyserial         │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │     Task Generator   │
                  │                      │
                  │ Periodic Tasks       │
                  │ Emergency Tasks      │
                  └──────────┬───────────┘
                             │
                             ▼
        ┌────────────────────────────────────────┐
        │             BLAZE-RT CORE              │
        │                                        │
        │  ┌─────────┐  ┌─────────┐  ┌────────┐ │
        │  │  FCFS   │  │   RMS   │  │  EDF   │ │
        │  └─────────┘  └─────────┘  └────────┘ │
        │                                        │
        │       Priority Inheritance             │
        └──────────────────┬─────────────────────┘
                           │
                           ▼
                ┌────────────────────┐
                │ Emergency Response │
                │                    │
                │ Buzzer             │
                │ LCD Alert          │
                │ Dashboard          │
                └────────────────────┘
```

---

# 🔥 Wildfire Simulation

BLAZE-RT uses a **Cellular Automaton** model to simulate wildfire propagation.

The environment is represented as a grid where each cell can represent a different state:

| Cell State | Meaning              |
| ---------- | -------------------- |
| `0`        | Empty / non-burnable |
| `1`        | Vegetation           |
| `2`        | Burning              |
| `3`        | Burned               |

The simulation can generate fire events based on neighboring burning cells and environmental parameters.

The purpose of the simulation is **not to accurately predict real-world wildfire propagation**, but to generate realistic emergency events that can be supplied to the real-time scheduler.

---

# ⏱️ Real-Time Task Model

It separates tasks into two major categories.

### Periodic Tasks

Routine monitoring tasks execute at predefined intervals.

Example:

| Task                   |  Period | Priority | Type     |
| ---------------------- | ------: | -------: | -------- |
| Temperature Monitoring |  500 ms |      Low | Periodic |
| Humidity Monitoring    | 1000 ms |      Low | Periodic |
| Flame Sensor Check     |  500 ms |   Medium | Periodic |
| Dashboard Update       | 2000 ms |      Low | Periodic |

### Emergency Tasks

Fire-detection events are generated dynamically.

| Task                 | Trigger             | Priority | Deadline |
| -------------------- | ------------------- | -------- | -------: |
| Fire Alert           | Flame detected      | High     |   100 ms |
| Emergency Response   | Fire confirmed      | High     |   200 ms |
| Warning Notification | Emergency triggered | High     |   300 ms |

> The exact task periods, execution times, and deadlines will be determined during experimentation.

---

# ⚙️ Scheduling Algorithms

## 1. First-Come-First-Served (FCFS)

FCFS executes tasks according to their arrival order.

```text
Task A → Task B → Task C → Task D
```

FCFS is used as the **baseline** to demonstrate how emergency tasks may experience delays when routine tasks arrive first.

---

## 2. Rate-Monotonic Scheduling (RMS)

RMS assigns priority according to task period.

> **Shorter period → Higher priority**

Example:

```text
Task A → Period = 100 ms → Highest Priority
Task B → Period = 500 ms → Medium Priority
Task C → Period = 1000 ms → Low Priority
```

RMS is primarily applied to periodic sensor-monitoring tasks.

---

## 3. Earliest-Deadline-First (EDF)

EDF dynamically assigns priority according to the nearest deadline.

> **Earlier deadline → Higher priority**

Example:

```text
Task A → Deadline = 500 ms
Task B → Deadline = 100 ms  ← Executes first
Task C → Deadline = 300 ms
```

EDF is particularly useful for handling dynamically generated emergency tasks.

---

# 🔒 Priority Inversion

Priority inversion occurs when a high-priority task is indirectly blocked by a low-priority task holding a shared resource.

Example:

```text
Low Priority Task
       │
       │ locks shared resource
       ▼
   RESOURCE
       ▲
       │
High Priority FIRE Task
       │
       │ blocked
       ▼
  Response Delayed
```

A medium-priority task can make the situation worse by continuously executing while the low-priority task is unable to release the resource.

---

# 🛡️ Priority Inheritance

BLAZE-RT uses **Priority Inheritance** to handle priority inversion.

When a high-priority fire-response task is blocked by a low-priority task:

```text
Low Priority Task
       │
       │ holds resource
       ▼
High Priority Task arrives
       │
       │ blocked
       ▼
Priority Inheritance
       │
       ▼
Low Task temporarily receives
High Task's priority
       │
       ▼
Resource released
       │
       ▼
High Priority Fire Task executes
```

This allows the blocking task to complete its critical section faster and reduces the response delay experienced by the emergency task.

---

# 📊 Evaluation Metrics

BLAZE-RT evaluates the scheduling algorithms using the following metrics.

| Metric                      | Description                                                                |
| --------------------------- | -------------------------------------------------------------------------- |
| **Deadline-Miss Rate**      | Percentage of tasks that fail to complete before their deadlines.          |
| **Response Latency**        | Time between fire detection and emergency-task execution/response.         |
| **System Utilization**      | Fraction of processor capacity consumed by executing tasks.                |
| **Task Waiting Time**       | Time a task remains in the ready queue before execution.                   |
| **Emergency Response Time** | Time required for a detected fire event to trigger the response mechanism. |

### Deadline-Miss Rate

```text
Deadline Miss Rate =
(Number of missed deadlines / Total tasks) × 100
```

### System Utilization

For periodic tasks:

```text
U = Σ(Cᵢ / Tᵢ)
```

where:

* `Cᵢ` = execution time of task `i`
* `Tᵢ` = period of task `i`

---

# 🧪 Experimental Design

The main experiment compares three scheduling strategies:

```text
              ┌─────────────┐
              │ Task Set    │
              └──────┬──────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      FCFS          RMS          EDF
        │            │            │
        └────────────┼────────────┘
                     ▼
              Performance
               Evaluation
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Latency     Deadline    Utilization
                   Misses
```

The workload will be gradually increased to determine how each scheduler behaves under different processor-load conditions.

---

# 📈 Planned Comparison

| Scheduling Method | Priority Model                      | Dynamic Priority | Primary Use           | Expected Behavior                             |
| ----------------- | ----------------------------------- | ---------------- | --------------------- | --------------------------------------------- |
| **FCFS**          | Arrival order                       | No               | Baseline              | Emergency tasks may wait behind routine tasks |
| **RMS**           | Shorter period = higher priority    | No               | Periodic tasks        | Effective for fixed periodic workloads        |
| **EDF**           | Earliest deadline = higher priority | Yes              | Deadline-driven tasks | Better adaptation to dynamic deadlines        |

The final evaluation will determine the actual performance rather than assuming that one algorithm will always outperform another.

---

# 🧪 Priority Inheritance Experiment

A separate experiment will intentionally create a priority inversion scenario.

### Without Priority Inheritance

```text
Low Task locks resource
        ↓
High Fire Task arrives
        ↓
High Task becomes blocked
        ↓
Medium Task executes
        ↓
Low Task waits longer
        ↓
High Task response delayed
```

### With Priority Inheritance

```text
Low Task locks resource
        ↓
High Fire Task arrives
        ↓
High Task becomes blocked
        ↓
Low Task inherits High priority
        ↓
Low Task releases resource
        ↓
High Fire Task executes
```

The response latency before and after priority inheritance will be compared.

---


---

# 💻 Technology Stack

| Layer                | Technology                        |
| -------------------- | --------------------------------- |
| Programming Language | Python                            |
| Scheduling           | Custom RMS / EDF / FCFS scheduler |
| Real-Time Model      | Custom Task and Scheduler classes |
| Fire Simulation      | Cellular Automaton                |
| Hardware             | Arduino Uno                       |
| Sensors              | Flame Sensor, DHT11               |
| Communication        | Serial / PySerial                 |
| Visualization        | Pygame / Matplotlib               |
| Data Analysis        | Python                            |
| Development          | VS Code                           |
| Version Control      | Git / GitHub                      |

> AI tools such as ChatGPT and Claude may be used during development for planning, debugging, and documentation. They are **not runtime components of the scheduler**.

---

# 📁 Project Structure

```text
RTOS/
│
├── scheduler/
│   ├── task.py
│   ├── scheduler.py
│   └── priority_inheritance.py
│
├── simulation/
│   ├── fire_grid.py
│   └── clock.py
│
├── hardware/
│   ├── arduino_sketch/
│   │   └── blaze_rt.ino
│   └── serial_reader.py
│
├── dashboard/
│   └── visual_dashboard.py
│
├── evaluation/
│   └── metrics.py
│
├── tests/
│   └── test_scheduler.py
│
├── docs/
│   └── report/
│
├── requirements.txt
└── README.md
```

---

# 🗓️ Development Timeline

| Phase       | Activities                                                 | Duration |
| ----------- | ---------------------------------------------------------- | -------- |
| **Phase 1** | Literature survey, problem definition and requirements     | Week 1–2 |
| **Phase 2** | Task model, FCFS, RMS and EDF scheduler implementation     | Week 3   |
| **Phase 3** | Priority inversion and priority inheritance implementation | Week 4   |
| **Phase 4** | Cellular Automaton wildfire simulation                     | Week 5   |
| **Phase 5** | Arduino sensor integration and serial communication        | Week 6   |
| **Phase 6** | Dashboard and visualization                                | Week 7   |
| **Phase 7** | Experimental evaluation and comparison                     | Week 8   |
| **Phase 8** | Documentation, report and final presentation               | Week 8   |

---

# 🌟 Project Novelty

It focuses on **real-time response guarantees and schedulability**, rather than only improving fire-detection accuracy.

The main contributions are:

### 1. Real-Time Scheduling for Wildfire Response

Applies classical real-time scheduling algorithms to emergency wildfire-response tasks.

### 2. Deadline-Oriented Evaluation

Measures whether emergency tasks meet predefined deadlines under increasing workload.

### 3. Priority Inversion Demonstration

Explicitly models the priority inversion problem within an emergency-response scenario.

### 4. Priority Inheritance

Demonstrates how Priority Inheritance can reduce delays caused by shared-resource contention.

### 5. Hardware + Simulation

Combines real sensor input from Arduino with a simulated wildfire environment.

### 6. Reproducible Experiments

Uses controlled task sets and measurable performance metrics so scheduler behavior can be reproduced and compared.

---

# 🎯 Expected Results

The project will experimentally investigate:

* Whether FCFS causes greater emergency-task waiting times under heavy load.
* Whether RMS can maintain deadline compliance for periodic sensor tasks.
* Whether EDF adapts better to tasks with different deadlines.
* How increasing processor utilization affects deadline-miss rate.
* How priority inversion affects emergency response latency.
* How Priority Inheritance reduces the blocking time of high-priority tasks.
* How scheduler performance changes as the number of sensor tasks increases.

The conclusions will be based on experimental measurements rather than predetermined assumptions.

---

# 🚀 Future Scope

It can be extended with:

* Multi-core real-time scheduling.
* Wireless sensor networks.
* LoRa-based long-range communication.
* Multiple distributed sensor nodes.
* Adaptive wildfire-risk estimation.
* Edge computing deployment.
* Real-time geographic mapping.
* Dynamic task creation based on multiple sensor events.
* Fault-tolerant emergency communication.
* Integration with autonomous drones or robotic response systems.

---

# 📚 References

1. Kalyanasundaram et al., *A Survey on Scheduling Algorithms in Real-Time Systems*.
2. Avazov et al., *An Edge Computing Environment for Early Wildfire Detection*.
3. C. L. Liu and J. W. Layland, *Scheduling Algorithms for Multiprogramming in a Hard-Real-Time Environment*, Journal of the ACM, 1973.
4. Jane W. S. Liu, *Real-Time Systems*, Prentice Hall.
5. Burns and Wellings, *Real-Time Systems and Their Programming Languages*.

---

# 📌 Current Status

🚧 **In Development — RTOS Project Review 1**

### Completed / Planned

| Component              | Status         |
| ---------------------- | -------------- |
| Problem Definition     | ✅ Completed    |
| System Architecture    | ✅ Designed     |
| Task Model             | 🔄 In Progress |
| FCFS Scheduler         | 🔄 Planned     |
| RMS Scheduler          | 🔄 Planned     |
| EDF Scheduler          | 🔄 Planned     |
| Priority Inheritance   | 🔄 Planned     |
| Fire Simulation        | 🔄 Planned     |
| Arduino Integration    | 🔄 Planned     |
| Dashboard              | 🔄 Planned     |
| Performance Evaluation | ⏳ Pending      |
| Final Documentation    | ⏳ Pending      |

---

# 👥 Project Focus

**It is primarily a Real-Time Operating Systems / Scheduling project.**

The wildfire scenario provides the application context, while the core technical contribution lies in:

**Task Modeling → Scheduling → Priority Management → Deadline Analysis → Performance Evaluation**

---

## 🔥 One-Line Summary

> **It is a real-time scheduling framework that evaluates how FCFS, RMS, and EDF handle wildfire emergency tasks under deadline constraints, while demonstrating priority inversion and Priority Inheritance using simulated and physical sensor workloads.**
