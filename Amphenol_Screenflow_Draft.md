# Amphenol — Persona Screenflow Draft (pre-FigJam review)

This is the structure I'll build into the FigJam. Review for accuracy / persona naming / missing flows before I commit to canvas.

Layout target: one end-to-end FigJam board. **Seven horizontal swimlanes** (one per persona) running left-to-right. Each persona's lane shows the screens / states they touch, with arrows for forward routing, dotted arrows for reject loops, and a green fast-path arrow from CSR → PPP.

---

## Personas (swimlanes, top to bottom)

| # | Persona | Role code | Stage(s) owned | Ownership model | Headcount (per Anthony) |
|---|---------|-----------|----------------|------------------|-------------------------|
| 1 | Customer Service Rep | CSR | Order Management (S1) | Self-select / pool | (multiple) |
| 2 | Application Engineer — Validation | AE | Design Validation (S2) | Assigned by CS or AE Lead; EM can reassign | 6 AEs |
| 3 | Drafter / Drawing Services | DS | Drawing Services (S3) | Team pool; EM can reassign | 5 Drafters |
| 4 | Application Engineer — Approval | AE | Design Approval (S4) | Auto from S2; same person who validated | 6 AEs |
| 5 | Manufacturing Engineer | ME | Build Approval (S5) | Team pool, single claim | (TBD) |
| 6 | Configuration Mgmt | CM | Configuration Management (S6) | Team pool / self-select | (TBD) |
| 7 | Production Release | PPP | Planning, Procurement, Production Release (S7) | Team pool / self-select | (TBD) |

**Cross-lane overlay (not a lane):** Engineering Manager (EM) — appears as a floating "Reassign / Workload" control hovering over the AE, DS, and ME lanes. One EM total.

**Off-flow consumers (kept off the main flow, called out as side panels):**
- Leadership — Dashboards (items in process, late count, $ value, bottleneck, throughput)
- Admin — Master data (rejection codes, doc types, paths, stage statuses, role mappings)

---

## Screens / states per persona (the boxes in each lane)

### CSR lane
1. **My Queue — Unassigned Orders** (Evo-fed list, NPS-numbered)
2. **Claim Order** → CSR self-selects from pool
3. **Intake Workspace** — Standard Order template, CUI / FAI / SI / Bid Review fields, attachments + doc-type tagging, permanent notes
4. **Route Decision** — choose path: `→ PPP (fast)` or `→ AE (engineering)`
5. **(later, after PPP)** Send Print to Customer (manual; outside the system but called out)

Statuses CSR can set in-stage: Open · Assigned · Cust PO Correction · Cust Inquiry · App Eng Inquiry · Waiting Bid Review · Hold · Escalated

### AE — Validation lane
1. **AE Pool / My Work** — bucket view; AE can self-claim, EM can reassign
2. **Design Validation Workspace** — Evo BOM context, intake docs, AE-owned fields
3. **AE Design Checklist** — Quote notes, drawing notes, test reqs, specs, customer reqs
4. **Outcome action** — Complete → DS · Reject (with code + closing note) · Escalate · Hold
5. (Same workspace re-entered if DS or AE-Approval rejects back here)

Statuses: Assigned · Cust Inquiry · Mfg Eng Inquiry · Design Hold

### Drafter (DS) lane
1. **DS Pool — Awaiting Drawing** — pooled queue; drafter claims one
2. **Drawing Services Workspace** — AE checklist output + BOM + docs
3. **Drawing Checklist** — must be complete before submission
4. **Outcome action** — Complete → AE Approval · Reject → AE

### AE — Approval lane
1. **Auto-routed inbox** (no claim — same AE who validated)
2. **Design Approval Workspace** — DS package review
3. **Outcome action** — Approve → ME · Reject → DS (or further back)

### ME lane
1. **ME Pool — Awaiting Build Approval** — single-claim
2. **Build Approval Workspace** — drawings + design data, manufacturability check
3. **Outcome action** — Approve → CM · Reject → AE Approval (or further back)

### CM lane
1. **CM Pool — Awaiting Release Review** — pooled
2. **Configuration Mgmt Workspace** — drawing pkg, part setup, release context, signoff conditions
3. **Outcome action** — Complete → PPP · Reject → ME (or further back)

### PPP lane
1. **PPP Pool — Final Release Queue** — pooled, self-select
2. **Final Release Workspace** — drawing pkg + ERP/BOM signals
3. **Action** — Establish initial ESD → **Close Workflow** (timestamp, item exits active queues)

---

## Arrows / routing on the canvas

- **Solid black** — happy-path forward arrows between lanes (CSR → AE → DS → AE → ME → CM → PPP)
- **Solid green** — CSR fast-path: from CSR Route Decision directly to PPP Pool (skips engineering)
- **Dotted red** — reject / loopback arrows. Per the requirements, returns can skip backward more than one stage. Specifically: DS→AE, AE-Approval→DS or →AE-Validation, ME→AE-Approval or further, CM→ME or further. Each labeled "Reject + code + note."
- **Dashed blue** (overlay) — EM "Reassign" arrows hovering across AE / DS / ME pools, indicating manager can shift ownership at any time.

## Cross-cutting elements (callout strip below the lanes)

These show up on every workspace, so I'll put one strip beneath the swimlanes instead of repeating:

- **Stage Visit log** — entry/exit timestamp, owner, outcome, code, notes (immutable)
- **Audit events** — On Stage Entry · Status Change · Assignment Change · Complete · Reject · Approve · Escalate · Hold
- **Permanent notes + doc upload** with stage + doc-type tags
- **SLA timer** — calendar days; does not pause on Hold; tracks both stage touch time and total stage span across re-visits
- **My Work view** — global; every persona has it

## Side panels (not in the flow)

- **Leadership dashboard panel** — Items in process · Late items · $ value · Current bottleneck · Throughput · Days-till-ESD
- **Manager dashboard panel** — Per-contributor turnaround · Items/month · Daily load · Late count · Repeat-return offenders · Monthly bottleneck trend
- **Admin panel** — Rejection codes · Doc types · Workflow paths · Stage statuses · Role/security mappings

---

## Open questions I'd flag for Anthony at our next session

1. **AE Validation = AE Approval = same person?** The Kickoff sheet says S4 is "Automatic from Stage 2," implying yes. Worth confirming — affects whether we render two AE lanes or collapse them.
2. **CM and PPP headcounts / ownership model nuance** — confirmed pool, but how many people sit in each?
3. **Notification model** — when an item lands in your pool / is assigned to you, how do you find out? In-app only, email, both?
4. **"My Work" scope** — does an EM see their team's work in My Work, or only their own? (Their reassign view is separate.)
5. **Hold ≠ pause SLA** is unusual — confirm leadership wants to keep that behavior or whether it's a candidate for v2 refinement.

