<h1 align="center">Source Code Standards — Guideline</h1>

---

> [!WARNING]
> Follow this guideline for **all** thesis project source code in the BMW Lab. Code that does not meet these standards will not pass the handover review.

> [!NOTE]
> **Relationship to Project Documentation:**
> The class diagram, flowcharts, and system parameters in [project-documentation.md](./project-documentation.md) are the design contract. Source code must implement exactly what those diagrams describe — names, interfaces, and responsibilities must match.

---

## Table of Contents

- [1. Design-First Approach](#1-design-first-approach)
- [2. Object-Oriented Programming (OOP)](#2-object-oriented-programming-oop)
- [3. Design Patterns](#3-design-patterns)
  - [3.1 Adapter Pattern](#31-adapter-pattern)
  - [3.2 Abstract Factory Pattern](#32-abstract-factory-pattern)
  - [3.3 Strategy Pattern](#33-strategy-pattern)
- [4. Code Documentation (Sphinx / Doxygen)](#4-code-documentation-sphinx--doxygen)
- [5. Folder Structure](#5-folder-structure)
- [6. General Architecture](#6-general-architecture)
- [7. Production Readiness](#7-production-readiness)
  - [7.1 Container Images](#71-container-images)
  - [7.2 Git Submodule Initialization](#72-git-submodule-initialization)
  - [7.3 Known Issues & Workarounds](#73-known-issues--workarounds)
  - [7.4 API Endpoint Naming Consistency](#74-api-endpoint-naming-consistency)
  - [7.5 Secrets & Environment Variables](#75-secrets--environment-variables)
- [8. Standards & Parameter Reference Convention](#8-standards--parameter-reference-convention)

---

## 1. Design-First Approach

Before writing any code, produce the following artifacts **in order**. These belong in [project-documentation.md](./project-documentation.md).

| Order | Artifact | Purpose |
| ----- | -------- | ------- |
| 1 | **Flowchart** | Define the overall program logic — initialization → processing → output |
| 2 | **Class Diagram** | Define classes, attributes, methods, and relationships |
| 3 | **System Parameters Table** | Define all inputs/outputs with 3GPP/IEEE standard references |

> [!CAUTION]
> Do not start implementation until the flowchart and class diagram are reviewed and approved in a weekly meeting.

**Flowchart Reference:** [Introduction to Flowcharts](https://www.geeksforgeeks.org/competitive-programming/an-introduction-to-flowcharts/)

---

## 2. Object-Oriented Programming (OOP)

Structure all code around OOP principles. Reference: [Python OOP Guide](https://realpython.com/python3-object-oriented-programming/)

**Required before implementation:**

- Define every class from the class diagram
- Specify attributes following 3GPP parameter naming (e.g., `DRB.PrbUtilDL`)
- Define all methods that correspond to use cases in the MSC
- Establish relationships: inheritance (`is-a`), composition (`has-a`), aggregation

**OOP Principles to follow:**

| Principle | Application |
| --------- | ----------- |
| **Encapsulation** | Group related data and methods in one class; use `private`/`protected` access modifiers |
| **Abstraction** | Define abstract base classes or interfaces for components with multiple implementations |
| **Inheritance** | Use class hierarchies (e.g., `BaseRApp` → `EnergySavingRApp`) |
| **Polymorphism** | Define overridable methods for vendor-specific or platform-specific behavior |

---

## 3. Design Patterns

For rApp and xApp development, three design patterns are required to keep the framework modular and extensible.

Reference: [Refactoring.Guru Design Patterns](https://refactoring.guru/design-patterns/)

### 3.1 Adapter Pattern

**Purpose:** Convert proprietary vendor parameters into a common model (e.g., 3GPP KPIs), so the same rApp logic works across different platforms.

Reference: [Adapter Pattern](https://refactoring.guru/design-patterns/adapter)

**When to use:**

- Converting proprietary gNB telemetry → 3GPP KPIs
- Converting VIAVI / simulator parameters → common rApp input format

**Example:**

```python
class GnbTelemetryAdapter:
    """Adapts vendor-specific gNB telemetry to standard 3GPP KPIs."""

    def __init__(self, vendor_client: VendorTelemetryClient):
        self._client = vendor_client

    def get_prb_util_dl(self) -> float:
        """Returns DRB.PrbUtilDL (TS 28.552 §5.1.1.12.1) as a float percentage."""
        raw = self._client.get_metric("dl_prb_usage_pct")
        return float(raw)
```

### 3.2 Abstract Factory Pattern

**Purpose:** Create platform-specific modules (simulation, testbed, test equipment) without changing the core rApp logic.

Reference: [Abstract Factory Pattern](https://refactoring.guru/design-patterns/abstract-factory)

**Platforms to support:**

- `Ns3Platform` — ns-3 simulation environment
- `ViaviPlatform` — VIAVI RSG test equipment
- `PhysicalPlatform` — real gNB testbed

**Each factory creates compatible modules:**

- `ScenarioRunner` — controls the simulation or test scenario
- `TelemetryCollector` — collects raw KPI data from the platform
- `KpiAnalyzer` — processes raw data into standardized KPIs

**Example:**

```python
class RAppPlatformFactory(ABC):
    """Abstract factory for creating platform-specific rApp components."""

    @abstractmethod
    def create_scenario_runner(self) -> ScenarioRunner: ...

    @abstractmethod
    def create_telemetry_collector(self) -> TelemetryCollector: ...

    @abstractmethod
    def create_kpi_analyzer(self) -> KpiAnalyzer: ...


class Ns3PlatformFactory(RAppPlatformFactory):
    """Factory for ns-3 simulation environment."""

    def create_scenario_runner(self) -> ScenarioRunner:
        return Ns3ScenarioRunner()

    def create_telemetry_collector(self) -> TelemetryCollector:
        return Ns3TelemetryCollector()

    def create_kpi_analyzer(self) -> KpiAnalyzer:
        return Ns3KpiAnalyzer()
```

### 3.3 Strategy Pattern

**Purpose:** Switch between different evaluation or optimization algorithms without modifying the rApp framework.

Reference: [Strategy Pattern](https://refactoring.guru/design-patterns/strategy)

**When to use:**

- Swapping traffic steering algorithms
- Testing different handover optimization policies
- Comparing ML models for energy saving prediction

**Example:**

```python
class OptimizationStrategy(ABC):
    """Abstract base for rApp optimization algorithms."""

    @abstractmethod
    def evaluate(self, kpis: KpiReport) -> PolicyDecision: ...


class ThresholdBasedStrategy(OptimizationStrategy):
    def evaluate(self, kpis: KpiReport) -> PolicyDecision:
        if kpis.prb_util_dl < self.threshold_low:
            return PolicyDecision.SLEEP
        return PolicyDecision.ACTIVE


class MlBasedStrategy(OptimizationStrategy):
    def evaluate(self, kpis: KpiReport) -> PolicyDecision:
        return self._model.predict(kpis)
```

---

## 4. Code Documentation (Sphinx / Doxygen)

Every class and function must have a docstring that can be auto-generated by Sphinx (Python) or Doxygen (C/C++).

**Python — Sphinx-compatible format:**

```python
def monitor_traffic_load(self, cell_id: str, interval_ms: int = 100) -> TrafficStatus:
    """Monitor real-time traffic load for a given cell via E2 KPM.

    :param cell_id: The cell identifier (NR Cell Global ID).
    :param interval_ms: KPM report interval in milliseconds.
    :return: TrafficStatus containing PRB utilization and active UE count.
    :raises E2ConnectionError: If the E2 subscription fails.
    """
```

**C++ — Doxygen-compatible format:**

```cpp
/**
 * @brief Send RIC Control Request to activate a cell.
 *
 * @param cell_id  Target cell identifier.
 * @param timeout  Maximum wait time in milliseconds.
 * @return true if the cell was activated successfully, false otherwise.
 */
bool activateCell(const std::string& cell_id, int timeout = 5000);
```

> [!NOTE]
> Run Sphinx/Doxygen as part of the project documentation. Generated API docs should be linked from the project's installation guide.

---

## 5. Folder Structure

Use this structure for all rApp/xApp projects:

```
project-name/
├── src/
│   ├── core/               # Core rApp/xApp logic (platform-independent)
│   │   ├── strategies/     # Strategy pattern — optimization algorithms
│   │   └── models/         # Data models and 3GPP parameter classes
│   ├── adapters/           # Adapter pattern — vendor-specific parameter converters
│   ├── factories/          # Abstract Factory — platform-specific module creators
│   │   ├── ns3/
│   │   ├── viavi/
│   │   └── physical/
│   └── main.py             # Entry point
├── docs/
│   ├── drawio/             # Source .drawio files (system architecture, MSC, etc.)
│   └── api/                # Generated Sphinx/Doxygen output
├── tests/                  # Unit and integration tests
├── simulation/             # Simulation scripts, raw data, and result notebooks
├── config/                 # Configuration files and environment templates
│   └── .env.example
├── requirements.txt        # Python dependencies (or CMakeLists.txt for C++)
└── README.md               # Project Documentation (follows project-documentation.md)
```

> [!WARNING]
> The `docs/drawio/` folder must be kept in the repository. Export diagrams as PNG/SVG for embedding in documentation, but always commit the source `.drawio` files for future updates.

---

## 6. General Architecture

A typical modular rApp architecture applying all three patterns:

```
rApp Core Logic (Strategy)
        |
        v
PlatformFactory (Abstract Factory)
        |
    +---+---+
    |       |
Platform A  Platform B
    |       |
    v       v
ScenarioRunner
TelemetryCollector  →  Adapter  →  3GPP KPIs
KpiAnalyzer
```

- **Adapter** — normalizes proprietary inputs into the common 3GPP model used by core logic
- **Abstract Factory** — creates the correct platform modules (ns-3, VIAVI, physical testbed) without touching core logic
- **Strategy** — selects which optimization or evaluation algorithm the core logic executes

> [!TIP]
> Design the core logic first (Strategy + data models), then build the factory and adapters around it. This keeps the thesis contribution (the algorithm) cleanly separated from integration plumbing.

---

## 7. Production Readiness

Before handover, all components must pass the following production-readiness checks. Code that fails any of these checks is **not ready for handover** even if all other SOP criteria are met.

> [!WARNING]
> These checks exist because they caused real handover failures in previous projects (expired registry tokens, empty submodules, undocumented environment variables). Each check must be verified by the outgoing student — not assumed to work.

### 7.1 Container Images

Every container image referenced in `README.md`, Helm values files, or Dockerfiles must be pullable by a new user.

**Required:**
- Document the full registry URL, required credentials, and how to renew an expired pull token.
- If an image build requires local compilation, document the complete build command:

  ```bash
  docker build -t registry.example.com/project/component:latest \
    --build-arg BASE_IMAGE=... \
    -f src/component/Dockerfile .
  ```

- Test the pull from a fresh machine (not one with a cached image) and confirm it succeeds.
- If the registry pull token expires periodically, document the renewal process (e.g., `docker login`, Helm secret rotation).

### 7.2 Git Submodule Initialization

If the repo uses git submodules (`.gitmodules` present), the README **Quick Start** section must include:

```bash
# Option A — clone with submodules in one step
git clone --recursive <repo-url>

# Option B — after a plain clone
git submodule update --init --recursive
```

> [!CAUTION]
> Always test a plain `git clone <repo-url>` (without `--recursive`) and document exactly which directories will be empty. A successor who does a plain clone and silently loses a critical subsystem (e.g., a sideloader service) cannot reproduce the project.

### 7.3 Known Issues & Workarounds

Every repo must include a **Known Issues** section in `README.md` or a dedicated `docs/KNOWN_ISSUES.md` file. This section is **required** — not optional.

**Minimum required contents:**
- Failure modes a new user will encounter on their first deployment
- Expired credentials or tokens that need renewal (with steps to renew)
- Hardware quirks, timing-sensitive setup steps, or platform-specific constraints
- Unresolved integration issues — state whether they are "out of scope", "future work", or need a workaround
- Any data anomalies in experimental results (failed configs, sensor glitches, outliers)

**Example template:**

```markdown
## Known Issues

| Issue | Severity | Status | Workaround |
|-------|----------|--------|------------|
| NFO container image pull 401 UNAUTHORIZED | ❌ BLOCK | Token expired | Rebuild from source: `docker build ...` |
| `src/sideloader/` empty on plain clone | ❌ BLOCK | Submodule | Run `git submodule update --init` |
| TEIV adapter: "FOCOM JSON is empty" | ⚠️ WARN | Open | Restart FOCOM pod first, then TEIV |
| Pegatron/OKD/8c/700Mbps always FAILED | ℹ️ INFO | Hardware limit | Exclude from benchmarks |
```

### 7.4 API Endpoint Naming Consistency

Every API endpoint must be named **identically** in:
1. Thesis diagrams (MSC, flowcharts)
2. `README.md` and User Guide descriptions
3. Actual source code

If an endpoint was renamed during development, add a mapping note in the User Guide:

```markdown
> **Note:** Thesis Fig 3.2 shows `POST /vnf/create`. The implementation uses `POST /nfo/deploy`
> (renamed in v1.2 to clarify the caller is the NFO, not a generic VNF interface).
```

### 7.5 Secrets & Environment Variables

- Document every required environment variable in `config/.env.example`. Never commit secrets.
- Mark any credential that expires with a renewal deadline note and the steps to renew.
- List all required service accounts, API keys, or certificates and where to obtain them.

---

## 8. Standards & Parameter Reference Convention

All parameter names used in source code, documentation, and diagrams **must be linked to their authoritative specification** with an exact section and page reference.

> [!IMPORTANT]
> This requirement applies to **every** parameter name in: class attributes, function parameters, docstrings, System Parameters tables, README sections, MSC message labels, and flowchart condition labels.
>
> An unlinked parameter name is an unverifiable claim. Future students and LLMs cannot confirm the correctness of the parameter semantics without the spec reference.

### 8.1 Rule: Link Every Parameter Name

When you write a parameter name (e.g., `DRB.PrbUtilDL`, `cellActivationCmd`, `prbThresholdLow`):

| Context | Required format |
|---------|----------------|
| Documentation (README, guides, diagrams) | `[DRB.PrbUtilDL](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip) (TS 28.552 §5.1.1.12.1)` |
| Code docstrings | `:param drb_prb_util_dl: DL PRB utilization (TS 28.552 §5.1.1.12.1, Table 5.1.1.12.1-1, p.47)` |
| System Parameters table | Include **Page / §** column — see §8.3 |

### 8.2 3GPP & O-RAN Specification Links

**3GPP archive root:** `https://www.3gpp.org/ftp/Specs/archive/`

| Series | Archive path | Example |
|--------|-------------|---------|
| TS 28.xxx (OAM / PM) | `.../28_series/28.{spec}/` | [TS 28.552](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/) |
| TS 38.xxx (NR radio) | `.../38_series/38.{spec}/` | [TS 38.331](https://www.3gpp.org/ftp/Specs/archive/38_series/38.331/) |
| TS 36.xxx (LTE) | `.../36_series/36.{spec}/` | [TS 36.214](https://www.3gpp.org/ftp/Specs/archive/36_series/36.214/) |
| TS 38.300 (NR overall) | `.../38_series/38.300/` | [TS 38.300](https://www.3gpp.org/ftp/Specs/archive/38_series/38.300/) |

Link directly to the ZIP of the version you used. Example:

```
[DRB.PrbUtilDL](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip)
→ TS 28.552 §5.1.1.12.1, Table 5.1.1.12.1-1 (v18.5.0, p.47)
```

**O-RAN Alliance specifications:** [https://specifications.o-ran.org/](https://specifications.o-ran.org/)

**IEEE standards:** [https://ieeexplore.ieee.org/](https://ieeexplore.ieee.org/)

### 8.3 System Parameters Table — Required Columns

The System Parameters table in your project documentation **must** include a **Page / §** column:

| Category | Parameter | Type | Unit | Standard | Spec Section | Page / § | Description |
|----------|-----------|------|------|----------|-------------|----------|-------------|
| E2 KPM Input | [`DRB.PrbUtilDL`](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip) | Float | % | TS 28.552 | §5.1.1.12.1 | Table 5.1.1.12.1-1, p.47 | Downlink PRB utilization |
| E2 KPM Input | [`RRC.ConnectedUE`](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip) | Integer | count | TS 28.552 | §5.1.1.1.1 | Table 5.1.1.1.1-1, p.21 | Number of RRC-connected UEs |
| E2 RC Output | [`cellActivationCmd`](https://www.3gpp.org/ftp/Specs/archive/38_series/38.331/38331-i40.zip) | Enum | — | TS 38.331 | §6.2.2 | §6.2.2, p.184 | Cell activation command |

The **Page / §** column must reference the specific table, paragraph, or page number in the specification ZIP so that future students and LLMs can verify the parameter definition without searching the entire document.

> [!TIP]
> Open the spec PDF, find your parameter's defining table, and note the section heading and page number. Include the version (e.g., `v18.5.0`) so the reference stays unambiguous as specs are updated.
