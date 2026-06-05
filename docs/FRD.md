# Functional Requirements Document

| Field | Detail |
|-------|--------|
| **Document Title** | Functional Requirements Document |
| **Project Name** | AI-Assisted Delivery Failure and Reattempt Coordination System |
| **Version** | 1.0 |
| **Prepared by** | Business Analysis Team |
| **Date** | 05 June 2026 |
| **Status** | Draft |

---

## Section 1 — Purpose and Scope

This document defines the functional and non-functional requirements for the AI-Assisted Delivery Failure and Reattempt Coordination System (hereafter referred to as "the System"), a web-based enterprise application designed to streamline the end-to-end management of failed delivery cases for an online retail organisation. The System addresses a recurring operational pain point whereby undelivered orders are tracked and coordinated manually through fragmented channels including spreadsheets, email threads, courier portals, and customer service notes. This manual approach leads to inconsistent follow-up, reattempt delays, missed escalations, and poor customer experience. By providing a centralised, structured, and AI-augmented workflow, the System enables faster resolution of delivery failures and more consistent handling across all back-office roles.

The primary users of the System are Back-Office Operations Analysts, Customer Support Executives, and Delivery Coordination Associates. These roles interact with the System to create and manage delivery failure cases, validate reattempt eligibility, coordinate with courier partners and warehouse teams, communicate with customers, and close resolved cases. The System is intended for use within the organisation's internal operations network and is accessed via a web browser. All case data is persisted in a relational database, and an integrated large language model (LLM) assists users with case summarisation, message drafting, and next-action recommendations.

**In Scope:** Creation and lifecycle management of delivery failure cases; capture of order, customer, courier, and failure details; classification of failure reasons; reattempt checklist validation; exception identification; follow-up task creation; LLM-assisted summary and message generation; case status management; audit trail maintenance; and role-based access to all screens described in this document. **Out of Scope:** Direct integration with live courier partner APIs or order management systems (data entry is manual or via sample datasets in this phase); customer self-service portal; payment or refund processing; mobile native application; and real-time GPS tracking of delivery personnel.

---

## Section 2 — Stakeholders and Roles

| Role | Description | Primary Responsibilities in this System |
|------|-------------|------------------------------------------|
| **Back-Office Operations Analyst** | An internal operations team member responsible for overseeing the overall delivery failure resolution process and ensuring cases are progressed to closure. | Create delivery failure cases; capture and validate all order and courier details; run checklist validation; review and approve AI-generated summaries; assign follow-up tasks; update case status; maintain audit trail oversight. |
| **Customer Support Executive** | A front-line support team member who handles inbound and outbound customer communication related to failed deliveries. | View assigned cases; review AI-drafted customer messages; send or finalise customer-facing communications; update case notes with customer responses; escalate cases requiring reattempt or return-to-warehouse action. |
| **Delivery Coordination Associate** | A logistics-focused team member responsible for coordinating reattempt schedules and communicating with courier partners and warehouse teams. | Review courier remarks and exception flags; generate and send internal courier or warehouse follow-up notes; schedule reattempt actions; update reattempt status; flag cases for return-to-warehouse processing. |

---

## Section 3 — Functional Requirements

### FR-01: Create Delivery Failure Case

| Field | Detail |
|-------|--------|
| **ID** | FR-01 |
| **Name** | Create Delivery Failure Case |
| **Description** | The System must allow authorised users to create a new delivery failure case for any online retail order where the first or subsequent delivery attempt was unsuccessful. The case creation form must accept all mandatory order and customer details and assign a unique case ID upon submission. A new case must default to the status "New" on creation. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive |
| **Trigger / Precondition** | User is authenticated and navigates to the Case Creation Form. A valid order number is available. |
| **Expected Outcome** | A new delivery failure case record is created in the database with a unique Case ID, all captured fields, default status "New", and a timestamp. The case appears on the Case Dashboard. |

**Acceptance Criteria:**
- AC1: When a user submits the case creation form with all mandatory fields populated, the System creates a case record and displays a confirmation message containing the generated Case ID within 3 seconds.
- AC2: If any mandatory field (Order Number, Customer Name, Customer Contact, Delivery Address, Courier Partner, Delivery Attempt Date, Failure Reason, Assigned Owner) is left blank, the System must display an inline field-level validation error and prevent form submission.
- AC3: The newly created case must appear on the Case Dashboard with status "New" and the correct creation timestamp, visible to all authorised roles without requiring a page refresh.

---

### FR-02: Capture Order, Customer, Courier, and Failure Details

| Field | Detail |
|-------|--------|
| **ID** | FR-02 |
| **Name** | Capture Order, Customer, Courier, and Failure Details |
| **Description** | The System must provide a structured form to capture the full set of case details including: Order Number, Customer Name, Customer Contact (phone and/or email), Delivery Address (with pincode), Product Category, Courier Partner, Delivery Attempt Date, Failure Reason, Courier Remarks, and Assigned Back-Office Owner. All captured data must be stored persistently and be editable by authorised users prior to case closure. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Trigger / Precondition** | A delivery failure case has been created (FR-01) or an existing case is opened for editing. |
| **Expected Outcome** | All entered details are saved to the case record and displayed accurately in the Case Detail View. Field-level edits are logged in the audit trail. |

**Acceptance Criteria:**
- AC1: The form must display all specified fields with appropriate input types (text, date picker, dropdown, text area), and all mandatory fields must be visually marked with an asterisk (*).
- AC2: On saving, all field values must be persisted to the database and retrievable without data loss when the case is reopened.
- AC3: Any update to a previously saved field must create an audit trail entry recording the field name, old value, new value, user, and timestamp.

---

### FR-03: Classify Failure Reason

| Field | Detail |
|-------|--------|
| **ID** | FR-03 |
| **Name** | Classify Failure Reason |
| **Description** | The System must provide a controlled dropdown for classifying the reason for delivery failure into exactly eight predefined categories. The selected failure reason must be stored with the case and used as input to the LLM summarisation and message drafting functions. The failure reason must be selectable at case creation and editable thereafter until the case is Closed. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive |
| **Trigger / Precondition** | Case creation form is active, or case is in an editable status (not Closed). |
| **Expected Outcome** | The selected failure reason category is saved to the case record and displayed in the Case Detail View, Case Dashboard, and as a tagged label in the AI Panel input. |

**Acceptance Criteria:**
- AC1: The failure reason dropdown must contain exactly the following eight values in order: "Customer Unavailable", "Phone Not Reachable", "Incorrect Address", "Incomplete Address", "Customer Refused Delivery", "Package Damaged in Transit", "Courier Delay", "Return to Warehouse Required".
- AC2: The System must not allow case submission without a failure reason selected; a validation error must be displayed if the field is empty.
- AC3: Changing the failure reason on an existing case must trigger an audit trail entry and must not automatically regenerate any previously saved AI output unless the user explicitly requests regeneration.

---

### FR-04: Validate Reattempt Checklist

| Field | Detail |
|-------|--------|
| **ID** | FR-04 |
| **Name** | Validate Reattempt Checklist |
| **Description** | The System must present a five-point reattempt eligibility checklist within the Case Detail View that users must validate before a reattempt action can be scheduled. Each checklist item must be individually confirmable (checkbox), and the overall checklist status must reflect whether all, some, or none of the points are confirmed. The checklist result must be saved and included in the audit trail. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Delivery Coordination Associate |
| **Trigger / Precondition** | A delivery failure case exists with status "New" or "Pending Courier". |
| **Expected Outcome** | The checklist validation result (pass/partial/fail) is saved to the case record. If all five points are confirmed, the case becomes eligible for reattempt scheduling. If any point is unconfirmed, the System flags the case with the relevant exception. |

**Acceptance Criteria:**
- AC1: The checklist must contain exactly five items: "Customer Contact Available", "Address Completeness Checked", "Courier Remarks Captured", "Reattempt Eligibility Confirmed", and "Next Action Selected". Each must render as a checkbox with a label.
- AC2: The "Schedule Reattempt" action button must be disabled unless all five checklist items are checked; an inline tooltip must explain why the button is disabled.
- AC3: The completed checklist state (all checked items with timestamp and user) must be saved to the audit trail when the user clicks "Save Checklist".

