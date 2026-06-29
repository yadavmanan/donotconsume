# FDA Recall Traceability Agent

UiPath-powered recall orchestration for fast lot tracing, quarantine coordination, and FDA-ready reporting.

## Overview

Food recalls are the kind of problem that looks administrative until people are still eating the product while someone is searching five systems and twelve emails for one lot code.

The 2018 E. coli romaine lettuce outbreak made that painfully real: it hit 210 people across 36 states and killed 5 before the FDA traced it back to Yuma, Arizona, a process that took weeks.

That was the spark for this project. In food safety, the data usually exists, but the truth is scattered across ERP records, lab results, shipment logs, warehouse inventory, and supplier emails. By the time a human stitches it together, the lettuce has already had a better travel itinerary than most people.

![System architecture](https://ik.imagekit.io/jroc4gnvp/architecture.png)

*Caption: End-to-end architecture of the FDA recall traceability system.*

![The crisis this solves](https://ik.imagekit.io/jroc4gnvp/crisis_solve.png)

*Caption: Why recall speed matters: human risk, compliance pressure, and economic cost.*

We wanted to build a system that turns recall response from slow detective work into immediate action.

Walmart proved in 2019, with IBM blockchain, that faster traceability is possible. But blockchain comes with supplier buy-in, dedicated infrastructure, and long rollout cycles. Our approach is simpler and more deployable: UiPath agents pull traceability records from existing ERP, WMS, email, and supplier systems in minutes, without asking the whole supply chain to rebuild itself first.

$$
T_{\text{response}} \ll 7\ \text{days}
$$

In less mathematical terms: fewer spreadsheets, fewer delays, fewer chances for contaminated product to keep moving.

## At a Glance

- **7 days to 90 seconds** for traceability
- **Under 4 hours** to assemble the compliance package
- **100% KDE completeness** with **0 missed nodes** in the sample run

## What We Learned

The biggest lesson was that recall management is not mainly a data problem. It is a coordination problem.

The crisis above makes that point clearly: the problem is not that companies know nothing, it is that they know different things in different places while the clock, the FDA, and reality keep moving.

Each system knows one part of the story:

- the email knows there is danger
- the LIMS result knows what contamination was detected
- the traceability API knows where the lot traveled
- the inventory system knows what can still be quarantined
- downstream confirmations know whether anybody actually acted

The hard part is aligning all of those fragments around one shared identifier: the traceability lot code.

$$
\text{Truth} = \sum_{i=1}^{n} \text{fragment}_i
\qquad \text{but only if the keys align}
$$

We also learned that compliance output has zero tolerance for improvisation. An FDA-ready report cannot be “basically correct.” It has to be structured, grounded, and defensible.

## How We Built It

We built an orchestrated recall workflow in UiPath using Maestro, Agent Builder, UiPath Apps, API workflows, and email-driven automations. The flow starts with a food safety signal and ends with a final recall summary after quarantine confirmations are collected.

### Workflow

1. **Recall Trigger** reads the inbound email or case payload, or a lab upload from UiPath Apps, extracts product and lot details, classifies urgency, routes the trigger to the same mailbox, and opens the recall case.
2. **Lot Tracing** calls the mock ERP and traceability API to reconstruct the path of `LOT-2026-0437` across origin, inventory, shipments, and downstream nodes.
3. **KDE Report Generation** creates the FSMA 204 compliance package, recall summary, and FDA-facing content.
4. **Quarantine Inventory** marks affected inventory as `QUARANTINE_HOLD`.
5. **Quarantine Notification** sends downstream quarantine notices with lot, units, dates, and action instructions.
6. **Confirmation Listener** waits for reply emails and appends only valid, non-duplicate confirmations.
7. **Final Report Sender** compiles the collected events into a final recall and quarantine confirmation report.

![UiPath Maestro case orchestration](https://ik.imagekit.io/jroc4gnvp/uipath_maestro_case.png)

*Caption: End-to-end recall orchestration in UiPath Maestro Case.*

### Tech stack

- **UiPath Maestro + Agent Builder** for orchestration
- **UiPath Apps** for laboratory-side trigger uploads
- **Python + Flask mock ERP / traceability API** for lot history, inventory, LIMS-style results, and shipment data
- **Email automation** for signal intake, quarantine notifications, and confirmation capture
- **Structured reporting** for FSMA 204 KDE and recall package generation

### What the system outputs

The project produces visible, judge-friendly artifacts rather than just promises:

- a case orchestration view showing the full recall workflow
- an initial recall classification result from the trigger stage
- a lot tracing output tied to the affected lot
- a downstream quarantine notification
- an impact dashboard with the headline metrics
- a final report showing collected recall confirmations

![Recall classification output](https://ik.imagekit.io/jroc4gnvp/recall_classification.png)

*Caption: Initial trigger classifies the recall and extracts the risk context.*

![Lot tracing output](https://ik.imagekit.io/jroc4gnvp/lot_tracing.png)

*Caption: Lot tracing maps the affected path of LOT-2026-0437.*

![Quarantine notification output](https://ik.imagekit.io/jroc4gnvp/quarantine_notification..png)

*Caption: Downstream partners receive immediate quarantine instructions.*

![Impact metrics dashboard](https://ik.imagekit.io/jroc4gnvp/impact_metrics.png)

*Caption: Impact metrics show speed, completeness, and response coverage.*

![Final confirmation report](https://ik.imagekit.io/jroc4gnvp/result_dashboard.png)

*Caption: Final report consolidates confirmations and recall actions.*

In the sample run shown in the outputs:

- traceability time drops from **7 days to 90 seconds**
- the full KDE package is ready in **under 4 hours**
- the dashboard shows **100% KDE completeness** and **0 missed nodes**
- the traced lot response includes the recall class and affected supply-chain path early in the workflow

That is the practical win: the system does not just say “there may be a problem.” It shows the lot, the route, the inventory, the notifications, and the confirmations.

## Challenges We Faced

### 1. One bad lot code breaks everything

The entire workflow depends on consistent lot-code alignment across emails, trace data, inventory records, and reports. One mismatch turns traceability into decorative fiction.

So we made the lot code the anchor for every downstream step.

### 2. Parallel steps are fast and slightly dramatic

KDE generation and quarantine actions run in parallel. That is great for speed and not always great for workflow state unless updates are idempotent.

We had to prevent the case from behaving like two people editing the same document at the same time.

### 3. The mock ERP became mission-critical

At first it looked like demo plumbing. Very quickly, it became the source-of-truth engine for traceability, inventory state, shipments, and lab evidence.

So it had to behave like a believable operating system, not a fake endpoint with nice formatting.

### 4. Compliance content cannot hallucinate

LLMs are useful, but FDA paperwork is not the place for creative writing. We had to keep outputs structured, validated, and tied to actual trace data.

## Closing

We built this project to answer a simple question: when a Class I recall hits, can an agentic system turn fragmented operational data into fast, defensible action?

Our answer is yes.

The workflow detects the signal, traces the lot, quarantines inventory, notifies the supply chain, captures confirmations, and delivers the final report in one coordinated loop.

Basically, we taught recall response to move at internet speed instead of spreadsheet speed.

## Repository layout

- `final/` - main UiPath solution package
- `docs/structured_recall_workflow.md` - detailed workflow reference
- `assets/images/` - evidence images used in the submission
- `assets/emails/` - sample trigger and confirmation emails

## UiPath components used

- UiPath Maestro
- UiPath Agent Builder
- UiPath Apps
- UiPath Integration Service
- UiPath API Workflows
- UiPath Document Understanding
- UiPath Case Management

## Supporting tech

- Python
- Flask
- REST APIs
- Email automation

## Short summary

This is a UiPath-first recall response system that detects a food safety event, traces the lot, quarantines inventory, notifies partners, and sends a final report with no infrastructure rebuild required.

## License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.
