# Amphenol — Screen Journey Spec (for FigJam transcription)

This spec is written to match Nick's house style observed in the "Sample Screen Journey" template:

- **Screen card** = rounded white card with the screen title at top in caps, followed by a `View` block listing what's displayed, and a `Form Actions` block listing buttons. Optional `View Audit Trail` link sits between View and Form Actions.
- **Action button** = blue pill placed to the right of the screen card, connected by an arrow.
- **State-change sticky** = teal note placed to the right of (or above) the action button, describing what changes in the system when the action fires. Match the depth of the Inquiry example: status transition, queue movement, and ownership change.
- **Reject loops** = dotted red arrow back to the prior persona's stage entry screen, with a teal sticky describing the rejection side effects.
- **Fast-path** = solid green arrow from CS Route Decision directly to PPP queue.

Layout target in FigJam: one section ("Screen Journeys") containing **8 horizontal flows**, stacked vertically, in this order: CSR · AE Validation · DS · AE Approval · ME · CM · PPP · Engineering Manager (Workload).

Convention used below:
- `[Screen Title]` — screen card
- `((Action))` — blue pill action button
- `{sticky: ...}` — teal sticky describing state change
- `→` solid black arrow · `⇢ green` fast-path arrow · `⇠ red dotted` reject arrow

---

## Persona 1 — Customer Service Rep (CSR)

Stage owned: **S1 Order Management** · Ownership: Self-select / pool · Statuses: Open · Assigned · Cust PO Correction · Cust Inquiry · App Eng Inquiry · Waiting Bid Review · Hold · Escalated

```
[CS POOL — UNCLAIMED ORDERS]
View
 • List of Evo-fed orders awaiting intake
 • Columns: NPS #, SO / SOLine, Customer Name, Part, Part Desc, Qty, ESD, ASD
 • Filter: Unclaimed · My Claimed · All
 • Status badge per row (Open)
View Audit Trail
Form Actions
 • Claim Order
```
→ ((Claim Order)) → {sticky: "Status: Open → Assigned · Owner = current CSR · Stage Visit opened (S1 entry timestamp) · Item removed from CS Pool"}

```
[ORDER MANAGEMENT WORKSPACE]
View
 • Read-only Evo data: SO, SOLine, Customer, Part, Part Desc, Qty, ESD, ASD
 • CSR-owned fields (editable): CUI · FAI · SI · Bid Review Required (Y/N each)
 • Quote document viewer
 • Permanent Notes (append-only)
 • Attachments table (file · doc type · intended stage)
 • Stage Status selector: Open · Cust PO Correction · Cust Inquiry · App Eng Inquiry · Waiting Bid Review
View Audit Trail
Form Actions
 • Save (in-progress)
 • Upload Attachment
 • Add Note
 • Change Status
 • Hold
 • Escalate
 • Route Forward →
```
→ ((Save)) → {sticky: "Stage Visit remains open · Field edits committed · No status change · SLA timer continues"}

→ ((Change Status: e.g. Cust Inquiry)) → {sticky: "Status: Assigned → Cust Inquiry · Audit event 'Status Change' written · Item stays in CS owner = current CSR · SLA timer continues (does not pause on hold)"}

→ ((Hold)) → {sticky: "Status: Assigned → Hold · Outcome 'Hold' on Stage Visit · Audit event 'OnHold' · SLA does NOT pause (per Anthony)"}

→ ((Escalate)) → {sticky: "Status: Assigned → Escalated · Audit event 'OnEscalate' · Item flagged in Manager dashboard"}

→ ((Route Forward)) → opens routing modal

```
[ROUTE DECISION — modal]
View
 • Item summary header
 • Two route choices presented as cards:
   – PPP Direct (item already set up in ERP)
   – Engineering Path (CS → AE → DS → AE Approval → ME → CM → PPP)
 • Required: Closing Note (optional) · Route confirmation
View Audit Trail
Form Actions
 • Cancel
 • Confirm: Route to PPP
 • Confirm: Route to AE
```