---

### FR-05: Identify Exceptions and Blockers

| Field | Detail |
|-------|--------|
| **ID** | FR-05 |
| **Name** | Identify Exceptions and Blockers |
| **Description** | The System must automatically evaluate the case data and checklist results to surface applicable exception flags that indicate a blocker preventing reattempt. Exception flags must be displayed as a labelled warning banner or tag within the Case Detail View. Each active exception must link to the relevant field or checklist item that requires resolution. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Trigger / Precondition** | The user saves the checklist or attempts to schedule a reattempt. |
| **Expected Outcome** | All applicable exceptions are displayed in the Exceptions panel. The case cannot progress to "Reattempt Scheduled" while unresolved exceptions remain active. |

**Acceptance Criteria:**
- AC1: The System must evaluate and display flags for the following exception conditions: "Missing Customer Contact", "Incomplete Address", "Unclear Courier Remarks", "Reattempt Limit Reached", "Damaged Package Photo Not Available", and "Customer Confirmation Pending".
- AC2: Each exception flag must include a short description and a "Resolve" or "Mark as Acknowledged" action that records the resolution in the audit trail.
- AC3: If no exceptions are detected, the Exceptions panel must display the message "No active exceptions — case is eligible for reattempt."

---

### FR-06: Create Follow-Up Task

| Field | Detail |
|-------|--------|
| **ID** | FR-06 |
| **Name** | Create Follow-Up Task |
| **Description** | The System must allow authorised users to create one or more follow-up tasks associated with a delivery failure case. Each task must capture a task description, priority level, assigned owner, due date, and required action type. Tasks must be viewable in the Follow-Up Task Panel within the Case Detail View and must update independently of the overall case status. |
| **Priority** | Medium |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Trigger / Precondition** | A delivery failure case exists in any status other than "Closed". |
| **Expected Outcome** | The follow-up task is saved to the database, displayed in the Follow-Up Task Panel, and visible to all users with access to the case. Overdue tasks must be highlighted. |

**Acceptance Criteria:**
- AC1: The task creation form must capture: Task Description (free text, max 500 characters), Priority (High / Medium / Low dropdown), Assigned Owner (user lookup), Due Date (date picker, must not be in the past), and Required Action (dropdown: "Contact Customer", "Contact Courier", "Verify Address", "Schedule Reattempt", "Escalate to Manager", "Return to Warehouse", "Other").
- AC2: Tasks with a due date that has passed and status not "Completed" must display a red "Overdue" badge in the task panel.
- AC3: Completing a task must require the user to enter a resolution note before marking it "Completed", and this note must be saved to the audit trail.

---

### FR-07: Generate AI-Assisted Delivery Failure Summary

| Field | Detail |
|-------|--------|
| **ID** | FR-07 |
| **Name** | Generate AI-Assisted Delivery Failure Summary |
| **Description** | The System must provide an LLM-powered action that generates a structured, human-readable summary of the delivery failure case. The summary must be based on the case's captured fields, failure reason, courier remarks, checklist results, and current status. The generated summary must be displayed in the AI Panel within the Case Detail View and must be editable before saving. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst |
| **Trigger / Precondition** | A delivery failure case exists with at minimum: Order Number, Failure Reason, Courier Remarks, and Delivery Attempt Date populated. |
| **Expected Outcome** | The LLM returns a formatted case summary within 10 seconds. The summary is displayed in a review pane, editable by the user, and can be saved to the audit trail. |

**Acceptance Criteria:**
- AC1: Clicking "Generate Summary" must send the full case context to the LLM and display the result in the AI Panel within 10 seconds under normal load conditions.
- AC2: The generated summary must include at minimum the following labelled sections: Case Overview, Failure Reason, Courier Remarks, Checklist Status, Exceptions (if any), and Recommended Next Action.
- AC3: The user must be able to edit the generated text inline within the AI Panel, and clicking "Save to Audit Trail" must persist the final (edited or unedited) version with a "AI-Generated" tag and the user's name and timestamp.

---

### FR-08: Draft Customer-Facing Message

| Field | Detail |
|-------|--------|
| **ID** | FR-08 |
| **Name** | Draft Customer-Facing Message |
| **Description** | The System must allow the LLM to draft a polite, professional customer-facing message based on the case details and failure reason. The message must be appropriate to the specific failure scenario (e.g., requesting address confirmation, phone availability, preferred slot, or reattempt confirmation). The drafted message must be displayed in the AI Panel, editable, and copyable for use by the Customer Support Executive. |
| **Priority** | High |
| **Roles Involved** | Customer Support Executive, Back-Office Operations Analyst |
| **Trigger / Precondition** | A delivery failure case exists with Failure Reason and Customer Contact fields populated. |
| **Expected Outcome** | A context-appropriate customer message is generated and displayed for review. The finalised message is logged in the audit trail with the user's name and timestamp. |

**Acceptance Criteria:**
- AC1: The drafted message must address the customer by name, reference the order number, state the delivery failure reason in polite language, and include a clear call-to-action relevant to the failure reason (e.g., "Please confirm your availability" or "Please verify your delivery address").
- AC2: The message must not include any internal operational jargon, internal case IDs, or courier remarks intended for internal use only.
- AC3: A "Copy to Clipboard" button must be available alongside the message draft, and clicking "Save Draft" must log the message content, generation timestamp, and user to the audit trail.

---

### FR-09: Draft Internal Courier or Warehouse Follow-Up Note

| Field | Detail |
|-------|--------|
| **ID** | FR-09 |
| **Name** | Draft Internal Courier or Warehouse Follow-Up Note |
| **Description** | The System must provide an LLM-assisted function to generate an internal follow-up note addressed to the courier partner, warehouse team, or customer support team. The note must include structured operational details such as case ID, order number, failure reason, reattempt instructions, or return-to-warehouse directions. The note must be editable and saveable to the audit trail. |
| **Priority** | High |
| **Roles Involved** | Delivery Coordination Associate, Back-Office Operations Analyst |
| **Trigger / Precondition** | A delivery failure case exists; the note recipient type (Courier Partner / Warehouse / Customer Support) is selected by the user. |
| **Expected Outcome** | A structured internal note is generated, displayed in the AI Panel, and optionally saved to the audit trail after user review and edit. |

**Acceptance Criteria:**
- AC1: The user must be able to select the note recipient from a dropdown ("Courier Partner", "Warehouse Team", "Customer Support Team") before generating the note, and the generated content must be tailored to the selected recipient.
- AC2: The internal note must include: Case ID, Order Number, Customer Name (first name only for privacy), Failure Reason, Courier Remarks, Reattempt or RTW instruction, and a standard closing line.
- AC3: Saving the internal note must create an audit trail entry tagged "Internal Note – [Recipient Type]" with the user's name, timestamp, and full note content.

---

### FR-10: Update Case Status

| Field | Detail |
|-------|--------|
| **ID** | FR-10 |
| **Name** | Update Case Status |
| **Description** | The System must allow authorised users to update the status of a delivery failure case through a defined set of eight status values. Status transitions must follow a logical workflow and certain transitions must be gated by prerequisite conditions (e.g., a case cannot move to "Reattempt Scheduled" if the reattempt checklist is incomplete). Each status change must be recorded in the audit trail. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Trigger / Precondition** | A delivery failure case exists and the user has the appropriate role permission to update status. |
| **Expected Outcome** | The case status is updated in the database, reflected immediately on the Case Dashboard and Case Detail View, and the status change is logged in the audit trail with the user's name, old status, new status, and timestamp. |

**Acceptance Criteria:**
- AC1: The status dropdown must contain exactly the following eight values: "New", "Pending Customer", "Pending Courier", "Reattempt Scheduled", "Reattempt Completed", "Returned to Warehouse", "Resolved", "Closed". The current status must be pre-selected and the current value must be excluded from selectable options.
- AC2: The System must enforce these transition rules: "Reattempt Scheduled" requires all five checklist items confirmed; "Closed" requires status to be either "Resolved" or "Returned to Warehouse" first; "Returned to Warehouse" requires Failure Reason to be "Return to Warehouse Required" or an active "Reattempt Limit Reached" exception.
- AC3: Every status change must generate an audit trail entry within 1 second of saving, capturing: Case ID, Previous Status, New Status, Changed By, Change Timestamp, and an optional comment field.

