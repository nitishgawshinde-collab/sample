# DocuFlow · Two-sprint development plan
## Single Tax Document Verification + Country-Specific Tax Documents

**Planning date:** October 23, 2026  
**Duration:** Two sprints × 15 working days = 30 working days. If your organization means calendar days, replan capacity before commitment. No start/end dates are assumed.  
**Outcome:** A developer-verified React → Java API → Flowable → PostgreSQL implementation of both document workflows, demonstrated with synthetic data and isolated development adapters.  
**External systems:** iCRM, HIVE and OBS receive analysis stories only. Their actual connectors, live API calls, external event subscriptions and production write-back are outside these sprints.  
**QA:** No QA allocation or dependency in this plan. Developers own unit, API, model and UI checks. Formal independent QA/UAT and production readiness follow separately.

> Working note: build the common document lifecycle once, finish one vertical workflow in Sprint 1, and reuse it for the country-specific path in Sprint 2. Avoid separate document models and screens for each country.

## 1. Scope fence

### Included
- React 19 document intake, metadata/index fields, role-based task queues, task review, read-only multipage preview and document history.
- JDK 17 Java application APIs for intake, document queries, authorized task actions, deficiency/correction and outcome/history retrieval.
- Flowable Enterprise models for EW1 and EW2, common delegates and configurable routing based on approved document type/ownership/format rules.
- PostgreSQL application schema, migrations, integrity constraints and access patterns; Flowable manages its own engine schema.
- Document identity/version references, controlled intake, structured reasons, retained rejected/replaced records, and distinct document/workflow outcomes.
- Development-only synthetic personas and integration test doubles, clearly isolated from production configuration.
- External integration discovery, data mapping and contract proposals only.

### Deferred
- BCTS assessment, Tax Relief, Payment, account opening, OBS data input/check and account-created orchestration.
- Real HIVE/iCRM/OBS connectors, live synchronization, real signatures, Office editing, OCR/AI extraction, full-text search and native mobile applications.
- General-purpose workflow designer, full retention/legal-hold/disposal implementation, bulk migration and production archival automation.
- Production identity onboarding, deployment certification and independent QA/UAT. Reuse existing identity infrastructure where available; do not turn its absence into an implied production delivery commitment.

## 2. Team and capacity — make the six development roles explicit

| Role alias | Person allocation | Main ownership |
|---|---:|---|
| Modeler | 1 | BPMN/DMN, deployment definitions, model scenario checks |
| React Dev | 1 | React components, accessibility, preview and UI integration |
| FS-A / Tech Lead | 1 | API/security contracts, review, Flowable application boundary |
| FS-B | 1 | PostgreSQL migrations, document service and data contracts |
| FS-C | 1 | Workflow/task services, frontend support and developer integration checks |
| Junior | 1 fresher | Bounded fixture, documentation and supervised UI/test support |
| QA | 1 | **Zero allocation in these two sprints**, as requested |

FS-A is one of the three full-stack developers, not an additional person. The plan assumes each full-stack developer can support React/Java as needed.

**Per sprint:** 5 experienced developers × 15 days = 75 gross person-days. Commit **50 experienced person-days**: approximately 10 per person. Reserve the remaining 25 for ceremonies, technical leadership, reviews, mentoring, defects and dependency delays. This reserve is not extra feature capacity.

Junior: up to **6 supervised support days** per sprint; the remainder supports learning and feedback. Junior output is not a prerequisite for a critical-path story. Mentoring is covered by the experienced-team reserve. Do not count the fresher as a full-capacity developer.

Estimates below are combined experienced-person effort, not elapsed duration or story points. They are planning estimates to refine after the first design session. Shared stories are delivered once for both workflows.

## 3. Decisions needed at the start

| Decision | Needed by | Proposed owner / required answer |
|---|---|---|
| Authoritative scope and rules | Sprint 1 day 3 | Business + document operations: confirm the stated 20 EW1 and 25 EW2 type/ownership combinations; provide the actual matrix |
| Workflow trigger boundary | Sprint 1 day 3 | Modeler + business: EW1 after valid registration; EW2 receipt/intake versus post-registration start; agree one start boundary and deduplication key |
| Role and signature responsibility | Sprint 1 day 3 | Front Office, CDDM Luxembourg/Poland, Tax: who may upload, verify signature, review quality and reject/return |
| Formats and country exceptions | Sprint 1 day 3 | Business: permitted Paper Original/Digital Original/Copy Valid as Original/Simple Copy combinations; no blanket country rule |
| Tax documentary review | Sprint 1 day 3 | Tax: when required, eligible reviewer, and whether it changes document completion |
| Return/incomplete/rejection | Sprint 1 day 3 | Business: terminal versus correctable states, same-process continuation, resubmission and permitted replacement behavior |
| Identity and version semantics | Sprint 1 day 3 | Architecture + document team: logical document ID, version ID and replacement lineage; no destructive overwrite |
| Stack and local startup | Sprint 1 day 2 | Tech lead/platform: actual licensed Flowable version and corporate parent; database, React baseline and local authentication availability |