⇢ green ((Confirm: Route to PPP)) → {sticky: "S1 Stage Visit closed · Outcome 'Complete' · Route value = 'CS→PPP Fast' · S7 PPP Stage Visit opened · Item appears in PPP Pool · Workflow current stage = PPP"} → jumps to PPP lane

→ ((Confirm: Route to AE)) → {sticky: "S1 Stage Visit closed · Outcome 'Complete' · Route value = 'Engineering' · S2 Design Validation Stage Visit opened · CSR (or AE Lead) selects target AE · Item appears in AE's assigned queue (not pool)"} → jumps to AE Validation lane

```
[(Later) SEND PRINT TO CUSTOMER]
View
 • Item closed-state summary
 • Final drawing package preview / download
 • Customer contact info
Form Actions
 • Mark Sent (timestamp + initials)
```
*(Note: this final CSR touch happens after PPP closes the workflow; not part of the active workflow stages but called out so CSR's full journey is honest.)*

---

## Persona 2 — Application Engineer (Validation)

Stage owned: **S2 Design Validation** · Ownership: Assigned by CS or AE Lead · EM can reassign · Statuses: Assigned · Cust Inquiry · Mfg Eng Inquiry · Design Hold

```
[MY WORK — AE]
View
 • Items assigned to me (AE)
 • Columns: NPS #, Customer, Part, ESD, Days in Stage, Status
 • Filter: My Validation Work · My Approval Work · All My AE Work
View Audit Trail
Form Actions
 • Open Item
```
→ ((Open Item)) → {sticky: "No state change · Workspace opens for active Stage Visit"}

```
[DESIGN VALIDATION WORKSPACE]
View
 • Header: Item summary (NPS#, Customer, Part, ESD, Current Status)
 • Intake context (read-only): Evo SO data, CSR notes, CUI/FAI/SI/Bid flags, attachments tagged "Intake"
 • Evo BOM panel (read-only)
 • AE Design Checklist (Mendix form):
   – Quote notes
   – Drawing notes
   – Test requirements
   – Specs / customer requirements
   – Design manufacturable? (Y/N)
   – BOM set up correctly? (Y/N)
 • Permanent Notes (append-only)
 • Attachments table (file · doc type · intended stage)
 • Stage Status selector: Assigned · Cust Inquiry · Mfg Eng Inquiry · Design Hold
View Audit Trail
Form Actions
 • Save
 • Upload Attachment
 • Add Note
 • Change Status
 • Hold
 • Escalate
 • Reject ⇠
 • Complete →
```
→ ((Save)) → {sticky: "Field edits committed · Stage Visit stays open · SLA continues"}

→ ((Change Status: e.g. Mfg Eng Inquiry)) → {sticky: "Status: Assigned → Mfg Eng Inquiry · Audit 'Status Change' · SLA continues"}

→ ((Hold)) → {sticky: "Status: Assigned → Design Hold · Audit 'OnHold' · SLA continues (no pause)"}

→ ((Reject)) → opens rejection modal:

```
[REJECT — modal]
View
 • Required: Rejection Code (controlled list)
 • Required: Return-to-Stage selector (CS only here, since S2 is the first engineering stage)
 • Optional: Closing Note
Form Actions
 • Cancel
 • Confirm Reject
```
⇠ red dotted ((Confirm Reject)) → {sticky: "S2 Stage Visit closed · Outcome 'Reject' · Rejection Code stored · S1 (CS) Stage Visit re-opened · Item appears in CSR pool (or assigned to original CSR — TBD with Anthony) · Audit 'OnReject'"} → loops back to CSR Order Management Workspace

→ ((Complete)) → {sticky: "S2 Stage Visit closed · Outcome 'Complete' · S3 Drawing Services Stage Visit opened · Item appears in DS Pool (team pool) · Audit 'OnComplete'"} → next: DS

---

## Persona 3 — Drafter / Drawing Services (DS)

Stage owned: **S3 Drawing Services** · Ownership: Team Pool · Statuses: Open · Assigned

```
[DS POOL — AWAITING DRAWING]
View
 • Items in DS pool
 • Columns: NPS #, Customer, Part, ESD, Time in Pool, Source AE
 • Status badge: Open
View Audit Trail
Form Actions
 • Claim Item
```
→ ((Claim Item)) → {sticky: "Status: Open → Assigned · Owner = current Drafter · Item removed from DS Pool · Manager can still reassign"}

```
[DRAWING SERVICES WORKSPACE]
View
 • Header: Item summary
 • AE Design Checklist output (read-only)
 • All upstream attachments (CSR + AE)
 • Evo BOM panel
 • Drawing Checklist (Mendix form):
   – Drawings created
   – Drawings revised
   – Build package drawings done
   – Q notes / dwg notes / test req captured
 • Permanent Notes
 • Attachments table — drafter uploads drawing files; doc type = Drawing; intended stage = AE Approval / ME / CM as appropriate
View Audit Trail
Form Actions
 • Save
 • Upload Drawing
 • Add Note
 • Reject ⇠
 • Complete →
```
→ ((Reject)) → modal (Code + Return-to-Stage = AE Validation) → ⇠ red dotted ((Confirm Reject)) → {sticky: "S3 Stage Visit closed · Outcome 'Reject' · S2 Stage Visit re-opened for original AE · Item back in AE's My Work · Audit 'OnReject'"}

→ ((Complete)) → {sticky: "S3 Stage Visit closed · Outcome 'Complete' · S4 AE Design Approval Stage Visit opened · Auto-routed to original AE (same person who validated) · Audit 'OnComplete'"} → next: AE Approval

---

## Persona 4 — Application Engineer (Approval)

Stage owned: **S4 Design Approval** · Ownership: Auto from S2 (same AE who validated) · Statuses: Assigned

*Note: same human as Persona 2. Rendered as a distinct journey because the workspace, available actions, and audit events differ.*

```
[MY WORK — AE (Approval section)]
View
 • Auto-routed approval queue (no claim — already mine)
 • Columns: NPS #, Customer, Part, ESD, Days in Stage
View Audit Trail
Form Actions
 • Open Item
```
→ ((Open Item))

```
[DESIGN APPROVAL WORKSPACE]
View
 • Header: Item summary
 • DS drawing package (read-only viewer + downloads)
 • Drawing Checklist output (read-only)
 • Full upstream history collapsed (intake + validation + drawing)
 • Sign-off panel (timestamp + initials on submit)
View Audit Trail
Form Actions
 • Reject ⇠
 • Approve →
```
→ ((Reject)) → modal (Code + Return-to-Stage = DS *or* AE Validation depending on issue) → ⇠ red dotted → {sticky: "S4 Stage Visit closed · Outcome 'Reject' · Selected prior Stage Visit re-opened · Audit 'OnReject' · Rejection Code stored"}

→ ((Approve)) → {sticky: "S4 Stage Visit closed · Outcome 'Approve' · Approver name + timestamp stored as electronic-signature evidence · S5 ME Build Approval Stage Visit opened · Item appears in ME Pool · Audit 'OnApprove'"} → next: ME

---

## Persona 5 — Manufacturing Engineer (ME)

Stage owned: **S5 Build Approval** · Ownership: Team Pool, single claim · Statuses: Open · Assigned

```
[ME POOL — AWAITING BUILD APPROVAL]
View
 • Items in ME pool
 • Columns: NPS#, Customer, Part, ESD, Time in Pool
View Audit Trail
Form Actions
 • Claim Item
```
→ ((Claim Item)) → {sticky: "Status: Open → Assigned · Owner = current ME · Removed from ME Pool · Single-claim enforced (no co-reviewer allowed in v1)"}

```
[BUILD APPROVAL WORKSPACE]
View
 • Header: Item summary
 • Drawing package (read-only)
 • Design data + AE approval sign-off (read-only)
 • Manufacturability review fields (Y/N + notes):
   – Tolerances achievable
   – Tooling available
   – Material lead time acceptable
 • Permanent Notes
 • Attachments
View Audit Trail
Form Actions
 • Save
 • Reject ⇠
 • Approve →
```
→ ((Reject)) → modal (Code + Return-to-Stage = AE Approval, AE Validation, or DS depending on issue) → ⇠ red dotted → {sticky: "S5 Stage Visit closed · Outcome 'Reject' · Selected prior Stage Visit re-opened · Audit 'OnReject'"}

→ ((Approve)) → {sticky: "S5 Stage Visit closed · Outcome 'Approve' · ME approver + timestamp stored · S6 CM Stage Visit opened · Item appears in CM Pool · Audit 'OnApprove'"} → next: CM

---

## Persona 6 — Configuration Management (CM)

Stage owned: **S6 Configuration Management** · Ownership: Team Pool / self-select · Statuses: Open · Assigned

```
[CM POOL — AWAITING RELEASE REVIEW]
View
 • Items awaiting CM
 • Columns: NPS#, Customer, Part, ESD, Time in Pool
View Audit Trail
Form Actions
 • Claim Item
```
→ ((Claim Item)) → {sticky: "Status: Open → Assigned · Owner = current CM user · Removed from CM Pool"}

```
[CONFIGURATION MANAGEMENT WORKSPACE]
View
 • Header: Item summary
 • Full drawing package (read-only)
 • Evo part setup data (read-only)
 • Release/sign-off checklist:
   – Revision controls verified
   – Procedures followed
   – Test/compliance controls applied
 • Permanent Notes
View Audit Trail
Form Actions
 • Save
 • Reject ⇠
 • Complete →
```
→ ((Reject)) → modal (Code + Return-to-Stage = ME or further back) → ⇠ red dotted → {sticky: "S6 Stage Visit closed · Outcome 'Reject' · Selected prior Stage Visit re-opened · Audit 'OnReject'"}

→ ((Complete)) → {sticky: "S6 Stage Visit closed · Outcome 'Complete' · S7 PPP Stage Visit opened · Item appears in PPP Pool · Audit 'OnComplete'"} → next: PPP

---

## Persona 7 — Production Release (PPP)

Stage owned: **S7 Planning, Procurement, & Production Release** · Ownership: Team Pool / self-select · Statuses: Open · Assigned

```
[PPP POOL — FINAL RELEASE QUEUE]
View
 • Items awaiting final release
 • Columns: NPS#, Customer, Part, ESD, Days till ESD, Source path (Engineering / Fast-path)
View Audit Trail
Form Actions
 • Claim Item
```
→ ((Claim Item)) → {sticky: "Status: Open → Assigned · Owner = current PPP user · Removed from PPP Pool"}

```
[FINAL RELEASE WORKSPACE]
View
 • Header: Item summary
 • Drawing package + ERP/BOM signals (read-only)
 • Initial ESD field (editable — establish or confirm)
 • Permanent Notes
View Audit Trail
Form Actions
 • Save
 • Establish Initial ESD
 • Close Workflow →
```
→ ((Establish Initial ESD)) → {sticky: "Initial ESD value persisted · Audit 'Status Change' if changed · No stage transition yet"}

→ ((Close Workflow)) → {sticky: "S7 Stage Visit closed · Outcome 'Complete' · Workflow Item Status: Active → Closed · Workflow Closed Timestamp stamped · Item removed from all active queues · Total cycle time finalized · Closed item remains searchable for audit/reporting · Audit 'OnComplete' (workflow close)"}

→ Triggers CSR's "Send Print to Customer" downstream task (out-of-system today).

---

## Persona 8 — Engineering Manager (Workload Management)

*Not a stage owner. EM has a dedicated workload view that overlays the AE / DS / ME pools and assignments.*

```
[ENGINEERING MANAGER — WORKLOAD DASHBOARD]
View
 • Three side-by-side panels: AE · DS · ME
 • For each panel:
   – Active items grouped by current owner (or "Unclaimed" for pools)
   – Per-person counters: # assigned · # in-progress · # late · avg days in stage
   – Color-coded heatmap of load per person
 • Filter: Stage · Status · Late only · Customer · ESD window
 • Per-row item summary: NPS#, Customer, Part, ESD, Days in Stage, Current Owner, Status
View Audit Trail
Form Actions
 • Reassign Item
 • View Item Workspace
 • View Stage History
```
→ ((Reassign Item)) → opens modal:

```
[REASSIGN — modal]
View
 • Current owner (read-only)
 • New owner picker (filtered by role: AE / DS / ME)
 • Required: Reason (controlled list or note)
Form Actions
 • Cancel
 • Confirm Reassign
```
→ ((Confirm Reassign)) → {sticky: "Owner changed · Audit 'Assignment Change' written · Stage Visit remains the same (no new visit) · Reason stored · New owner sees item in My Work · Old owner no longer sees it"}

→ ((View Stage History)) → opens read-only timeline of all Stage Visits for the item — entry/exit, owner changes, outcomes, codes, notes.

---

## Cross-cutting elements (callout strip below the lanes)

These behaviors apply on every persona's workspace and don't need to be re-drawn per lane. Place once as a horizontal strip beneath the 8 journeys:

- **Stage Visit log** — every transition writes a row (entry ts, exit ts, owner, outcome, code, note); immutable
- **Audit events** fired: `OnStageEntry · OnStatusChange · OnAssignmentChange · OnComplete · OnReject · OnApprove · OnEscalate · OnHold`
- **Permanent notes + classified attachments** (file · doc type · intended stage) — append-only on every workspace
- **SLA** — calendar days, never pauses (even on Hold), tracks stage touch time *and* total stage span across re-visits, plus total workflow age and days-till-ESD
- **My Work** — global personal view available to every persona
- **Single owner / single stage** invariant — no parallel ownership; reassign moves ownership without opening a new Stage Visit

---

## Side dashboards (separate frames; not in the screen-flow strip)

- **Leadership Dashboard** — Items in process · Late items · $ value in process · Current bottleneck stage · Throughput · Days-till-ESD distribution
- **Manager Dashboard** — Per-contributor turnaround · Items processed / month · Daily avg load · Late count · Repeat-return offenders · Monthly bottleneck trend
- **Admin Console** — Rejection codes (by stage) · Document types · Workflow paths · Stage statuses · Role/security mappings

---

## Open questions to confirm with Anthony before final FigJam pass

1. **Reject from S2 (AE Validation) back to CS** — does the item land in the CSR pool again, or get assigned back to the original CSR? The Kickoff sheet doesn't specify. Recommend: assigned back to original CSR for accountability.
2. **Reassignment audit reason** — controlled list or free text? Recommend controlled list to support reporting.
3. **EM's role at S5 (ME)** — Anthony's email mentions EM reassigns AE/DS/ME. Confirm EM scope covers ME, since ME has a different reporting line at most orgs.
4. **Notification model** — when does a user learn an item is in their queue/assigned to them? In-app badge only, or email push too?
5. **"Hold doesn't pause SLA"** is unusual. Confirm Anthony wants this in v1 or wants a "Hold reasons" controlled list with selective pause behavior in v2.
6. **CSR fast-path eligibility** — purely CSR judgement, or driven by an "Already in ERP" flag from Evo? Affects whether Route Decision modal needs a guardrail.