---

### FR-11: Maintain Audit Trail

| Field | Detail |
|-------|--------|
| **ID** | FR-11 |
| **Name** | Maintain Audit Trail |
| **Description** | The System must automatically record a timestamped audit trail entry for every significant action performed on a delivery failure case. The audit trail must be immutable (entries cannot be edited or deleted by any user) and must be viewable in the Audit Trail Panel within the Case Detail View. The audit trail must capture the actor, action type, affected field or component, and a brief description for each entry. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst (view); all roles (generate entries) |
| **Trigger / Precondition** | Any of the following actions occur: case created, field edited, checklist saved, exception flagged or resolved, follow-up task created or updated, AI output generated or saved, status changed, message drafted or saved. |
| **Expected Outcome** | A complete, chronological, uneditable log of all case actions is available in the Audit Trail Panel, sortable by date and filterable by action type. |

**Acceptance Criteria:**
- AC1: The audit trail must capture entries for at minimum: Case Created, Field Updated (per field), Checklist Saved, Exception Flagged, Exception Resolved, Task Created, Task Completed, AI Summary Generated, AI Summary Saved, Customer Message Drafted, Customer Message Saved, Internal Note Drafted, Internal Note Saved, Status Changed.
- AC2: No user interface element or API endpoint must permit editing or deletion of audit trail entries; the Audit Trail Panel must not display any edit or delete controls.
- AC3: The Audit Trail Panel must support filtering by Action Type and sorting by Timestamp (ascending and descending), and must display a minimum of 50 entries per page with pagination.

---

### FR-12: LLM-Assisted Actions

| Field | Detail |
|-------|--------|
| **ID** | FR-12 |
| **Name** | LLM-Assisted Actions (Summarise, Draft, Suggest, Convert) |
| **Description** | The System must integrate with an LLM to support four distinct AI-assisted actions available within the Case Detail View AI Panel: (1) Summarise the case, (2) Draft a customer or internal message, (3) Suggest the recommended next action, and (4) Convert customer or courier response notes into structured follow-up comments. Each action must be explicitly triggered by the user and must not run automatically without user intent. |
| **Priority** | High |
| **Roles Involved** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Trigger / Precondition** | Case Detail View is open with sufficient case data. User clicks one of the four LLM action buttons. |
| **Expected Outcome** | The LLM returns a relevant, context-specific output for the selected action type within 10 seconds. The output is displayed in the AI Panel, editable by the user, and saveable to the audit trail. |

**Acceptance Criteria:**
- AC1: The AI Panel must display four distinctly labelled action buttons: "Generate Summary", "Draft Customer Message", "Draft Internal Note", and "Suggest Next Action". A fifth button "Convert Response to Comment" must appear when a courier or customer response note is present in the case.
- AC2: The "Suggest Next Action" output must present one primary recommended action from this list: "Request Customer Confirmation", "Schedule Reattempt", "Route to Courier Team", "Mark as Return to Warehouse", "Close Case" — along with a one-sentence rationale based on the current case state.
- AC3: All LLM action outputs must include a visible disclaimer: "AI-generated content — please review before saving" displayed beneath the output panel, and outputs must only be logged to the audit trail upon explicit user action ("Save to Audit Trail" button click).

---

## Section 4 — Non-Functional Requirements

### NFR-01: Performance

| Field | Detail |
|-------|--------|
| **ID** | NFR-01 |
| **Category** | Performance |
| **Description** | The System must load all primary screens and respond to user interactions within defined time thresholds under normal and peak load conditions. LLM-assisted actions are subject to a separate, higher latency tolerance given external API dependencies. |
| **Measurement Criteria** | Page load time (Case Dashboard, Case Detail View) ≤ 2 seconds for up to 500 concurrent users. Form submission response ≤ 1 second. LLM response (summary, draft, suggestion) ≤ 10 seconds at p95. Database read queries ≤ 500ms. Status update write operations ≤ 1 second. |
| **Acceptance Criteria** | Load testing with 500 simulated concurrent users must show that 95% of non-LLM page requests complete within 2 seconds and 99% within 5 seconds; LLM actions must complete within 10 seconds for 90% of requests under normal API availability. |

---

### NFR-02: Usability

| Field | Detail |
|-------|--------|
| **ID** | NFR-02 |
| **Category** | Usability |
| **Description** | The System must present a clean, intuitive interface that allows new users to complete a full case creation and reattempt workflow without external training documentation. The interface must use consistent design patterns, clear labels, inline help text, and contextual validation messages throughout. |
| **Measurement Criteria** | A new user must be able to create a case, validate the checklist, generate an AI summary, and update case status within 10 minutes of first use. System Usability Scale (SUS) score ≥ 75. All form fields must include placeholder text or tooltip guidance. |
| **Acceptance Criteria** | Usability testing with 5 representative users must yield a mean SUS score ≥ 75; task completion rate for the core workflow (case create → checklist → AI summary → status update) must be ≥ 90% without facilitator assistance. |

---

### NFR-03: Reliability

| Field | Detail |
|-------|--------|
| **ID** | NFR-03 |
| **Category** | Reliability |
| **Description** | The System must maintain high availability during standard business hours and must handle partial failures (e.g., LLM API unavailability) gracefully without causing data loss or application crashes. Core case management functions must remain operational even when the LLM integration is unavailable. |
| **Measurement Criteria** | System uptime ≥ 99.5% during business hours (8 AM – 8 PM local time). Mean Time Between Failures (MTBF) ≥ 720 hours. LLM API failure must degrade gracefully: AI Panel must display "AI service temporarily unavailable — please try again shortly" without disrupting other case management functions. |
| **Acceptance Criteria** | Uptime monitoring over a 30-day period must confirm ≥ 99.5% availability. Simulated LLM API failure test must confirm that all non-AI case management functions (create, edit, status update, audit trail) remain fully operational. |

---

### NFR-04: Security

| Field | Detail |
|-------|--------|
| **ID** | NFR-04 |
| **Category** | Security |
| **Description** | The System must enforce role-based access control (RBAC) ensuring that users can only access screens and perform actions appropriate to their assigned role. All data in transit must be encrypted, and customer personally identifiable information (PII) must be protected at rest. The System must maintain a session timeout policy and prevent unauthorised access. |
| **Measurement Criteria** | All HTTP traffic must use TLS 1.2 or higher. PII fields (Customer Contact, Delivery Address) must be encrypted at rest using AES-256. Session timeout after 15 minutes of inactivity. All authentication attempts (success and failure) must be logged. Role-based screen visibility must be enforced at the API layer, not only at the UI layer. |
| **Acceptance Criteria** | Penetration testing must reveal no Critical or High severity vulnerabilities related to authentication, authorisation, or data exposure. RBAC enforcement must be verified by attempting cross-role API calls and confirming 403 Forbidden responses. |

---

### NFR-05: Scalability

| Field | Detail |
|-------|--------|
| **ID** | NFR-05 |
| **Category** | Scalability |
| **Description** | The System must be designed to scale horizontally to support growing case volumes and increasing numbers of concurrent users without requiring architectural changes. The database schema must accommodate growth in case records without performance degradation. |
| **Measurement Criteria** | The System must support at least 10,000 active cases in the database without query performance degradation beyond the thresholds in NFR-01. The application tier must support horizontal scaling to at least 4 instances. Database indexes must be defined on all primary filter and sort columns (Case ID, Status, Assigned Owner, Attempt Date). |
| **Acceptance Criteria** | Load testing with a 10,000-record dataset and 500 concurrent users must confirm that p95 response times remain within the thresholds defined in NFR-01. Adding a second application server instance must not require code changes or downtime. |

---

### NFR-06: Accessibility

