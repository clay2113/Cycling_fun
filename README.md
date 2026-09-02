# 🚴 BREAKAWAY

### Cycling Race Strategy & Aerodynamic Analysis Engine

> **Upload a ride. Find where to attack. Measure how aerodynamic you were. Simulate how much faster you could be.**

BREAKAWAY is a cycling analytics engine that processes TCX ride files and combines **data structures & algorithms, cycling physics, numerical analysis, and interactive visualization** to analyze a ride from two perspectives:

* 🏁 **Race Strategy** — identifies tactically valuable attack, recovery, and sprint sections.
* 💨 **Aerodynamics** — estimates local and overall CdA from power, speed, elevation, and environmental data.
* ⚡ **Performance Simulation** — estimates the effect of changing power, CdA, weight, or other parameters.

The project is deliberately designed so that the algorithms are not hidden behind the UI. Every major recommendation can be traced back to the underlying data, algorithm, and physical model.

---

## ✨ Features

### 🏁 Race Strategy Engine

BREAKAWAY segments a ride into smaller sections and evaluates each section using:

* Gradient
* Elevation change
* Speed
* Power
* Energy expenditure
* Local route characteristics
* Tactical potential

The engine ranks candidate attack locations and produces recommendations such as:

```text
🟢 CONSERVE
🟡 PREPARE
🔴 ATTACK
⚡ SPRINT
🔵 RECOVER
```

Example:

```text
12.4 km

Gradient:       +4.8%
Speed:           31.7 km/h
Power:             342 W

Attack Score:       94/100

Recommendation:   ATTACK
```

---

### 💨 Aerodynamic Analysis

The Aero Engine estimates cycling CdA from ride telemetry.

Inputs can include:

* Rider weight
* Bike/equipment weight
* Power
* Speed
* Elevation
* Temperature
* Atmospheric pressure/altitude
* Rolling resistance coefficient
* Drivetrain efficiency

The model uses a cycling power balance involving:

$$
P_{total}
=
P_{aero}
+
P_{rolling}
+
P_{gravity}
$$

with aerodynamic power approximated by:

$$
P_{aero}
=
\frac12\rho C_dA v^3
$$

and rolling resistance by:

$$
P_{rolling}
=
C_{rr}mgv
$$

The resulting CdA should be treated as an **estimate**, because real-world field measurements are sensitive to wind, road gradient, power-meter accuracy, speed measurement, rolling resistance, drivetrain losses, air density, and rider movement.

---

## 📊 Local CdA Analysis

Instead of producing a single CdA number for an entire ride, BREAKAWAY evaluates rolling windows of telemetry.

Example:

```text
Distance       Estimated CdA

5.0 km         0.231
7.0 km         0.224
9.0 km         0.218
11.0 km        0.207
13.0 km        0.201
```

This makes it possible to identify:

* Best aerodynamic sections
* Worst aerodynamic sections
* Changes in riding position
* Potential sensor anomalies
* Stable sections suitable for comparison

---

## ⚡ Aero Simulator

BREAKAWAY can simulate hypothetical changes.

For example:

```text
Current CdA      0.214 m²
Target CdA       0.190 m²
Power              300 W
```

The system estimates the corresponding change in speed and theoretical time over a chosen distance.

This turns the application from a passive ride analyzer into an interactive performance simulator.

---

# 🧠 Data Structures & Algorithms

One of the main goals of BREAKAWAY is to apply DSA to a real-world problem rather than implementing algorithms in isolation.

## 1. Prefix Sums

Used for efficiently calculating aggregate telemetry over route intervals.

Applications:

* cumulative distance
* cumulative elevation
* rolling statistics

Complexity:

```text
Preprocessing: O(N)
Range query:   O(1)
```

---

## 2. Sliding Window

Used for local route analysis.

Examples:

```text
average gradient over 300 m
average power over 30 seconds
local CdA over 60 seconds
```

Instead of recalculating every window:

```text
O(N × W)
```

the implementation maintains the current window incrementally:

```text
O(N)
```

---

## 3. Monotonic Deque

A monotonic deque is used to efficiently identify extreme values inside route windows.

Applications:

* steepest sustained sections
* maximum local gradient
* minimum/maximum telemetry values
* candidate attack sections

Complexity:

```text
O(N)
```

because each element enters and leaves the deque at most once.

---

## 4. Priority Queue

Candidate attack sections are ranked using a max-heap.

Conceptually:

```text
Candidate
    ↓
Attack Score
    ↓
Max Heap
    ↓
Best attack locations
```

Complexity:

```text
Insertion: O(log N)
Top candidate: O(1)
```

---

## 5. Dynamic Programming

The strategy engine models energy as a limited resource.

A simplified state can be represented as:

```text
dp[position][energy]
```

At each segment the cyclist can choose an action:

```text
CONSERVE
NORMAL
ATTACK
SPRINT
```

The transition evaluates the tactical reward and energy cost of each decision.

The objective is to maximize expected tactical advantage while respecting the available energy budget.

This transforms the problem from:

> "Which section looks steepest?"

into:

> "Given limited energy, where should I spend it?"

---

## 6. Rolling Median / Two Heaps

Real-world sensor data contains noise and outliers.

For example:

```text
0.214
0.218
0.211
0.743   ← abnormal measurement
0.216
0.213
```

