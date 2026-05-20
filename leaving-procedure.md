# Laboratory Leaving Procedure

This document defines the complete leaving procedure for all members of the BMW Lab (Broadband Multimedia Wireless Networks Laboratory), NTUST. All departing members must complete the applicable checklists and sign the [Lab Non-Disclosure Agreement](./NDA/nda-template.md) before their departure is formally acknowledged.

This procedure applies to:

- Students completing their degree and **graduating**
- Students who wish to **change their supervising professor** and leave the lab mid-program

## Table of Contents

- [Laboratory Leaving Procedure](#laboratory-leaving-procedure)
  - [Table of Contents](#table-of-contents)
  - [1. General Principle](#1-general-principle)
  - [2. Written Record of Commitments](#2-written-record-of-commitments)
  - [3. Professor Change Policy](#3-professor-change-policy)
  - [4. Oral Examination Requirements](#4-oral-examination-requirements)
  - [5. Project Handover Requirements](#5-project-handover-requirements)
  - [6. Handover Candidates](#6-handover-candidates)
  - [7. NDA Requirement](#7-nda-requirement)

## 1. General Principle

> [!IMPORTANT]
> No student may formally leave the lab until all agreed deliverables have been completed and verified. The Professor's signature on any departure or transfer documentation is contingent on the full completion of all committed work.

The lab operates as a professional R&D environment. All platforms, research systems, and documentation developed within the lab must remain **reproducible by any incoming member at any time**. The purpose of this procedure is to ensure the continuity of all ongoing research platforms, regardless of who built them.

## 2. Written Record of Commitments

> [!IMPORTANT]
> All deliverables and tasks assigned to a student must be documented in the lab's GitHub system — GitHub Issues, Project Board, or milestone entries. This applies to all active students and must be done at the time of assignment, not at departure.

Verbal agreements are not sufficient. Any task agreed upon between the Professor and a student must be recorded in GitHub before work begins. This ensures:

- Transparency in workload expectations
- A verifiable record of what was agreed upon
- Prevention of disputes about scope or completion status at the time of departure

> [!WARNING]
> If a task is not documented in GitHub, it cannot be formally used as a basis for requiring or verifying completion at the time of departure.

## 3. Professor Change Policy

Students who decide to change their supervising professor mid-program must adhere to the following policy before any transfer can proceed.

> [!IMPORTANT]
> A student who wishes to change their supervising professor must first **complete all deliverables and tasks that were agreed upon** during their time in the lab. The Professor will only agree to sign the professor-exchange documentation after all promised work is verified as finished.

This policy exists to:

1. Protect the continuity of ongoing research platforms built with lab resources
2. Ensure that the investment of lab time and resources results in a complete and usable handover
3. Maintain fairness and consistency across all students

**Process for Professor Change:**

1. The student formally notifies the Professor of their intent to change supervisor in writing (email or GitHub Issue).
2. The Professor and the student jointly review all outstanding commitments documented in GitHub.
3. The student completes all outstanding tasks as agreed.
4. Ph.D. students [Ian/Bimo] verify that all deliverables are complete and reproducible.
5. Upon verified completion, the Professor signs the professor-exchange documentation.
6. The student signs the [Lab NDA](./NDA/nda-template.md) before departure.

> [!NOTE]
> This rule applies uniformly to all students. No exceptions are made based on individual circumstances. This ensures the policy remains fair and enforceable for all current and future lab members.

## 4. Oral Examination Requirements

These requirements apply to **graduating students**. Complete them in order — earlier items unblock later ones.

> [!WARNING]
> Item 4.1.1 applies only to students who have **not** received Prof's approval for *Chapter IV. Results and Analysis*.

- [ ] **4.0** [Recognition Form for Academic Field of Expertise](link) *(Chinese & English Abstract)*
- [ ] **4.1** Code & Project Review *(conducted during weekly meeting)*
  - [ ] **4.1.1** (**DL: D-60 before oral exam**) Early review — *required only if Chapter IV is not yet approved*
  - [ ] **4.1.2** (**DL: D-30 before oral exam**) Final code and project review
- [ ] **4.2** Simulation results complete — see [Section 4 of README](./README.md#4-simulation--data-pipeline)
  - [ ] **4.2.1** [RAW Dataset](pCloud link) — uploaded to pCloud
  - [ ] **4.2.2** [Experiment scripts & results](GitHub link) — includes all graphs and figures used in the thesis
- [ ] **4.3** [Thesis Book](Overleaf link) — full LaTeX on **Prof's Overleaf account**, ready for submission
- [ ] **4.4** [IEEE Paper](Overleaf link) — full LaTeX on **Prof's Overleaf account**
- [ ] **4.5** [Presentation Slide](Slides link) — final oral exam slide, complete and approved by Prof
- [ ] **4.6** Complete [pCloud Repository](http://u.pc.cd/Dbd) — using the [Template folder](http://u.pc.cd/yzQrtalK)

```bash
├── [Year-[Fall/Spring]]-[English Full Name]-[Topic]
│   ├── Demo video.mp4
│   ├── Dataset/               # all raw datasets for simulation/model/results
│   └── Slides/
│       ├── (Student ID) - (Name) - Oral Exam.pptx
│       ├── (Student ID) - (Name) - Rehearsal 1.pptx
│       ├── (Student ID) - (Name) - Rehearsal 2.pptx
│       ├── (Student ID) - (Name) - Rehearsal 3.pptx
│       └── (title) diagrams.drawio
```

## 5. Project Handover Requirements

These requirements apply to **all departing students**, whether graduating or changing supervisor. They ensure the incoming student — or any future lab member — can fully continue the project. All items are reviewed by Ph.D. students [Ian/Bimo] and verified together with the incoming student.

- [ ] **5.0** Handover to: `<incoming student name>`

**A. GitHub Repository** *(Complete this first — everything else depends on it)*

- [ ] **5.1** [Repository](GitHub repo link) exists under the `bmw-ece-ntust` organization
- [ ] **5.2** Source code aligns with all feature claims in the paper and project documentation
- [ ] **5.3** Code structure follows [source-code-guide.md](./source-code-guide.md) standards *(OOP, design patterns, Sphinx/Doxygen docstrings)*

**B. Documentation** *(Must be 100% complete and internally consistent)*

- [ ] **5.4** [Project Documentation](GitHub repo link) — all diagrams complete and aligned with the source code:
  - [ ] System Architecture diagram
  - [ ] Use Case diagram
  - [ ] Message Sequence Charts (MSC)
  - [ ] Flowcharts
  - [ ] Class Diagram
- [ ] **5.5** [Integration Guide](GitHub repo link) — per-component installation steps + end-to-end system setup; each step shows expected input/output; includes [installation video](video link) and [integration video](video link)
- [ ] **5.6** [User Guide](GitHub repo link) — operating instructions with example I/O; includes [demo video](pCloud/YouTube link)

**C. Simulation** *(Must be reproducible end-to-end by the incoming student)*

- [ ] **5.7** [Simulation Guide](GitHub repo link) — step-by-step instructions to reproduce the simulation and all raw data
- [ ] **5.8** [Simulation Video](YouTube link) — uploaded to BMW Lab YouTube channel
- [ ] **5.9** InfluxDB pipeline working — incoming student can store and process data with the rApp
- [ ] **5.10** Grafana dashboard working — incoming student can visualize data from InfluxDB
- [ ] **5.11** [Raw Dataset](pCloud link) — thesis/paper simulation data in pCloud for the incoming student to verify

**D. Handover Review** *(deadline: 15th Aug)*

- [ ] **5.12** (**DL: 15th Aug**) Handover review by Ph.D. students [Ian/Bimo] — *presented by the incoming student*

## 6. Handover Candidates

| Outgoing Student | Project Topic | Incoming Student |
| ---------------- | ------------- | ---------------- |
| *(fill in)*      | *(fill in)*   | *(fill in)*      |

## 7. NDA Requirement

All departing students, regardless of the reason for departure, must sign the [Lab Non-Disclosure Agreement](./NDA/nda-template.md) before their departure is formally completed.

> [!IMPORTANT]
> The NDA must be signed **before** the Professor signs any departure or transfer documentation.

The NDA covers:

- Confidentiality of unpublished research, data, and methodologies developed in the lab
- Prohibition on distributing internal lab information, decisions, or communications without the Professor's written consent
- Prohibition on misrepresenting the lab's policies, agreements, or internal arrangements to external parties

See [NDA/nda-template.md](./NDA/nda-template.md) for the full agreement text.