| Field | Detail |
|-------|--------|
| **ID** | NFR-06 |
| **Category** | Accessibility |
| **Description** | The System must conform to WCAG 2.1 Level AA accessibility standards to ensure it is usable by individuals with visual, motor, or cognitive disabilities. All interactive elements must be keyboard-navigable, screen-reader-compatible, and must meet minimum contrast ratios. |
| **Measurement Criteria** | WCAG 2.1 AA compliance across all five screens. Minimum contrast ratio of 4.5:1 for normal text and 3:1 for large text. All form fields must have programmatically associated labels. All interactive controls must be operable via keyboard alone. No content must rely solely on colour to convey meaning. |
| **Acceptance Criteria** | Automated accessibility audit using an industry-standard tool (e.g., axe or Lighthouse) must return zero Critical or Serious violations across all five primary screens. Manual keyboard navigation test must confirm all interactive elements are reachable and operable without a mouse. |

---

## Section 5 — User Stories with Acceptance Criteria

### Group 1: Back-Office Operations Analyst

---

### US-01: Create a New Delivery Failure Case

| Field | Detail |
|-------|--------|
| **ID** | US-01 |
| **Role** | Back-Office Operations Analyst |
| **Story** | As a Back-Office Operations Analyst, I want to create a new delivery failure case by entering all relevant order and customer details so that the failed delivery is formally tracked and assigned for resolution. |
| **Related FR** | FR-01, FR-02 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I am on the Case Creation Form, When I fill in all mandatory fields and click "Create Case", Then a new case record is created with a unique Case ID, status "New", and a success notification is displayed.
- AC2: Given I have left the "Order Number" field blank, When I click "Create Case", Then an inline validation error "Order Number is required" appears and the form is not submitted.
- AC3: Given a case is successfully created, When I navigate to the Case Dashboard, Then the new case appears at the top of the list with status "New" and the correct creation timestamp.

---

### US-02: Classify the Failure Reason

| Field | Detail |
|-------|--------|
| **ID** | US-02 |
| **Role** | Back-Office Operations Analyst |
| **Story** | As a Back-Office Operations Analyst, I want to classify the failure reason from a predefined list so that the case is accurately categorised for the right resolution path. |
| **Related FR** | FR-03 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I am on the Case Creation Form, When I open the "Failure Reason" dropdown, Then I see exactly eight options in the defined order.
- AC2: Given I select "Customer Unavailable" and save the case, When I open the Case Detail View, Then the Failure Reason displays as "Customer Unavailable" and is reflected in the AI Panel context.
- AC3: Given the case is not yet Closed, When I change the Failure Reason to "Incorrect Address" and save, Then an audit trail entry records the field change with old and new values.

---

### US-03: Validate the Reattempt Checklist

| Field | Detail |
|-------|--------|
| **ID** | US-03 |
| **Role** | Back-Office Operations Analyst |
| **Story** | As a Back-Office Operations Analyst, I want to validate the five-point reattempt checklist so that I can confirm the case meets eligibility criteria before scheduling a reattempt. |
| **Related FR** | FR-04, FR-05 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I am on the Case Detail View, When I check all five checklist items and click "Save Checklist", Then the checklist status updates to "Complete" and the "Schedule Reattempt" button becomes active.
- AC2: Given I have not checked "Courier Remarks Captured", When I attempt to click "Schedule Reattempt", Then the button remains disabled and a tooltip reads "Complete checklist to enable reattempt scheduling".
- AC3: Given I save a partially completed checklist, When I view the Audit Trail Panel, Then I see an entry for "Checklist Saved" showing which items were checked and the timestamp.

---

### US-04: Generate AI-Assisted Case Summary

| Field | Detail |
|-------|--------|
| **ID** | US-04 |
| **Role** | Back-Office Operations Analyst |
| **Story** | As a Back-Office Operations Analyst, I want to generate an AI-assisted case summary so that I can quickly review the full case situation in plain language and share it with stakeholders. |
| **Related FR** | FR-07, FR-12 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given the case has at minimum Order Number, Failure Reason, Courier Remarks, and Delivery Attempt Date, When I click "Generate Summary", Then the AI Panel displays a structured summary within 10 seconds.
- AC2: Given the summary is displayed, When I edit a section of the summary text and click "Save to Audit Trail", Then the saved version reflects my edits and is tagged "AI-Generated (Edited)" in the audit trail.
- AC3: Given the LLM service is unavailable, When I click "Generate Summary", Then the AI Panel displays "AI service temporarily unavailable — please try again shortly" and no error or crash occurs.

---

### US-05: Update Case Status

| Field | Detail |
|-------|--------|
| **ID** | US-05 |
| **Role** | Back-Office Operations Analyst |
| **Story** | As a Back-Office Operations Analyst, I want to update the status of a case as it progresses through the resolution workflow so that all team members have visibility of the current case state. |
| **Related FR** | FR-10, FR-11 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given the case checklist is fully validated, When I update the status to "Reattempt Scheduled" and click "Save Status", Then the status updates immediately on the Case Detail View and Case Dashboard.
- AC2: Given the case status is "New", When I attempt to set status to "Closed" directly, Then the System displays a validation message "Case must be Resolved or Returned to Warehouse before it can be Closed."
- AC3: Given any status change is saved, When I view the Audit Trail Panel, Then an entry shows the Previous Status, New Status, Changed By, and Timestamp.

---

### Group 2: Customer Support Executive

---

### US-06: View Assigned Cases

| Field | Detail |
|-------|--------|
| **ID** | US-06 |
| **Role** | Customer Support Executive |
| **Story** | As a Customer Support Executive, I want to view a list of cases assigned to me or my team so that I can prioritise and action customer follow-ups efficiently. |
| **Related FR** | FR-01, FR-10 |
| **Priority** | Medium |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I am logged in as a Customer Support Executive, When I open the Case Dashboard, Then I see only cases where my name is listed as Assigned Owner or the case status requires customer action.
- AC2: Given there are cases with status "Pending Customer" and status "New", When I apply the "Pending Customer" status filter, Then only cases with that status are displayed.
- AC3: Given no cases match my filter criteria, When the filter is applied, Then the dashboard displays the empty state message "No cases match the selected filters."

---

### US-07: Draft and Finalise Customer-Facing Message

| Field | Detail |
|-------|--------|
| **ID** | US-07 |
| **Role** | Customer Support Executive |
| **Story** | As a Customer Support Executive, I want to generate and review an AI-drafted customer message so that I can send accurate, professional communication to the customer in less time. |
| **Related FR** | FR-08, FR-12 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I am on the Case Detail View, When I click "Draft Customer Message", Then the AI Panel generates a context-appropriate message addressed to the customer by name within 10 seconds.
- AC2: Given the draft message is displayed, When I edit the message body and click "Save Draft", Then the finalised text is logged to the audit trail with my name and timestamp.
- AC3: Given the message is saved, When I click "Copy to Clipboard", Then the full message text is copied and a brief confirmation tooltip "Copied!" is displayed.

---

### US-08: Update Case with Customer Response

| Field | Detail |
|-------|--------|
| **ID** | US-08 |
| **Role** | Customer Support Executive |
| **Story** | As a Customer Support Executive, I want to log a customer's response and convert it into a structured follow-up comment so that the case record stays accurate and actionable. |
| **Related FR** | FR-12, FR-11 |
| **Priority** | Medium |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I paste or type a customer's raw response into the response notes field, When I click "Convert Response to Comment", Then the AI generates a structured summary of the response within 10 seconds.
- AC2: Given the structured comment is generated, When I click "Add to Case", Then the comment is appended to the case record and visible in the Case Detail View under "Customer Responses".
- AC3: Given the comment is added, When I view the Audit Trail Panel, Then an entry records "Customer Response Logged" with the user, timestamp, and a preview of the comment.

---

### US-09: Escalate a Case with an Exception

| Field | Detail |
|-------|--------|
| **ID** | US-09 |
| **Role** | Customer Support Executive |
| **Story** | As a Customer Support Executive, I want to identify and acknowledge active exceptions on a case so that I can escalate or resolve blockers before the case stalls. |
| **Related FR** | FR-05, FR-06 |
| **Priority** | Medium |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given a case has the exception "Missing Customer Contact" flagged, When I open the Case Detail View, Then the exception banner is displayed prominently above the checklist section.
- AC2: Given I resolve the exception by updating the Customer Contact field, When I click "Mark as Resolved" on the exception flag, Then the exception is removed from the active exceptions panel and an audit trail entry is created.
- AC3: Given I need to escalate, When I create a follow-up task with Required Action "Escalate to Manager" and Priority "High", Then the task appears in the Follow-Up Task Panel with a red priority badge.