A rolling median can be used to reduce the influence of extreme values.

The implementation can use two heaps:

```text
Max Heap → lower half
Min Heap → upper half
```

---

## 7. Binary Search

Used for efficient lookup of telemetry corresponding to:

* route distance
* timestamp
* segment boundaries
* nearest recorded point

Complexity:

```text
O(log N)
```

---

# 🏗️ System Architecture

```text
                         ┌───────────────┐
                         │   TCX FILE    │
                         └───────┬───────┘
                                 │
                                 ▼
                       ┌──────────────────┐
                       │   TCX PARSER     │
                       └────────┬─────────┘
                                │
                                ▼
                       ┌──────────────────┐
                       │ TELEMETRY MODEL  │
                       │                  │
                       │ GPS              │
                       │ Time             │
                       │ Speed            │
                       │ Power            │
                       │ Elevation        │
                       └────────┬─────────┘
                                │
                 ┌──────────────┴──────────────┐
                 ▼                             ▼
       ┌───────────────────┐         ┌───────────────────┐
       │ STRATEGY ENGINE   │         │    AERO ENGINE    │
       │                   │         │                   │
       │ Sliding Window    │         │ Cycling Physics   │
       │ Monotonic Deque   │         │ Air Density       │
       │ Priority Queue    │         │ Rolling Resistance│
       │ Dynamic Programming│        │ CdA Estimation    │
       └─────────┬─────────┘         └─────────┬─────────┘
                 │                             │
                 └──────────────┬──────────────┘
                                ▼
                         ┌───────────────┐
                         │   FASTAPI     │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │     REACT     │
                         │   Dashboard   │
                         └───────────────┘
```

---

# 🛠️ Technology Stack

## Frontend

| Technology     | Purpose                   |
| -------------- | ------------------------- |
| React          | UI architecture           |
| TypeScript     | Type safety               |
| Vite           | Development/build tooling |
| Tailwind CSS   | UI styling                |
| Recharts       | Telemetry visualizations  |
| MapLibre GL JS | Interactive route maps    |

## Backend

| Technology | Purpose               |
| ---------- | --------------------- |
| Python     | Analysis engine       |
| FastAPI    | REST API              |
| Pydantic   | Data validation       |
| NumPy      | Numerical computation |
| Pandas     | Telemetry processing  |
| pytest     | Testing               |

## Engineering

```text
Git
GitHub
ESLint
Prettier
pytest
REST APIs
JSON
XML
TypeScript
OOP
Unit Testing
```

---

# 📁 Project Structure

```text
breakaway/
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── types/
│   └── package.json
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── algorithms/
│   │   ├── parsing/
│   │   ├── physics/
│   │   ├── models/
│   │   └── services/
│   │
│   └── tests/
│
├── data/
│   └── sample.tcx
│
├── docs/
│   ├── algorithms.md
│   └── physics.md
│
└── README.md
```

---

# 🔬 Development Philosophy

BREAKAWAY follows a simple principle:

> **Every visual feature should correspond to a meaningful computation.**

The UI is not designed merely to display numbers.

For example:

```text
Attack marker
      ↓
Candidate detection
      ↓
Priority Queue
      ↓
Energy DP
      ↓
Attack Score
```

Similarly:

```text
CdA graph
      ↓
Sliding Window
      ↓
Physics model
      ↓
Outlier filtering
      ↓
Local CdA estimate
```

This keeps the visualization tightly connected to the engineering underneath.

---

# 🚀 Future Improvements

Possible future versions could include:

* Wind estimation
* Wind direction
* Yaw-angle aerodynamic modelling
* Power-meter uncertainty
* Multiple ride comparison
* Position/aero comparison
* FTP-aware strategy modelling
* Crit circuit strategy
* Team tactics
* Breakaway probability
* GPX support
* Live sensor ingestion
* Machine-learning based strategy prediction

---

# ⚠️ Limitations

The aerodynamic model is intended for **analysis and experimentation**, not laboratory-grade aerodynamic measurement.

Field CdA estimation is affected by:

* Wind
* Road gradient
* Power-meter accuracy
* Speed sensor/GPS accuracy
* Rolling resistance
* Drivetrain efficiency
* Air density
* Rider movement
* Road surface
* Temperature and atmospheric conditions

Results should therefore be interpreted as estimates under the assumptions used by the model.

---

# 🎯 Learning Objectives

Building BREAKAWAY is intended to develop practical understanding of:

### Data Structures & Algorithms

* Sliding Window
* Prefix Sums
* Monotonic Queue
* Heap / Priority Queue
* Dynamic Programming
* Binary Search
* Two Heaps

### Computer Science

* Object-Oriented Programming
* Type Systems
* REST APIs
* Client/Server Architecture
* Data Serialization
* XML Parsing
* Numerical Computing
* Testing
* Algorithmic Complexity

### Cycling Science

* Aerodynamic drag
* CdA
* Rolling resistance
* Gravity
* Power balance
* Air density
* Gradient
* Cycling performance modelling

### Software Engineering

* Modular architecture
* API design
* Separation of concerns
* Unit testing
* Error handling
* Git/GitHub workflow
* Documentation

---

# 📌 Status

🚧 **Currently under development**

The project is being developed incrementally, beginning with the telemetry processing and algorithmic engine before adding the interactive frontend.

---

## License

MIT License