Earlier material mentions Flowable 3.17.16 and “2025.2.” Use the actual repository's approved dependency baseline; do not treat those labels as interchangeable or schedule an upgrade here.

If a rule remains unresolved, build the independent technical work and expose a visible decision/configuration hold. Demonstrate draft rules only in an isolated demo profile. Do not default missing rules to approval, omitted review or unrestricted upload. Escalate the delivery impact at the day-3 checkpoint.

## 4. Sprint 1 — common foundation + EW1 vertical slice

**Sprint goal:** An eligible, validly registered document automatically creates one EW1 workflow. An authorized reviewer can process the document, see every page and record a traceable outcome through the React UI.

| ID | Story / concrete deliverable | Primary / pairing | Effort |
|---|---|---|---:|
| S1-01 | Analyze EW1/EW2 states, role/format rules and trigger boundaries; baseline OpenAPI and screen contracts | Modeler + FS-A; React Dev/FS-B | 4 days |
| S1-02 | Establish Java application/security baseline and local personas; current user, resource authorization and safe errors | FS-A + FS-C | 5 days |
| S1-03 | Design shared PostgreSQL schema and migrations; document identity, version, context, outcomes and audit | FS-B + FS-A | 5 days |
| S1-04 | Model EW1 routing, human tasks, return/correction and final outcomes; approved rule table, model deployment and scenario checks | Modeler + FS-A | 6 days |
| S1-05 | Implement document intake/registration, idempotent local workflow start and authorized task-action APIs | FS-A + FS-B/FS-C | 8 days |
| S1-06 | Build React shared workspace, document intake, task list/detail and read-only multipage viewer | React Dev + FS-C | 9 days |
| S1-07 | Integrate the EW1 vertical slice; exercise negative paths, concurrency, audit and fresh-database startup | FS-C + all relevant owners | 7 days |
| S1-08 | External integration discovery: HIVE, iCRM and OBS applicability/ownership; contract proposals only | FS-B + FS-C/Modeler/FS-A | 6 days |
| | **Experienced-team commitment** | | **50 days** |

### Story acceptance notes

**S1-01 — analysis before hard-coding.** Deliver state-transition and role-action tables, EW1/EW2 sequence sketches, common field dictionary and decision log. Separate intake/technical validation, human verification and final business outcome. Mark proposed/unapproved routing explicitly. Agree UI actions and API outcomes before model implementation diverges.

**S1-02 — identity and scope.** Reuse the corporate parent/security components after inspecting the actual repository. Demonstrate authenticated current-user access, anonymous rejection and cross-document/task access rejection. Derive actor from authentication. Demo users must be limited to local/test profiles; browser persona selection is not the authorization implementation.

**S1-03 — schema first, shared once.** Commit reviewed migrations and ERD; verify a fresh database can start and migrations apply repeatably. Add document/context/type/version constraints, optimistic version columns and an approved deduplication constraint. Keep engine tables separate; do not duplicate Flowable's task state as a competing source of truth.

**S1-04 — EW1 engine behavior.** Candidate flow: registration → determine route → signature/ownership check if applicable → quality/index verification → Tax documentary review if applicable → record outcome. Model exact task ordering/parallelism from the approved matrix. Cover the chosen correction loop and terminal rejection separately. A missing route cannot silently approve. Record deployed definition and rule versions.

**S1-05 — usable API slice.** Provide document metadata/query, registration, authorized task listing/claim/completion and history APIs. Registration starts the configured workflow automatically; the user does not manually select an embedded workflow. Repeating an identical registration does not start a second instance. Conflicting replay and concurrent task completion produce the agreed safe response. Allowlist task fields/outcomes; do not pass unrestricted variable maps to Flowable.

**S1-06 — practical UI.** Provide labeled index fields, actionable errors, loading/empty states and stage/owner/next-action indicators. Reuse a common task grid; use the licensed AG Grid only where it improves the queue. Preview all pages of a specimen PDF, with navigation/zoom through an approved viewer. No editor/signature commitment. Upload controls reflect role/format restrictions while backend validation remains authoritative.