---

### Group 3: Delivery Coordination Associate

---

### US-10: Draft Internal Courier or Warehouse Note

| Field | Detail |
|-------|--------|
| **ID** | US-10 |
| **Role** | Delivery Coordination Associate |
| **Story** | As a Delivery Coordination Associate, I want to generate an internal follow-up note for the courier partner or warehouse team so that reattempt instructions or return-to-warehouse directions are communicated clearly and consistently. |
| **Related FR** | FR-09, FR-12 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given I select "Courier Partner" from the note recipient dropdown and click "Draft Internal Note", Then the AI generates a structured note referencing the Case ID, Order Number, Failure Reason, and reattempt instructions within 10 seconds.
- AC2: Given the note is generated, When I edit the reattempt instruction line and click "Save to Audit Trail", Then the saved note is tagged "Internal Note – Courier Partner" in the audit trail with my name and timestamp.
- AC3: Given I select "Warehouse Team" as the recipient, When the note is generated, Then the note content includes return-to-warehouse directions and the standard RTW closing line, not reattempt instructions.

---

### US-11: Schedule a Reattempt

| Field | Detail |
|-------|--------|
| **ID** | US-11 |
| **Role** | Delivery Coordination Associate |
| **Story** | As a Delivery Coordination Associate, I want to schedule a delivery reattempt after validating the checklist so that the courier partner has a confirmed date and any customer-confirmed preferences are captured. |
| **Related FR** | FR-04, FR-10 |
| **Priority** | High |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given all five checklist items are confirmed and no active exceptions exist, When I click "Schedule Reattempt" and enter a reattempt date, Then the case status updates to "Reattempt Scheduled" and the reattempt date is saved to the case record.
- AC2: Given the reattempt date I enter is in the past, When I click "Confirm Reattempt", Then the System displays a validation error "Reattempt date must be today or a future date" and does not save.
- AC3: Given the reattempt is scheduled, When I view the Audit Trail Panel, Then an entry records "Reattempt Scheduled" with the selected date, user, and timestamp.

---

### US-12: Flag a Case for Return to Warehouse

| Field | Detail |
|-------|--------|
| **ID** | US-12 |
| **Role** | Delivery Coordination Associate |
| **Story** | As a Delivery Coordination Associate, I want to flag a case for return-to-warehouse processing so that undeliverable packages are formally routed back without delay. |
| **Related FR** | FR-10, FR-09, FR-05 |
| **Priority** | Medium |

**Acceptance Criteria (Given/When/Then):**
- AC1: Given the Failure Reason is "Return to Warehouse Required" or the exception "Reattempt Limit Reached" is active, When I update the status to "Returned to Warehouse" and confirm, Then the status updates and the case is locked from further reattempt scheduling.
- AC2: Given the case is flagged for RTW, When I click "Draft Internal Note" and select "Warehouse Team", Then the generated note includes the RTW directive and the case's product category and order number.
- AC3: Given the case status is "Returned to Warehouse", When I attempt to check a reattempt checklist item, Then the checklist is read-only and displays the message "Reattempt not available — case is marked for return to warehouse."

---

## Section 6 — Functional Screen Designs

### SCR-01: Case Dashboard

| Field | Detail |
|-------|--------|
| **Screen ID** | SCR-01 |
| **Screen Name** | Case Dashboard |
| **Purpose** | Provides an at-a-glance overview of all delivery failure cases, enabling users to monitor, filter, search, and navigate to individual cases. |
| **Accessible By** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Related User Stories** | US-01, US-05, US-06 |
| **Related FRs** | FR-01, FR-10, FR-11 |

**Layout Description:**
The Case Dashboard is structured as a full-width application screen with a fixed top navigation bar spanning the full width. The top bar contains the application logo on the left, the application name "Delivery Failure Coordination" in the centre-left, and a user profile icon with the logged-in user's name and role on the right alongside a sign-out link. Directly below the top bar is a secondary action bar containing a prominent "New Case" button on the left in a primary colour, a search input field in the centre accepting free-text search by order number or customer name, and a row of filter dropdowns on the right: Status (multi-select), Failure Reason (multi-select), Assigned Owner (single-select), and Attempt Date Range (date range picker). Below the action bar, a summary statistics row displays four KPI tiles in a horizontal strip: "Total Cases", "Pending Customer", "Reattempt Scheduled", and "Overdue Tasks" — each tile showing a count and a coloured indicator. The main content area below the KPI tiles contains the case list table. The table spans the full width of the screen and includes the following columns: Case ID, Order Number, Customer Name, Failure Reason, Assigned Owner, Attempt Date, Status (with colour-coded badge), Last Updated, and an Actions column with "View" and quick "Update Status" icons. Rows are sorted by Last Updated descending by default. Clicking anywhere on a row navigates to the Case Detail View. Column headers for Case ID, Attempt Date, Status, and Last Updated are sortable by click. A pagination control sits below the table, showing the current page, total pages, total record count, and allowing the user to select rows per page (25, 50, 100).

**Primary Actions:**
- Click "New Case" to open Case Creation Form
- Click a table row or "View" icon to open Case Detail View
- Apply filters to narrow the case list
- Search by order number or customer name
- Sort table columns by clicking column headers
- Change page or rows-per-page using pagination controls
- Quick-update case status via the status icon in the Actions column

**Key Fields and Components:**

| Field / Component | Type | Validation Rules | Default Value | Notes |
|-------------------|------|-----------------|---------------|-------|
| New Case Button | Button (Primary) | — | — | Opens Case Creation Form |
| Search Input | Text Input | Max 100 chars | Empty | Searches Case ID, Order Number, Customer Name |
| Status Filter | Multi-select Dropdown | — | All statuses | See Case Status Values below |
| Failure Reason Filter | Multi-select Dropdown | — | All reasons | See FR-03 categories |
| Assigned Owner Filter | Single-select Dropdown | — | All owners | Populated from user directory |
| Attempt Date Range | Date Range Picker | Start ≤ End | Last 30 days | Format: DD MMM YYYY |
| Case ID Column | Text (sortable) | — | — | System-generated, e.g. DFC-20260001 |
| Status Badge | Colour-coded label | — | — | See status colours below |
| View Icon | Icon Button | — | — | Navigates to SCR-03 |
| Update Status Icon | Icon Button | — | — | Opens inline status update modal |
| Pagination | Control bar | — | Page 1, 25 rows | Total count displayed |

**Failure Reason Categories (Dropdown Values):**
1. Customer Unavailable
2. Phone Not Reachable
3. Incorrect Address
4. Incomplete Address
5. Customer Refused Delivery
6. Package Damaged in Transit
7. Courier Delay
8. Return to Warehouse Required

**Case Status Values with Colour Coding:**

| Status | Colour |
|--------|--------|
| New | Grey |
| Pending Customer | Amber |
| Pending Courier | Orange |
| Reattempt Scheduled | Blue |
| Reattempt Completed | Teal |
| Returned to Warehouse | Purple |
| Resolved | Green |
| Closed | Dark Grey |

**Default State:** The dashboard loads showing all cases in the database sorted by Last Updated descending, with no filters applied and the search field empty. KPI tiles reflect counts across all cases.

**Loaded State:** Table rows are populated with case data; status badges are colour-coded; KPI tile counts reflect current database state.

**Empty State:** When no cases exist or all cases are filtered out, the table area displays a centred illustration and the message: "No cases found. Try adjusting your filters or create a new case." with a "New Case" call-to-action button.

**Error States:**
- Network error fetching cases: Banner displayed "Unable to load cases. Please refresh the page."
- Filter returns zero results: Empty state message as above
- Status update fails: Inline toast notification "Status update failed. Please try again."

**Navigation Flows:**
- From this screen → Case Creation Form (SCR-02) when "New Case" is clicked
- From this screen → Case Detail View (SCR-03) when a case row or "View" icon is clicked

---

### SCR-02: Case Creation Form

