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
  - [8. Graduation Procedure (NTUST)](#8-graduation-procedure-ntust)

## 1. General Principle

> [!IMPORTANT]
> No student may formally leave the lab until all agreed deliverables have been completed and verified. The Professor's signature on any departure or transfer documentation is contingent on the full completion of all committed work.

The lab operates as a professional R&D environment. All platforms, research systems, and documentation developed within the lab must remain **reproducible by any incoming member at any time**. The purpose of this procedure is to ensure the continuity of all ongoing research platforms, regardless of who built them.

**Knowledge-transfer roles** — three steps, in order (aligned with the [Open Research Playbook](https://github.com/raycg/Open-Research-Playbook) prepare → verify → accept workflow):

1. **Graduating student** prepares the transfer package — Section 5, groups A–D.
2. **Incoming student** independently verifies it by reproducing the **Minimum Reproducible Result (MRR)** — the experimental results presented in the final defense slides (items 5.15–5.16).
3. **Professor** reviews the evidence and decides: **approved** / **conditionally approved** (corrections listed, departure only after they are confirmed complete) / **not approved**.

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
> Item 4.1.2 applies only to students who have **not** received Prof's approval for *Chapter IV. Results and Analysis*.

> [!NOTE]
> Backticked placeholders (e.g. `` `<pCloud link>` ``) are template fields — replace them with real links in your [student card](./templates/student-card.md); never leave a fake markdown link.

- [ ] **4.0** Recognition Form for Academic Field of Expertise — `<form link>` *(Chinese & English Abstract)*
- [ ] **4.1** Code & Project Review *(conducted during weekly meeting)*
  - [ ] **4.1.1** (**DL: D-180 before oral exam**) O1 / E2 / R1 interface test conformance — all checks in [oran-verification.md](./oran-verification.md) pass
  - [ ] **4.1.2** (**DL: D-60 before oral exam**) Early review — *required only if Chapter IV is not yet approved*
  - [ ] **4.1.3** (**DL: D-30 before oral exam**) Final code and project review
- [ ] **4.2** Simulation results complete — following [simulation.md](./simulation.md)
  - [ ] **4.2.1** RAW dataset uploaded to pCloud — `<pCloud link>`
  - [ ] **4.2.2** Experiment scripts & results — `<GitHub link>`; every thesis/paper figure regenerable from the raw dataset with one command
- [ ] **4.3** Thesis Book — `<Overleaf link>`; full LaTeX on **Prof's Overleaf account**, ready for submission
- [ ] **4.4** IEEE Paper — `<Overleaf link>`; full LaTeX on **Prof's Overleaf account**
- [ ] **4.5** Presentation Slide — `<slides link>`; final oral exam slide, complete and approved by Prof
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

- [ ] **5.1** Repository exists under the `bmw-ece-ntust` organization — `<GitHub repo link>`
- [ ] **5.2** Source code aligns with all feature claims in the paper and `research.md`
- [ ] **5.3** Code structure follows [programming.md](./programming.md) standards *(OOP, design patterns, Sphinx/Doxygen docstrings)*

**B. Documentation** *(Must be 100% complete and internally consistent)*

- [ ] **5.4** `research.md` — all diagrams complete and aligned with the source code:
  - [ ] System Architecture diagram
  - [ ] Use Case diagram
  - [ ] Message Sequence Charts (MSC)
  - [ ] Flowcharts
  - [ ] Class Diagram
- [ ] **5.5** `implementation.md` — per-component installation steps + end-to-end system setup; each step shows expected input/output; ends with the [O-RAN verification checks](./oran-verification.md); includes installation video (`<video link>`) and integration video (`<video link>`)
- [ ] **5.6** User Guide — section 3 of the project `README.md` ([readme-guide.md](./readme-guide.md)): operating instructions with example I/O; includes demo video (`<pCloud/YouTube link>`)

**C. Simulation** *(Must be reproducible end-to-end by the incoming student)*

- [ ] **5.7** Simulation Guide (`simulation/README.md`) — following [simulation.md](./simulation.md): scenarios, immutable raw files, dataset registry
- [ ] **5.8** Simulation Video — uploaded to the BMW Lab YouTube channel — `<YouTube link>`
- [ ] **5.9** InfluxDB pipeline working — incoming student can store and process data with the rApp
- [ ] **5.10** Grafana dashboard working — incoming student can visualize data from InfluxDB
- [ ] **5.11** Raw dataset in pCloud, verified by the incoming student — `<pCloud link>`

**D. Continuity & Access** *(the incoming student can work from day one)*

- [ ] **5.12** Four project memory files reconciled (`AGENTS.md`/`CLAUDE.md`, `CONTEXT.md`, `MEMORY.md`, `TODO.md`) and the graphify knowledge graph built (`graphify-out/`) — the incoming student's LLM is productive immediately
- [ ] **5.13** Credentials transferred via Vault ([credentials-vault.md](./lab-automation/credentials-vault.md)): all project secrets under `secret/lab/<project>/…`; incoming student's policy granted; departing student's tokens revoked and shared secrets rotated; registry pull tokens and renewal steps documented in Known Issues
- [ ] **5.14** Hardware & accounts inventory handed over — one table: item, location, owner, access method (testbed machines, SDRs/USRPs, SIM cards, UE modems, VM allocations, server accounts)
- [ ] **5.15** **MRR reproduced** per [simulation.md Section 6](./simulation.md#6-verification-at-handover): the incoming student independently regenerates the Minimum Reproducible Result — every experimental result in the final defense slides — from the raw dataset on a clean machine

**E. Handover Review** *(deadline: 15th Aug)*

- [ ] **5.16** (**DL: 15th Aug**) Handover review by Ph.D. students [Ian/Bimo] — *presented by the incoming student*; outcome reported to the Professor as **approved / conditionally approved (corrections + due dates) / not approved**

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

## 8. Graduation Procedure (NTUST)

> [!WARNING]
> Always check the current procedure on the [ECE NTUST website](https://www.academic.ntust.edu.tw/p/412-1048-8756.php?Lang=en) and follow the [NTUST academic calendar](https://www.academic.ntust.edu.tw/p/412-1048-8756.php?Lang=en) — these administrative steps change between semesters.

**Qualifications.** Master's students who meet graduation regulations may apply for the oral defense. Application deadlines: **Spring semester — 31st January**; **Fall semester — 31st July**.

**Oral-defense application:**

- [ ] 8.1 Complete "[The oral defense application of Master degree](https://ece.ntust.edu.tw/var/file/17/1017/img/Master_s_Academic_Degree_Examination_Orals_Recommend_Application_Form_1131008.docx)" (download from "[Forms and Links](https://ece.ntust.edu.tw/p/412-1017-1400.php?Lang=en)")
- [ ] 8.2 Download the required documents at the Student Information System ([link](https://www.academic.ntust.edu.tw/p/412-1048-8234.php?Lang=en))
- [ ] 8.3 Upload the digital thesis at the [Taiwan Tech Electronic Thesis System](https://etheses.lib.ntust.edu.tw/cgi-bin/gs32/gsweb.cgi/ccd=PrUzwJ/webmge?switchlang=en)
- [ ] 8.4 Print the "e-Thesis Authorization Form" after uploading your thesis, then proceed with the school leaving steps below

**Before the oral defense** *(complete at least 2 weeks before):*

- [ ] 8.5 Prepare one copy of "Qualification Form by Master Degree Examination Committee"
- [ ] 8.6 Print copies of "Oral Defense Evaluation Form" for all committee members
- [ ] 8.7 Bring the "Receipt of Payment" (provided before the oral defense)

**After the oral defense:**

- [ ] 8.8 Submit the "Qualification Form", "Evaluation Form", "Thesis Academic Ethics and Authentication of Originality Statement", and "Receipt of Payment" to the ECE office
- [ ] 8.9 Attach the "Recommendation Form" and "Qualification Form" with the thesis
- [ ] 8.10 Send the PDF file of **Moodle Turnitin** result by e-mail

> *To apply for delayed public access, download the "Application for Embargo of Thesis/Dissertation" and attach it to the first page of the thesis.*

**School leaving** *(deadline: the date before the next-semester registration deadline — follow the school calendar):*

- [ ] 8.11 Upload the digital thesis and submit one printed copy to the library
- [ ] 8.12 Download and complete the [School Leaving Application Form](https://www.academic.ntust.edu.tw/p/412-1048-8234.php?Lang=en) via the Student Information System
- [ ] 8.13 Submit a soft, yellow-cover copy of the thesis to the department office
- [ ] 8.14 Submit a printed copy of the thesis and the signed e-Thesis Authorization Form (NTUST & NCL) to the library
- [ ] 8.15 Complete all steps in the School Leaving Application Form and collect your diploma at the Office of Graduate Studies