**S1-07 — developer-owned verification.** Show normal completion, incomplete/return plus correction, rejection, unauthorized action, repeated registration and stale/concurrent completion. Assert the persisted document outcome and workflow history, not just that a task disappeared. Recheck all pages using at least a 3-page specimen and one larger specimen. No QA assignment is created.

**S1-08 — analysis only.** Deliver INT-A01/A02/A03 outputs described below. No production SDK, endpoint call, external database connection, service credential or write-back is needed to accept this story.

**Junior support:** maintain the synthetic document fixture inventory; document local setup; help build a simple read-only metadata panel with pairing; collect developer demo evidence. Do not assign security, database ownership, workflow routing or integration implementation.

### 15-day execution notes
- **Days 1–3:** design/rule workshop, repository startup, state/API/schema contracts; review the unresolved decisions on day 3.
- **Days 4–7:** migrations, EW1 model and backend services in parallel; React works from the same agreed contract/test fixtures.
- **Day 8:** first React → API → engine → database walkthrough. Avoid waiting for day 14 to integrate.
- **Days 9–12:** finish human actions, viewer and correction path; exercise failures and replay.
- **Days 13–15:** fix/review, developer verification, demo and Sprint 2 planning. Freeze new features during this window.

**Sprint 1 exit:** One repeatable EW1 vertical slice; shared migration/API/component baseline; visible unresolved-rule holds; integration discovery notes; demo script and actual verification evidence. Production external synchronization is not an exit condition.

## 5. Sprint 2 — EW2 + shared lifecycle hardening

**Sprint goal:** Country-specific document intake and verification reuse the same document services and React task workspace, retain history through correction/replacement, and run beside EW1 without duplicate starts.

| ID | Story / concrete deliverable | Primary / pairing | Effort |
|---|---|---|---:|
| S2-01 | Finalize country matrix and receipt/scanning handoff; close or explicitly block missing EW2 rules | Modeler + FS-A/React Dev/FS-B | 4 days |
| S2-02 | Implement EW2 BPMN/DMN, receipt/scanning/verification steps and scenario checks using the agreed boundary | Modeler + FS-A | 6 days |
| S2-03 | Extend APIs for country intake, scanning acknowledgment, metadata verification and correction with concurrency protection | FS-A + FS-B/FS-C | 8 days |
| S2-04 | Extend React with country/context intake, scanning and reviewer queues; reuse document/task/preview components | React Dev + FS-C | 9 days |
| S2-05 | Implement shared deficiency/resubmission and controlled version/replacement history, including EW1 regression | FS-B + FS-A/Modeler | 7 days |
| S2-06 | Add durable local completion-event records/replay checks; validate outcome/audit transactions without external publishing | FS-C + FS-B | 5 days |
| S2-07 | Refine external contracts and decision log with system teams; contract proposals and future backlog only | FS-B + FS-A/Modeler | 5 days |
| S2-08 | Developer verification of both paths, performance sanity check, demo and technical handoff | FS-C + React Dev/Modeler/FS-A | 6 days |
| | **Experienced-team commitment** | | **50 days** |

### Story acceptance notes

**S2-01 — approve the matrix.** Record country + document type + Account/Partner context, permitted formats, upload channels, routing and evidence requirements. Country name does not become a separate model by default. Include representative exceptions and missing-rule cases. Confirm how receipt records link to the registered document/version.

**S2-02 — explicit intake boundary.** Candidate flow: receipt → scanning/classification → valid registration → verification → finalization. If the approved start boundary is later, represent receipt/scanning in the intake service rather than starting a second workflow. A repeated receipt/scan acknowledgment cannot create another instance. Test wait/correction/rejection only where the agreed workflow actually requires them.

**S2-03 — common services, specialized context.** Reuse S1 document/task endpoints and validation services where semantics match. Add country/receipt/custody references, not document bytes in process variables. Enforce authorized scanning and verification; missing ownership or invalid formats cannot start verification. Do not make an OBS account lookup mandatory for documentary completion unless the approved rule explicitly requires it.

**S2-04 — reduce user effort.** Drive visible/required fields from the agreed form contract. Show pending receipt/scanning versus verification clearly. Provide claim/open/return/correct interactions, inline reasons and a visible history of prior submissions. Require a reason for return/rejection. Use professional status colors and text; never communicate outcomes by color alone.