| Field | Detail |
|-------|--------|
| **Screen ID** | SCR-02 |
| **Screen Name** | Case Creation Form |
| **Purpose** | Provides a structured form for creating a new delivery failure case by capturing all required order, customer, courier, and failure details. |
| **Accessible By** | Back-Office Operations Analyst, Customer Support Executive |
| **Related User Stories** | US-01, US-02 |
| **Related FRs** | FR-01, FR-02, FR-03 |

**Layout Description:**
The Case Creation Form opens as a full-page view, replacing the Case Dashboard, with a breadcrumb trail at the top reading "Case Dashboard > New Case". The form is presented as a single-column layout centred on the screen with a maximum width of 800px, providing a focused data-entry experience. The form is divided into three visually distinct sections separated by light horizontal dividers and section headings: "Order Details", "Customer & Delivery Details", and "Failure & Assignment Details". The "Order Details" section contains three fields in a two-column grid: Order Number (left), Product Category (right), and Delivery Attempt Date (left, spanning one column with a date picker). The "Customer & Delivery Details" section contains: Customer Name, Customer Phone, Customer Email (optional), and Delivery Address as a full-width multi-line text area, with a Pincode field to the right of the address block. The "Failure & Assignment Details" section contains: Courier Partner (dropdown), Failure Reason (dropdown — mandatory), Courier Remarks (full-width text area, max 1000 characters, with a character counter), and Assigned Back-Office Owner (searchable user dropdown). At the bottom of the form, a full-width sticky footer contains two action buttons: a "Cancel" button (secondary/outline style) on the left that returns to the Case Dashboard with a discard confirmation dialog, and a "Create Case" button (primary style) on the right. Mandatory fields are marked with a red asterisk. An info banner at the top of the form reads: "All fields marked * are mandatory. The case will be assigned status 'New' upon creation."

**Primary Actions:**
- Enter data into all form fields
- Select Failure Reason from dropdown
- Select Courier Partner from dropdown
- Select Assigned Owner from user lookup dropdown
- Click "Create Case" to submit and create the record
- Click "Cancel" to discard and return to Case Dashboard
- Clear individual fields using field-level clear icons

**Key Fields and Components:**

| Field / Component | Type | Validation Rules | Default Value | Notes |
|-------------------|------|-----------------|---------------|-------|
| Order Number * | Text Input | Alphanumeric, max 20 chars, unique | Empty | e.g. ORD-2026-00123 |
| Product Category | Dropdown | — | Select... | Electronics, Apparel, FMCG, Furniture, Other |
| Delivery Attempt Date * | Date Picker | Cannot be future date | Today | Format: DD MMM YYYY |
| Customer Name * | Text Input | Max 100 chars | Empty | — |
| Customer Phone * | Text Input | 10-digit numeric | Empty | Validated format |
| Customer Email | Text Input | Valid email format | Empty | Optional |
| Delivery Address * | Text Area | Max 300 chars | Empty | — |
| Pincode * | Text Input | 6-digit numeric | Empty | — |
| Courier Partner * | Dropdown | Must select | Select... | Populated from courier master data |
| Failure Reason * | Dropdown | Must select | Select... | 8 categories per FR-03 |
| Courier Remarks | Text Area | Max 1000 chars | Empty | Character counter displayed |
| Assigned Owner * | Searchable Dropdown | Must select | Current user | Populated from user directory |
| Create Case Button | Button (Primary) | All mandatory fields valid | — | Submits form |
| Cancel Button | Button (Secondary) | — | — | Discard dialog shown |

**Default State:** All fields empty; mandatory fields marked with asterisk; dropdowns showing "Select..." placeholder; "Create Case" button disabled until all mandatory fields are valid.

**Loaded State:** N/A — this is a creation form only.

**Empty State:** N/A

**Error States:**
- Mandatory field empty on submit: Inline field-level error "This field is required" in red beneath the field
- Invalid phone format: "Please enter a valid 10-digit phone number"
- Duplicate Order Number: "A case for this order number already exists. View existing case?" with a link
- API/save error: Toast notification "Case creation failed. Please try again or contact support."

**Navigation Flows:**
- From this screen → Case Dashboard (SCR-01) on successful case creation
- From this screen → Case Dashboard (SCR-01) on "Cancel" after discard confirmation

---

### SCR-03: Case Detail View with AI Panel

| Field | Detail |
|-------|--------|
| **Screen ID** | SCR-03 |
| **Screen Name** | Case Detail View with AI Panel |
| **Purpose** | Displays the full details of a selected delivery failure case and provides the primary workspace for validation, status management, exception review, and AI-assisted content generation. |
| **Accessible By** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Related User Stories** | US-02, US-03, US-04, US-05, US-07, US-08, US-09, US-10, US-11, US-12 |
| **Related FRs** | FR-02, FR-03, FR-04, FR-05, FR-07, FR-08, FR-09, FR-10, FR-11, FR-12 |

**Layout Description:**
The Case Detail View uses a two-column layout with a left content area (approximately 65% width) and a right AI Panel (approximately 35% width) separated by a subtle vertical divider. A breadcrumb trail at the top reads "Case Dashboard > Case ID: DFC-20260001". The left column is further divided into vertically stacked sections. At the top is the Case Header bar displaying the Case ID, current status badge, creation date, and an "Edit Case Details" icon button. Below the header, the Case Information section renders all captured fields (Order Number, Customer Name, Contact, Address, Product Category, Courier Partner, Attempt Date, Failure Reason, Courier Remarks, Assigned Owner) in a two-column read-only grid with an inline "Edit" pencil icon per field group. Below the Case Information section is the Reattempt Checklist section with the five checkbox items and a "Save Checklist" button. Below the checklist is the Exceptions Panel with a list of active and resolved exception flags. Below exceptions, the Follow-Up Tasks summary shows the count of open tasks with a "View All Tasks" link that expands the full Follow-Up Task Panel (SCR-04). At the very bottom of the left column, the Status Update control shows a labelled dropdown and a "Save Status" button. The right AI Panel has a sticky header showing "AI Assistant" with a small model indicator. Below the header are four action buttons stacked vertically: "Generate Summary", "Draft Customer Message", "Draft Internal Note", and "Suggest Next Action". A fifth button "Convert Response to Comment" appears when a customer/courier response note is present. Below the action buttons is the output text area where AI-generated content appears. Beneath the output area are three controls: "Save to Audit Trail", "Copy to Clipboard", and "Regenerate". A disclaimer reads: "AI-generated content — please review before saving."

**Primary Actions:**
- Edit case fields inline
- Check/uncheck reattempt checklist items and save
- Acknowledge or resolve exception flags
- Update case status
- Click any AI Panel action button to generate content
- Edit AI-generated content in the output area
- Save AI content to audit trail
- Copy AI content to clipboard
- Create a new follow-up task via the task summary section
- Navigate to Audit Trail Panel via a "View Audit Trail" link at the bottom

**Key Fields and Components:**

| Field / Component | Type | Validation Rules | Default Value | Notes |
|-------------------|------|-----------------|---------------|-------|
| Case ID | Read-only text | — | System-generated | Format: DFC-YYYYNNNNN |
| Status Badge | Colour label | — | Current status | Colour per status table |
| Edit Case Details | Icon Button | — | — | Toggles fields to editable mode |
| Checklist Items (×5) | Checkboxes | — | Unchecked | See FR-04 |
| Save Checklist Button | Button (Primary) | — | — | Saves checklist state |
| Exceptions Panel | Warning list | — | Empty if none | Each flag has Resolve/Acknowledge action |
| Status Dropdown | Dropdown | Valid transitions only | Current status | 8 values per FR-10 |
| Save Status Button | Button (Primary) | — | — | Updates status with audit entry |
| AI Panel Output | Editable text area | Max 3000 chars | Empty | Populated on LLM action |
| Generate Summary | Button | Case has min required fields | — | Triggers LLM FR-07 |
| Draft Customer Message | Button | Customer contact populated | — | Triggers LLM FR-08 |
| Draft Internal Note | Button | Recipient selected | — | Triggers LLM FR-09 |
| Suggest Next Action | Button | Case has status and reason | — | Triggers LLM FR-12 |
| Convert Response | Button | Response note present | Hidden otherwise | Triggers LLM FR-12 |
| Save to Audit Trail | Button (Secondary) | Output not empty | — | Logs AI content |
| Copy to Clipboard | Icon Button | Output not empty | — | Copies AI content |
| Recipient Dropdown | Dropdown | Required before Draft Internal Note | Select... | Courier Partner / Warehouse Team / Customer Support Team |

