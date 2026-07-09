<h1 align="center">Vibe-Coding — AI-Assisted Coding with Industrial-Grade Quality</h1>

---

> [!NOTE]
> **Who this is for.** Every student who lets an AI assistant (Claude Code, Copilot, Codex) write code — which in this lab is everyone. Read this once before your first AI-generated commit. It explains *why* the SOP asks for design patterns before you prompt, and gives you the working loop. The full source-code standard is [programming.md](./programming.md); this is the beginner's bridge to it.

## 1. What Vibe-Coding Is — and What Can Go Wrong

**Vibe-coding** is describing *what you want* in natural language and letting an LLM write the code. The lab encourages it: it is how we move fast. But an LLM optimizes for *plausible* code, not *maintainable* code. Left unguided, it happily produces a 900-line file with vendor names hard-coded into your algorithm and an `if platform == ...` ladder in every function — code that runs today and is unmaintainable the day you graduate.

The lab's rule is therefore simple:

> [!IMPORTANT]
> **The AI writes the code; you own the quality.** Every line you commit is *your* line — you must be able to explain it, and it must meet [programming.md](./programming.md) at handover review. "The AI wrote it" is never an accepted excuse.

## 2. Why Design Patterns Are the Answer

A design pattern is a proven, *named* solution to a recurring problem ([programming.md §3](./programming.md#3-design-patterns)). For vibe-coding, patterns do double duty:

| Without patterns | With patterns |
| --- | --- |
| You prompt "write an rApp that reads KPIs" — the LLM invents its own structure, differently every session | You prompt "implement `TelemetryCollector` behind an **Adapter**, selected by the **Abstract Factory**" — the LLM lands exactly in your architecture |
| Reviewers must read every line to understand the shape | "That's a Strategy" tells the reviewer the shape in one word |
| Regenerated code drifts from yesterday's code | The pattern is the stable contract; regeneration fills in the same slots |
| Your research (the algorithm) and the plumbing (O-RAN interfaces, vendors) tangle together | The algorithm stays swappable and comparable — which is your thesis |

In short: **patterns turn a vague vibe into a precise contract the LLM can hit.** That is what makes AI output industrial-grade instead of demo-grade.

The three patterns are mandatory for rApp / xApp projects (details and code in [programming.md §3](./programming.md#3-design-patterns)):

| Pattern | One-line why | What it protects you from |
| --- | --- | --- |
| **Adapter** | Translate vendor/simulator names into `ThreeGPPKpi` at the boundary | Vendor names leaking into your algorithm |
| **Abstract Factory** | One `RAPP_PLATFORM` switch builds the whole environment family (mock / osc / physical) | `if platform == ...` scattered everywhere; tests that need real hardware |
| **Strategy** | Each algorithm behind one `evaluate(kpis) → PolicyDecision` interface | An unfair thesis comparison and a framework full of branches |

## 3. The Lab Vibe-Coding Loop

Follow this loop for every feature. It is the SOP's design-first rule ([programming.md §1](./programming.md#1-design-first-approach)) adapted to AI-assisted work.

1. **Design before prompting.** Produce the flowchart, class diagram, state machine diagram, and System Parameters table in `research.md`, and get them approved in a weekly meeting. These diagrams *are your prompt contract* — the code must match them exactly.
2. **Brief the AI, don't just ask it.** Onboard the repo first (`/project-init` builds the graphify knowledge graph and the four project files — see [auto-lab.md](./auto-lab.md)). Then prompt with the design: name the classes, the pattern, and the spec references. Small, specific prompts beat "build my rApp".
3. **Generate small, review every diff.** Ask for one class or one method at a time. Read the diff before accepting. If you cannot explain a line, ask the AI to explain it — and if it still doesn't convince you, reject it.
4. **Verify mechanically.** Run the formatter/linter (`black`/`ruff`, `clang-format`), the unit tests on `RAPP_PLATFORM=mock`, and check the [O-RAN protocol rules](./programming.md#7-o-ran-protocol-rules). LLMs hallucinate APIs; the test run is where hallucinations die.
5. **Document and commit.** Docstrings in Sphinx/Doxygen format ([programming.md §4](./programming.md#4-code-documentation-sphinx--doxygen)), one logical change per commit, then `git push` — the daily-log writes itself ([auto-lab.md](./auto-lab.md)).

## 4. Non-Negotiable Rules for AI-Generated Code

- **Code matches the approved diagrams.** Class names, methods, and interfaces in the code must equal those in `research.md`. A change to one is a change to both.
- **No raw 3GPP strings.** Use the `ThreeGPPKpi` enum with spec references ([programming.md §8](./programming.md#8-3gpp-parameter-enumeration)); the LLM will happily typo `"DRB.PrbUtilDL"` — the enum catches it.
- **O-RAN interfaces only.** No direct simulator API calls, no InfluxDB bypass ([programming.md §7](./programming.md#7-o-ran-protocol-rules)). LLMs suggest the shortcut; the SOP forbids it.
- **No secrets in code or prompts.** Credentials come from the Vault credential layer ([lab-automation/credentials-vault.md](./lab-automation/credentials-vault.md)); env vars are documented in `config/.env.example`.
- **Never commit what you can't explain.** The oral exam and handover review are done by humans, to a human — you.

## 5. Failure Modes to Watch For

These are the classic unguided-LLM failures and the rule that prevents each:

| Symptom in generated code | Root cause | Prevention |
| --- | --- | --- |
| One giant file doing everything | No structure in the prompt | Design-first: prompt from the class diagram, [folder structure §5](./programming.md#5-folder-structure) |
| `dl_prb_usage_pct` inside your algorithm | Vendor name leaked past the boundary | Adapter + `ThreeGPPKpi` enum |
| `if platform == "osc": ... elif ...` | Environment branching | Abstract Factory + `RAPP_PLATFORM` |
| Algorithm hard-wired into the framework | No algorithm interface | Strategy pattern |
| Calls to an SDK method that doesn't exist | Hallucinated API | Run the tests; verify against pinned spec/doc links |
| Plausible-but-wrong parameter semantics | No spec grounding | System Parameters table with exact spec §/page ([programming.md §12](./programming.md#12-standards-and-parameter-reference-convention)) |

## 6. Where to Go Next

- [programming.md](./programming.md) — the full standard: OOP, the three patterns, docstrings, folder structure, O-RAN rules.
- [auto-lab.md](./auto-lab.md) — the lab automation that surrounds this loop: auto daily-log, long-term memory, credentials, and skills.
- [lab-automation/skills.md](./lab-automation/skills.md) — the catalog of lab skills your AI assistant already has.