**S2-05 — preserve evidence.** A corrected version/replacement retains the original and its outcome/history. Link version/replacement explicitly, revalidate the submitted facts and reroute according to the approved rule. Do not overwrite signed originals or assert an unapproved versioning policy. If lifecycle semantics remain open, demonstrate lineage with synthetic fixtures and keep the dependent acceptance transition blocked.

**S2-06 — internal reliability.** Persist the agreed verification-completed event payload alongside outcome/audit data, with stable event and document/version references. Exercise duplicate/replay locally. Use a small local outbox/event record if no shared implementation exists; this is internal application reliability, not a HIVE/iCRM/OBS connector. No external dispatcher is delivered. Keep delivery state separate from document completion.

**S2-07 — analysis only.** Revise INT-A01/A02/A03 contracts, field ownership, unresolved questions, error/retry/reconciliation scenarios and future implementation stories. Capture OBS as not needed for these paths if discovery confirms that. Do not invent a connector solely because the bank uses the system.

**S2-08 — evidence for the next phase.** Run both workflow suites against real models and a fresh/migrated database. Demonstrate switching eligible roles, multipage review, missing policy, duplicate input, correction history and distinct incomplete/rejected/complete outcomes. Run a small bounded document/queue test and record observations; no invented production throughput target. Produce handoff notes and remaining business/integration decisions. Formal QA/UAT is still a separate phase.

**Junior support:** extend synthetic country/document fixtures from approved rows, maintain field dictionary and screenshots, assist with supervised UI regression/checklists and setup notes. No independent ownership of migrations, rules or release readiness.

### 15-day execution notes
- **Days 1–3:** agree country rules and receipt boundary; finalize only the EW2-specific extensions.
- **Days 4–7:** model/API/React work in parallel; reuse S1 contracts and components.
- **Day 8:** first full EW2 walkthrough with receipt/scanning handoff and documentary decision.
- **Days 9–12:** replacement/correction, local event reliability and cross-workflow regressions.
- **Days 13–15:** stabilize, peer review, developer verification, business demo and handoff.

**Sprint 2 exit:** Two repeatable workflow slices, common schema/API/UI patterns, retained evidence/history, internal completion-event records and a reviewed integration-analysis backlog. Do not describe this as production release or independent QA sign-off.

## 6. External integration analysis stories — no implementation tasks

These are sub-stories of **S1-08 (6 days) and S2-07 (5 days)**, not additional capacity. Developers may use synthetic in-process fixtures to complete the core workflow, but those fixtures do not validate a real service contract.

| ID / estimate across both sprints | Analysis story | Required outputs / acceptance |
|---|---|---|
| INT-A01 · HIVE · 5 days | As an architect, establish the document golden-source integration contract so the later connector has clear ownership and failure behavior | Identify what HIVE actually is; API/event or other approved access mechanism; logical ID/version/replacement rules; binary storage versus Blob pointers; metadata/status ownership; authorization; upload/download limits; malware-check responsibility; consistency; dedupe; error/retry/reconciliation; proposed sequence and mapping with open items labeled |
| INT-A02 · iCRM · 3 days | As an architect, define context and responsibility lookup for document ownership and task routing | Client/Partner/Account references; accountable owner versus uploader; lookup/search fields; freshness; access; missing/mismatch/unavailable responses; proposed request/response examples; route/group mapping questions and contract-test candidates |
| INT-A03 · OBS · 3 days | As an architect, determine whether these two documentary workflows require OBS at all | Separate documentary completion from account/implementation milestones; identify any mandatory read-only lookup; authoritative identifiers, not-found/unavailable handling and ownership; either document a narrowly scoped future contract or record no dependency with business rationale |

**Sprint 1:** initial discovery and draft contracts. **Sprint 2:** review feedback, refine contracts and produce future connector stories. If system owners cannot review in time, acceptance is an explicit draft with questions/owner/status; it is not a claim that the contract is agreed.

## 7. Architecture notes to pin above the desks

**React:** shared document intake + task workspace + read-only multipage viewer + timeline. Keep common components separate from EW1/EW2 field/rule configuration. UI restrictions mirror server rules; they cannot approve access.

**Java:** controllers → application services → supported Flowable/document repositories. Services enforce role/resource access, transition validation, idempotency and audit. Thin delegates invoke these services. Public contracts expose purpose-specific DTOs and task actions, not raw engine entities or arbitrary variable maps.

**Flowable:** two embedded workflow definitions unless the approved requirements justify another arrangement; shared services and routing rules. Use licensed APIs already in the repository. Keep keys stable; bind process/rule/form versions. Human-task outcomes do not automatically mean successful document verification.