**LLM Panel:**
- **Trigger:** User clicks one of the four primary AI action buttons
- **Input:** Full case context including Case ID, Order Number, Customer Name, Failure Reason, Courier Remarks, Checklist Status, Active Exceptions, Current Status, and Assigned Owner
- **Output sections:** (for Generate Summary) Case Overview, Failure Reason, Courier Remarks, Checklist Status, Exceptions, Recommended Next Action; (for Draft Customer Message) Subject Line, Message Body, Call-to-Action; (for Draft Internal Note) Header, Case Reference, Failure Summary, Instructions, Closing; (for Suggest Next Action) Recommended Action, Rationale
- **Editable:** Yes — full text is editable in the output area before saving
- **Save behaviour:** Output is only saved to the audit trail when user explicitly clicks "Save to Audit Trail"; unedited and edited versions are distinguished by "AI-Generated" vs "AI-Generated (Edited)" tags

**Default State:** All case fields populated from the database; checklist items in their last saved state; AI Panel output area empty with placeholder text "Click an action above to generate AI-assisted content."

**Loaded State:** All data visible; checklist, exceptions, and status reflecting current database state.

**Error States:**
- LLM timeout or failure: Output area shows "AI service temporarily unavailable — please try again shortly"; other functions unaffected
- Save fails: Toast notification "Save failed. Please try again."
- Invalid status transition: Inline error message beneath status dropdown

**Navigation Flows:**
- From this screen → Case Dashboard (SCR-01) via breadcrumb
- From this screen → Follow-Up Task Panel (SCR-04) via "View All Tasks" link
- From this screen → Audit Trail Panel (SCR-05) via "View Audit Trail" link

---

### SCR-04: Follow-Up Task Panel

| Field | Detail |
|-------|--------|
| **Screen ID** | SCR-04 |
| **Screen Name** | Follow-Up Task Panel |
| **Purpose** | Allows users to create, view, and manage follow-up tasks associated with a delivery failure case, tracking priority, ownership, and completion status. |
| **Accessible By** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Related User Stories** | US-06, US-09 |
| **Related FRs** | FR-06, FR-11 |

**Layout Description:**
The Follow-Up Task Panel opens as an expandable slide-over drawer from the right side of the Case Detail View, or optionally as a full sub-section below the Exceptions Panel depending on screen width. The panel has a header bar with the title "Follow-Up Tasks" and the case ID reference, and an "×" close button on the right to collapse the panel back into the summary state. Directly below the header is an "Add New Task" button in the secondary style, positioned on the right of the header row. The main content area of the panel is divided into two parts: the task list above and the task creation form below. The task list displays all tasks associated with the case as individual task cards arranged vertically. Each task card shows: a priority badge (High in red, Medium in amber, Low in grey) on the top-left, the task description as the primary text, the assigned owner as a user chip, the due date in smaller text below the description, and a "Status" tag (Open / In Progress / Completed) on the top-right of the card. Overdue tasks with status not "Completed" display an additional red "Overdue" badge. Each card has an "Edit" icon and a "Mark Complete" button. The task creation form at the bottom of the panel shows fields in a compact two-column layout: Task Description (full-width text area, top), Priority (dropdown, left), Required Action (dropdown, right), Assigned Owner (searchable dropdown, left), Due Date (date picker, right), and a full-width "Add Task" primary button at the bottom. The panel allows scrolling within the task list independently of the main Case Detail View.

**Primary Actions:**
- Click "Add New Task" to scroll to and focus the task creation form
- Fill in task creation form fields and click "Add Task"
- Click "Edit" on a task card to edit task details inline
- Click "Mark Complete" on a task to open a resolution note dialog
- Enter resolution note and confirm task completion
- Filter task list by status (Open / In Progress / Completed) using tab controls at the top of the list
- Close the panel using the "×" button

**Key Fields and Components:**

| Field / Component | Type | Validation Rules | Default Value | Notes |
|-------------------|------|-----------------|---------------|-------|
| Task Description | Text Area | Required, max 500 chars | Empty | Character counter shown |
| Priority | Dropdown | Required | Medium | High / Medium / Low |
| Required Action | Dropdown | Required | Select... | Contact Customer / Contact Courier / Verify Address / Schedule Reattempt / Escalate to Manager / Return to Warehouse / Other |
| Assigned Owner | Searchable Dropdown | Required | Current user | Populated from user directory |
| Due Date | Date Picker | Required, ≥ today | Today + 1 day | Past dates blocked |
| Add Task Button | Button (Primary) | All fields valid | — | Creates task record |
| Priority Badge | Colour label | — | Per selection | Red = High, Amber = Medium, Grey = Low |
| Overdue Badge | Red badge | Auto-shown | — | Shown if due date past and status ≠ Completed |
| Mark Complete Button | Button (Outline) | — | — | Opens resolution note modal |
| Resolution Note | Text Area (modal) | Required, min 20 chars | Empty | Required to complete a task |
| Status Tab Controls | Tabs | — | All | Open / In Progress / Completed |

**Default State:** Task list empty; task creation form fields empty with default values; panel shows "No tasks have been created for this case. Click 'Add New Task' to get started."

**Loaded State:** Task cards displayed per saved tasks; overdue badges shown where applicable; status tabs reflect task counts.

**Empty State:** "No tasks match the selected status filter." with a prompt to clear the filter.

**Error States:**
- Required field missing in task creation: Inline error beneath the field
- API save failure: Toast notification "Task could not be saved. Please try again."
- Mark Complete without resolution note: Modal inline error "Resolution note is required before completing a task."

**Navigation Flows:**
- From this panel → Case Detail View (SCR-03) via "×" close button or by scrolling up

---

### SCR-05: Audit Trail Panel

| Field | Detail |
|-------|--------|
| **Screen ID** | SCR-05 |
| **Screen Name** | Audit Trail Panel |
| **Purpose** | Displays a complete, chronological, read-only log of all actions and changes made to a delivery failure case, supporting accountability and operational transparency. |
| **Accessible By** | Back-Office Operations Analyst, Customer Support Executive, Delivery Coordination Associate |
| **Related User Stories** | US-03, US-04, US-05, US-07, US-08, US-10, US-11, US-12 |
| **Related FRs** | FR-11 |

**Layout Description:**
The Audit Trail Panel is accessible via the "View Audit Trail" link at the bottom of the Case Detail View, opening as a full-width sub-view within the Case Detail screen. The top of the panel displays the panel title "Audit Trail" alongside the Case ID, and a "Back to Case" link on the right to return to the main Case Detail View. Below the title row is a filter and sort control bar containing: an Action Type multi-select filter dropdown (pre-populated with all audit entry types), a date range picker to narrow entries by time period, a Sort By control (Newest First / Oldest First), and a search input to find entries by keyword. The main content area displays audit entries in a vertical timeline layout. Each audit entry is represented as a card with a left-side vertical timeline connector line, a circular event icon that reflects the action type (pencil for edits, refresh for status changes, robot icon for AI actions, checkmark for task completions, etc.), and the card body to the right of the icon. Each card body contains: the action type label in bold (e.g., "Status Changed", "AI Summary Saved"), the actor's name and role in smaller text, the timestamp in "DD MMM YYYY HH:MM" format, and a detail block showing the relevant information for that event (e.g., for status changes: "New → Pending Customer"; for field edits: "Customer Phone: [old] → [new]"; for AI saves: a preview of the first 100 characters of the saved content with a "Show full content" expand toggle). Entries are immutable — no edit or delete controls appear anywhere on this panel. A pagination control at the bottom supports navigation through entries, with 50 entries per page.

**Primary Actions:**
- Filter entries by Action Type using the multi-select dropdown
- Filter entries by date range using the date range picker
- Sort entries newest or oldest first
- Search entries by keyword
- Expand AI-generated content previews using "Show full content"
- Navigate through pages using pagination
- Click "Back to Case" to return to Case Detail View

