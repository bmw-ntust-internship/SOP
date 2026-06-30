<h1 align="center">Source Code Standards — Guideline</h1>

---

> [!WARNING]
> Follow this guideline for **all** thesis project source code in the BMW Lab.
> Code that does not meet these standards will not pass the handover review.

> [!NOTE]
> **Relationship to CONTEXT.md:**
> The class diagram, flowcharts, and system parameters in `CONTEXT.md` are the
> design contract.  Source code must implement exactly what those diagrams describe
> — names, interfaces, and responsibilities must match.

---

## Table of Contents

- [1. Design-First Approach](#1-design-first-approach)
- [2. Object-Oriented Programming (OOP)](#2-object-oriented-programming-oop)
- [3. Design Patterns](#3-design-patterns)
- [4. Code Documentation (Sphinx / Doxygen)](#4-code-documentation-sphinx--doxygen)
- [5. Folder Structure](#5-folder-structure)
- [6. General Architecture](#6-general-architecture)
- [7. O-RAN Protocol Rules](#7-o-ran-protocol-rules)
- [8. 3GPP Parameter Enumeration](#8-3gpp-parameter-enumeration)
- [9. Multi-vendor Support](#9-multi-vendor-support)
- [10. Intent-Based Networking (IBN)](#10-intent-based-networking-ibn)
- [11. Production Readiness](#11-production-readiness)
- [12. Standards and Parameter Reference Convention](#12-standards-and-parameter-reference-convention)

---

## 1. Design-First Approach

Before writing any code, produce the following artifacts **in order** in `CONTEXT.md`:

| Order | Artifact | Purpose |
| --- | --- | --- |
| 1 | **Flowchart** | Define program logic: initialization → processing → output |
| 2 | **Class Diagram** | Define classes, attributes, methods, relationships |
| 3 | **System Parameters Table** | Define all inputs/outputs with 3GPP/IEEE spec references |

> [!CAUTION]
> Do not start implementation until the flowchart and class diagram are reviewed
> and approved in a weekly meeting.

---

## 2. Object-Oriented Programming (OOP)

Reference: [Python OOP Guide](https://realpython.com/python3-object-oriented-programming/)

| Principle | Application |
| --- | --- |
| **Encapsulation** | Group related data and methods; use private (`_`) / protected access |
| **Abstraction** | Use ABCs or `Protocol` for components with multiple implementations |
| **Inheritance** | Class hierarchies (e.g. `BaseRApp` → `EnergySavingRApp`) |
| **Polymorphism** | Overridable methods for vendor-specific or platform-specific behavior |

---

## 3. Design Patterns

Three patterns are required for all rApp / xApp projects.

Reference: [Refactoring.Guru](https://refactoring.guru/design-patterns/)

### 3.1 Adapter Pattern

**Purpose:** Convert proprietary vendor parameters or simulator outputs into
3GPP-compliant KPIs so rApp core logic never sees vendor-specific names.

Reference: [Adapter Pattern](https://refactoring.guru/design-patterns/adapter)

**When to use:**
- Converting proprietary gNB / WiFi AP metrics → `ThreeGPPKpi` identifiers
- Converting O-RAN YANG keys → BMW Lab internal types (`KpiReport`, `PolicyDecision`)

**Example (vendor adapter):**

```python
class EricssonClient(VendorTelemetryClient):
    PARAM_MAP: ClassVar[dict[str, ThreeGPPKpi]] = {
        "dl_prb_usage_pct": ThreeGPPKpi.DRB_PRB_UTIL_DL,
        "ul_prb_usage_pct": ThreeGPPKpi.DRB_PRB_UTIL_UL,
        "active_ue_count":  ThreeGPPKpi.RRC_CONN_MEAN,
    }
```

### 3.2 Abstract Factory Pattern

**Purpose:** Create platform-specific modules for different O-RAN deployment
environments without changing core rApp logic.

Reference: [Abstract Factory Pattern](https://refactoring.guru/design-patterns/abstract-factory)

**Deployment environments (select via `RAPP_PLATFORM`):**

| `RAPP_PLATFORM` | Factory | Use when |
| --- | --- | --- |
| `mock` | `MockPlatformFactory` | Unit tests, demos, developer laptop |
| `osc` | `OscPlatformFactory` | Production OSC or simulation via BMW Lab TA rApp |
| `physical` | `PhysicalPlatformFactory` | Real gNB testbed |

> [!IMPORTANT]
> **Simulation rule:** Simulator lifecycle (start/stop VIAVI RSG or ns-3 scenarios,
> configure UE mobility) is the BMW Lab TA rApp's responsibility.
> The generic rApp / xApp uses `RAPP_PLATFORM=osc` pointed at the simulator's
> O-RAN interfaces — it never calls simulator APIs directly.
> TA rApp: <https://github.com/bmw-ece-ntust/nonrtric-rapp-test-automation>

**Example:**

```python
class RAppPlatformFactory(ABC):
    @abstractmethod
    def create_scenario_runner(self) -> ScenarioRunner: ...
    @abstractmethod
    def create_telemetry_collector(self) -> TelemetryCollector: ...
    @abstractmethod
    def create_kpi_analyzer(self) -> KpiAnalyzer: ...
```

### 3.3 Strategy Pattern

**Purpose:** Swap optimization or inference algorithms without modifying the
rApp framework.

Reference: [Strategy Pattern](https://refactoring.guru/design-patterns/strategy)

**When to use:**
- Swapping ML models, threshold rules, or NVIDIA NIM inference backends
- Comparing energy-saving algorithms

**Example:**

```python
class OptimizationStrategy(ABC):
    @abstractmethod
    def evaluate(self, kpis: KpiReport) -> PolicyDecision: ...

class ThresholdBasedStrategy(OptimizationStrategy):
    def evaluate(self, kpis: KpiReport) -> PolicyDecision:
        return PolicyDecision.SLEEP if kpis.prb_util_dl < self.threshold_low else PolicyDecision.ACTIVE

class NvidiaModelStrategy(OptimizationStrategy):
    def evaluate(self, kpis: KpiReport) -> PolicyDecision:
        return PolicyDecision(self._nim_infer(kpis))
```

---

## 4. Code Documentation (Sphinx / Doxygen)

Every class and function must have a docstring auto-generatable by Sphinx (Python)
or Doxygen (C/C++).

**Python — Sphinx/RST format:**

```python
def monitor_traffic_load(self, cell_id: str, interval_ms: int = 100) -> KpiReport:
    """Monitor real-time traffic load via E2 KPM.

    :param cell_id: NR Cell Global ID.
    :param interval_ms: KPM report interval in milliseconds (TS 28.552 §5.1.1.12.1).
    :return: :class:`~core.models.KpiReport` with current PM counters.
    :raises E2Error: If the E2 subscription fails.
    """
```

**C++ — Doxygen format:**

```cpp
/**
 * @brief Send RIC Control Request to activate a cell.
 * @param cell_id Target NR Cell Global ID.
 * @param timeout Maximum wait in milliseconds.
 * @return true if activated successfully.
 */
bool activateCell(const std::string& cell_id, int timeout = 5000);
```

---

## 5. Folder Structure

```text
project-name/
├── src/
│   ├── core/
│   │   ├── models/
│   │   │   ├── __init__.py     KpiReport, PolicyDecision
│   │   │   └── parameters.py   ThreeGPPKpi enum, NodeType enum, VendorParameterMap
│   │   └── strategies/         OptimizationStrategy ABC + implementations
│   ├── factories/              Abstract Factory — deployment environment creators
│   │   ├── mock/
│   │   ├── osc/
│   │   └── physical/
│   └── rapp/
│       ├── adapters/           Adapter pattern — O-RAN + vendor translators
│       │   ├── a1/
│       │   ├── e2/
│       │   ├── o1/
│       │   ├── r1/
│       │   ├── teiv/
│       │   ├── ves/
│       │   └── vendor/
│       ├── application/
│       │   ├── ports.py        Port Protocols (structural typing)
│       │   └── usecases.py
│       ├── config/settings.py  RAPP_* env vars
│       └── domain/
│           ├── intent.py       IBN IntentContract + IntentResolutionService
│           ├── models.py
│           ├── services.py
│           └── topology.py     NetworkTopology + NodeInfo
├── docs/
│   ├── drawio/                 Source .drawio files (version controlled)
│   └── api/                    Generated Sphinx / Doxygen output
├── tests/
├── simulation/
├── config/
│   └── .env.example
├── requirements.txt
├── CONTEXT.md                  Full PRD (design contract, paper writing)
└── README.md                   User guide (quick start, config, endpoints)
```

> [!WARNING]
> Store `.drawio` source files in `docs/drawio/`.  Export PNG/SVG for embedding
> in documentation; always commit the source file for future updates.

---

## 6. General Architecture

```text
Use-case (application layer)
        │
        ▼
Port Protocol (structural typing)
        │
        ▼
Adapter (O-RAN interface OR vendor normalization)
        │
        ▼
ThreeGPPKpi enum ──────── KpiReport ──────── OptimizationStrategy
                                                      │
                                                      ▼
                                               PolicyDecision
                                                      │
                                                      ▼
                                             A1Port / E2RcPort
```

> [!TIP]
> Design core logic first (Strategy + KpiReport), then build adapters around it.
> The research contribution (the algorithm) stays cleanly separated from
> integration plumbing.

---

## 7. O-RAN Protocol Rules

> [!IMPORTANT]
> These rules are mandatory for all rApp / xApp code in BMW Lab.

1. **O-RAN protocols only.** rApps and xApps communicate exclusively through
   O-RAN ALLIANCE standard interfaces: O1, A1, R1 (SME/ICS), and E2.
   No direct calls to simulator APIs (VIAVI RSG, ns-3, UERANSIM).

2. **No InfluxDB bypass.** Telemetry arrives via ICS subscription.
   InfluxDB is an internal SMO storage layer — the rApp never queries it directly.

3. **VES events via ICS.** O1 VES events (PM reports, fault notifications)
   are received through the ICS push callback, not by polling the VES Collector.

4. **Factory = deployment environment.** `RAPP_PLATFORM=mock` for tests,
   `osc` for simulation (via TA rApp) and production.

5. **Simulation via TA rApp.** All simulator lifecycle (start/stop scenarios,
   UE mobility, gNB config) is the BMW Lab TA rApp's responsibility.
   The rApp uses `osc` platform pointed at the simulator's O-RAN interfaces.

---

## 8. 3GPP Parameter Enumeration

Use the `ThreeGPPKpi` string enum for all 3GPP counter name references.
Never use raw string literals in business logic.

```python
# Wrong
raw.get("DRB.PrbUtilDL", 0.0)

# Correct
from core.models.parameters import ThreeGPPKpi
raw.get(ThreeGPPKpi.DRB_PRB_UTIL_DL.value, 0.0)
```

`ThreeGPPKpi` members with spec references:

| Enum member | Value | Spec | Section | Page |
| --- | --- | --- | --- | --- |
| `DRB_PRB_UTIL_DL` | `"DRB.PrbUtilDL"` | TS 28.552 | §5.1.1.12.1 | Table 5.1.1.12.1-1, p.47 |
| `DRB_PRB_UTIL_UL` | `"DRB.PrbUtilUL"` | TS 28.552 | §5.1.1.12.2 | Table 5.1.1.12.2-1, p.48 |
| `DRB_UE_THP_DL` | `"DRB.UEThpDL"` | TS 28.552 | §5.1.1.10.1 | Table 5.1.1.10.1-1, p.44 |
| `DRB_UE_THP_UL` | `"DRB.UEThpUL"` | TS 28.552 | §5.1.1.10.2 | Table 5.1.1.10.2-1, p.45 |
| `RRC_CONN_MEAN` | `"RRC.ConnMean"` | TS 28.552 | §5.1.1.1.1 | Table 5.1.1.1.1-1, p.21 |
| `RSRP` | `"RSRP"` | TS 36.214 | §5.1.1 | §5.1.1, p.9 |
| `RSRQ` | `"RSRQ"` | TS 36.214 | §5.1.2 | §5.1.2, p.10 |
| `SINR` | `"SINR"` | TS 36.214 | §5.1.4 | §5.1.4, p.12 |

---

## 9. Multi-vendor Support

Multi-vendor support is implemented through two mechanisms:

**A. Vendor metric normalization (vendor adapter):**
Subclass `VendorTelemetryClient` and declare `PARAM_MAP` mapping proprietary
keys to `ThreeGPPKpi`.  The adapter calls `VendorParameterMap.raw_to_standard()`
to translate.

**B. YANG key override (O1 adapter):**
Subclass `O1SdncAdapter` and override the `_YANG_MAX_UES` class attribute (or add
new vendor-specific YANG keys) without forking the adapter.

**C. WiFi APs:**
Use `NodeType.WIFI_AP` in `NodeInfo`.  Map IEEE 802.11 metrics (RSSI, channel
utilization, association count) to the closest `ThreeGPPKpi` equivalents in
the vendor adapter.  See BMW Lab Aruba project for reference implementation.

**D. Multi-gNB:**
Use `NetworkTopology` to register all cells at startup (via `TEIVAdapter`).
Use-cases iterate `topo.active_cells()` — never hard-code cell IDs.

---

## 10. Intent-Based Networking (IBN)

> [!NOTE]
> IBN support is planned via NVIDIA NeMo / NIM.  Use the stubs below as the
> extension point.

**Contract-based protocol:** Every intent is a cryptographically signed
`IntentContract` (HMAC-SHA256, IETF RFC 9315).  The `IntentResolutionService`
enforces three security checks before resolving:

1. **Whitelist** — only `IntentType` enum values are accepted.
2. **Expiry** — contracts past `expires_at` are rejected.
3. **Signature** — HMAC-SHA256 verified against `RAPP_INTENT_SECRET`.

**Integration pattern:**

```python
# In the ICS callback handler
contract = IntentContract(**payload)
svc = IntentResolutionService(secret_key=settings.intent_secret)
svc.validate(contract)           # raises IntentValidationError if invalid
decision = svc.resolve(contract, kpis)   # implement per use case
```

**NVIDIA NIM integration:**
Place NIM HTTP logic in `src/rapp/adapters/nim/`.
Pass `NimAdapter.infer` to `NvidiaModelStrategy`:

```python
nim = NimAdapter(base_url=..., model_id=..., api_key=os.environ["NVIDIA_API_KEY"])
strategy = NvidiaModelStrategy(nim_infer=nim.infer)
```

---

## 11. Production Readiness

Before handover, all of the following must pass.

### 11.1 Container Images

Document the full registry URL, required credentials, and how to renew an
expired pull token.  Test the pull from a fresh machine.

```bash
docker build -t registry.example.com/project/rapp:latest -f src/Dockerfile .
docker pull registry.example.com/project/rapp:latest   # verify from clean machine
```

### 11.2 Git Submodules

If `.gitmodules` is present, the `README.md` Quick Start section must include:

```bash
git clone --recursive <repo-url>
# or after plain clone:
git submodule update --init --recursive
```

### 11.3 Known Issues

Every repo must include a **Known Issues** table in `README.md`:

| Issue | Severity | Status | Workaround |
| --- | --- | --- | --- |
| \<describe issue\> | ❌ BLOCK / ⚠️ WARN / ℹ️ INFO | Open / Fixed | \<steps\> |

### 11.4 API Endpoint Consistency

Every API endpoint name must be identical in `CONTEXT.md` diagrams, `README.md`,
and source code.  If renamed during development, add a mapping note.

### 11.5 Secrets and Environment Variables

Document every env var in `config/.env.example`.  Never commit secrets.
Mark expiring credentials with renewal instructions.

---

## 12. Standards and Parameter Reference Convention

> [!IMPORTANT]
> Every parameter name in source code, documentation, and diagrams must be
> linked to its authoritative spec with exact section and page number.

### 12.1 Rule

| Context | Required format |
| --- | --- |
| Documentation (CONTEXT.md, guides) | `[DRB.PrbUtilDL](https://…/28552-i50.zip) (TS 28.552 §5.1.1.12.1)` |
| Code docstrings | `:param drb_prb_util_dl: DL PRB utilization (TS 28.552 §5.1.1.12.1, p.47)` |
| System Parameters table | Include **Page/§** column |

### 12.2 Spec Archive URLs

| Series | Archive |
| --- | --- |
| TS 28.xxx (OAM/PM) | <https://www.3gpp.org/ftp/Specs/archive/28_series/> |
| TS 38.xxx (NR radio) | <https://www.3gpp.org/ftp/Specs/archive/38_series/> |
| TS 36.xxx (LTE) | <https://www.3gpp.org/ftp/Specs/archive/36_series/> |
| O-RAN Alliance | <https://specifications.o-ran.org/> |
| IEEE | <https://ieeexplore.ieee.org/> |

Link directly to the ZIP of the version used.  Example:

```text
[DRB.PrbUtilDL](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip)
→ TS 28.552 §5.1.1.12.1, Table 5.1.1.12.1-1 (v18.5.0, p.47)
```

### 12.3 System Parameters Table — Required Columns

| Category | Parameter | Type | Unit | Spec | Spec Section | Page/§ | Description |
| --- | --- | --- | --- | --- | --- | --- | --- |
| E2 KPM Input | [`DRB.PrbUtilDL`](https://www.3gpp.org/ftp/Specs/archive/28_series/28.552/28552-i50.zip) | float | ratio | TS 28.552 | §5.1.1.12.1 | Table 5.1.1.12.1-1, p.47 | DL PRB utilization |
| E2 RC Output | `PolicyDecision` | Enum | — | O-RAN.WG2.A1AP | §8.2 | §8.2, p.31 | ACTIVE / SLEEP / HANDOVER |

The **Page/§** column must cite the exact table, paragraph, or page number in
the spec ZIP so future students and LLMs can verify the parameter definition
without searching the entire document.