**PostgreSQL:** own document metadata/references, verification outcomes and business audit. Flowable owns its tasks/process history. Use migrations and explicit ownership; do not write engine tables directly. Avoid a new microservice for every concern in this 30-day slice.

**Development content adapter:** isolate a disposable development content store or approved sandbox behind a content-service interface. No HIVE connector or live Blob integration is required here. Never enable a production fallback that silently stores golden-source documents in this test store. Keep bytes out of Flowable variables.

### Proposed application schema — review in S1-03

| Table / aggregate | Main data | Integrity / ownership note |
|---|---|---|
| document | Internal ID, external reference if known, type, ownership context, document outcome, row version | Preserve logical identity; external ID uniqueness follows confirmed source/context semantics |
| document_version | Version ID, document ID, checksum, content reference, file metadata, submitted-by/time | Immutable content reference; no in-place overwrite; approved version/replace policy needed |
| intake_record | Receipt/source/channel, country where relevant, custody reference, format, registration key, processing state | Durable deduplication and receipt-to-version linkage |
| verification_record | Document version, model instance/reference/version, decision/rule versions, outcome, actor/time | One permitted active verification per agreed version/context; restart/correction policy controls uniqueness |
| deficiency | Verification reference, structured reason, comment, status, resolution/acceptance references | Resolved and accepted are separate where required |
| document_relation | Old/new version or document IDs, relation type, reason | Replacement/supersession lineage; prevent invalid/cyclic references |
| business_audit_event | Event ID, target/version, action, actor, time, reason and correlation | Append through controlled application path; not a claim of compliant immutable storage |
| completion_event_outbox | Stable event ID, payload/schema version, verification reference, delivery/replay state | Added in Sprint 2; no external dispatcher in scope |

Routing/format configuration must have approved versioned ownership. Avoid maintaining competing copies of the same rule in React, Java, PostgreSQL and DMN. Decide the rule source in S1-01; store references/versions where needed.

## 8. Definition of done for development stories

- Reviewed code/model/migration with clear ownership and no fabricated FRD IDs.
- Relevant developer unit, API, model and UI checks executed; failed/blocked/not-run results reported honestly.
- Authentication and resource access enforced server-side; actor and allowed variables are controlled.
- Fresh-database startup and affected migrations verified when schema changes.
- Return/rejection reasons and evidence history retained; no silent approval on missing rule.
- Repeatable synthetic demo and concise setup notes. No real client content or secrets committed.
- Review with business owners where a story defines policy; unavailable approvals remain visible holds.

## 9. Delivery risks and trade-offs

| Risk | Effect | Response / reduction order |
|---|---|---|
| Rules not available by day 3 | Cannot truthfully finish all 45 stated combinations | Keep technical slice moving; show representative synthetic routes and explicitly report unvalidated coverage |
| Licensed baseline/security startup not working | Vertical slice stalls | Front-load S1-02; escalate platform access, reserve time; do not replace corporate dependencies |
| Multipage viewer library approval delayed | Preview integration slips | Use approved browser PDF rendering for initial read-only review; defer custom annotation/editor work |
| Scope grows into Tax assessment/OBS implementation | Two-sprint goal becomes unrealistic | Defer to new backlog; preserve the two embedded-document goals |
| Integration owners unavailable | Contracts remain drafts | Track owner/questions and schedule implementation later; fixtures remain development-only |
| Developer verification finds significant defects | Feature capacity reduces | Use reserve; defer cosmetic extras and nonessential grid polish first, never authorization/history checks |

## 10. Manager demo / day-30 acceptance

1. Upload/register an eligible synthetic EW1 document; observe one automatic workflow start.
2. Open the authorized task, inspect multiple pages, verify fields and record a documentary outcome.
3. Return an incomplete document with reasons, correct it through the agreed route and show retained history.
4. Register a country-specific document through the agreed receipt/scanning boundary and demonstrate EW2.
5. Attempt a disallowed format or unauthorized action; confirm backend rejection/hold.
6. Repeat an intake event/request; confirm no duplicate active workflow.
7. Show separate complete/incomplete/rejected outcomes and the internal completion-event record.
8. Present HIVE/iCRM/OBS contract drafts and unresolved decisions, with future connector work clearly identified.

**Day-30 promise:** a working development increment for both document workflows with evidenced limits. Coverage of every stated combination depends on approved rule matrices and available platform prerequisites; production integration, QA/UAT and release approval are follow-on work.