**Key Fields and Components:**

| Field / Component | Type | Validation Rules | Default Value | Notes |
|-------------------|------|-----------------|---------------|-------|
| Action Type Filter | Multi-select Dropdown | — | All types | Values: Case Created, Field Updated, Checklist Saved, Exception Flagged, Exception Resolved, Task Created, Task Completed, AI Summary Generated, AI Summary Saved, Customer Message Drafted, Customer Message Saved, Internal Note Drafted, Internal Note Saved, Status Changed |
| Date Range Filter | Date Range Picker | Start ≤ End | All dates | — |
| Sort By | Radio/Toggle | — | Newest First | Newest First / Oldest First |
| Keyword Search | Text Input | Max 100 chars | Empty | Searches entry description and content |
| Timeline Entry Card | Read-only display | No edits | — | One card per audit event |
| Action Type Label | Bold text | — | — | e.g., "Status Changed" |
| Actor Name + Role | Subtitle text | — | — | e.g., "Ameena – Back-Office Analyst" |
| Timestamp | Formatted text | — | — | DD MMM YYYY HH:MM |
| Detail Block | Read-only text | — | — | Shows changed values or content preview |
| Show Full Content | Expand toggle | Only for AI entries | Collapsed | Expands to show full AI-generated text |
| Pagination | Control bar | — | Page 1, 50 per page | Total entry count displayed |
| Back to Case Link | Text link | — | — | Returns to SCR-03 |

**Default State:** All audit entries for the current case displayed, sorted newest first, no filters applied.

**Loaded State:** Timeline populated with all recorded events in chronological order per sort selection.

**Empty State:** "No audit entries found for the selected filters." with a "Clear Filters" button.

**Error States:**
- API error fetching entries: "Unable to load audit trail. Please refresh the page."
- Search returns no matches: Empty state as above

**Navigation Flows:**
- From this panel → Case Detail View (SCR-03) via "Back to Case" link

---

## Section 7 — Traceability Matrix Skeleton

| FR ID | FR Name | User Story ID | Role | Screen ID | Screen Name | SSD Section | Test Case ID | Status |
|-------|---------|---------------|------|-----------|-------------|-------------|--------------|--------|
| FR-01 | Create Delivery Failure Case | US-01 | Back-Office Operations Analyst | SCR-02 | Case Creation Form | | | Open |
| FR-01 | Create Delivery Failure Case | US-06 | Customer Support Executive | SCR-01 | Case Dashboard | | | Open |
| FR-02 | Capture Order, Customer, Courier, and Failure Details | US-01 | Back-Office Operations Analyst | SCR-02 | Case Creation Form | | | Open |
| FR-02 | Capture Order, Customer, Courier, and Failure Details | US-02 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-03 | Classify Failure Reason | US-02 | Back-Office Operations Analyst | SCR-02 | Case Creation Form | | | Open |
| FR-03 | Classify Failure Reason | US-02 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-04 | Validate Reattempt Checklist | US-03 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-04 | Validate Reattempt Checklist | US-11 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-05 | Identify Exceptions and Blockers | US-03 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-05 | Identify Exceptions and Blockers | US-09 | Customer Support Executive | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-05 | Identify Exceptions and Blockers | US-12 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-06 | Create Follow-Up Task | US-09 | Customer Support Executive | SCR-04 | Follow-Up Task Panel | | | Open |
| FR-06 | Create Follow-Up Task | US-06 | Customer Support Executive | SCR-04 | Follow-Up Task Panel | | | Open |
| FR-07 | Generate AI-Assisted Delivery Failure Summary | US-04 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-08 | Draft Customer-Facing Message | US-07 | Customer Support Executive | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-09 | Draft Internal Courier or Warehouse Follow-Up Note | US-10 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-09 | Draft Internal Courier or Warehouse Follow-Up Note | US-12 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-10 | Update Case Status | US-05 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-10 | Update Case Status | US-11 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-10 | Update Case Status | US-12 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-11 | Maintain Audit Trail | US-05 | Back-Office Operations Analyst | SCR-05 | Audit Trail Panel | | | Open |
| FR-11 | Maintain Audit Trail | US-03 | Back-Office Operations Analyst | SCR-05 | Audit Trail Panel | | | Open |
| FR-11 | Maintain Audit Trail | US-08 | Customer Support Executive | SCR-05 | Audit Trail Panel | | | Open |
| FR-12 | LLM-Assisted Actions | US-04 | Back-Office Operations Analyst | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-12 | LLM-Assisted Actions | US-07 | Customer Support Executive | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-12 | LLM-Assisted Actions | US-08 | Customer Support Executive | SCR-03 | Case Detail View with AI Panel | | | Open |
| FR-12 | LLM-Assisted Actions | US-10 | Delivery Coordination Associate | SCR-03 | Case Detail View with AI Panel | | | Open |

---

## Section 8 — Glossary

| Term | Definition |
|------|------------|
| **Delivery Failure Case** | A formal record created in the System when a delivery attempt for an online retail order is unsuccessful, capturing all relevant order, customer, courier, and failure details for tracking and resolution. |
| **Reattempt** | A subsequent delivery attempt made after an initial failed delivery, scheduled after validating eligibility through the reattempt checklist and resolving any active exceptions. |
| **Failure Reason** | A predefined classification of why a delivery attempt failed, selected from eight standardised categories to ensure consistent case categorisation and reporting. |
| **Reattempt Eligibility** | The confirmation that a case meets all five checklist criteria required before a reattempt can be scheduled, including verified customer contact, confirmed address, captured courier remarks, and a selected next action. |
| **Audit Trail** | An immutable, chronological log of all significant actions and changes made to a delivery failure case, including who performed each action and when, used for accountability, dispute resolution, and operational review. |
| **Back-Office Owner** | The Back-Office Operations Analyst or team member assigned responsibility for a specific delivery failure case, responsible for driving it to resolution. |
| **RTW (Return to Warehouse)** | The process of routing an undeliverable package back to the originating warehouse, initiated when a case is flagged as ineligible for further reattempts or when the failure reason is "Return to Warehouse Required". |
| **Checklist Validation** | The process of confirming each of the five reattempt eligibility checklist items within the Case Detail View, resulting in a validated state that unlocks reattempt scheduling. |
| **Case Status** | The current lifecycle stage of a delivery failure case, selected from eight predefined values (New, Pending Customer, Pending Courier, Reattempt Scheduled, Reattempt Completed, Returned to Warehouse, Resolved, Closed), used to track progress and gate workflow transitions. |
| **Follow-Up Task** | A discrete action item created within a delivery failure case, capturing a specific required action, priority, assigned owner, due date, and completion status to ensure timely resolution of case-related activities. |
| **LLM (Large Language Model)** | The AI language model integrated into the System to provide automated content generation capabilities including case summarisation, customer message drafting, internal note drafting, next-action suggestion, and response conversion. |
| **Courier Partner** | The third-party logistics or delivery company assigned to transport and deliver the customer's order, whose delivery attempt remarks and failure notifications are central inputs to the case record. |
| **Courier Remarks** | Free-text notes entered by the courier partner or captured from the courier portal describing the circumstances of the failed delivery attempt, used as input to exception detection and LLM summarisation. |
| **Exception** | A flagged condition on a delivery failure case that indicates a blocker preventing reattempt scheduling or case progression, such as missing customer contact, incomplete address, or reattempt limit reached. |
| **Docket** | A reference document or record number associated with a delivery attempt, used by courier partners to identify specific shipment events, and referenced in internal notes and warehouse communications. |
| **AI Panel** | The right-hand panel in the Case Detail View that hosts all LLM-assisted actions, displays generated content, and provides controls for editing, saving, and copying AI-generated summaries, messages, and notes. |
| **RBAC (Role-Based Access Control)** | A security model in which a user's access to screens, data, and actions is determined by their assigned system role (Back-Office Operations Analyst, Customer Support Executive, or Delivery Coordination Associate). |
| **Case ID** | A system-generated unique identifier assigned to each delivery failure case upon creation, formatted as DFC-YYYYNNNNN (e.g., DFC-20260001), used for referencing the case across all system screens and communications. |
