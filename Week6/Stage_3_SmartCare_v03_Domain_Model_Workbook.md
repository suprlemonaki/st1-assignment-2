# SmartCare v0.3 — Domain Model Workbook

---

## Requirement-to-Concept Trace

| Requirement | Concept | State/behaviour | Decision |
|---|---|---|---|
| FR-01 | Patient | name, contactDetails (state) | Confirmed — attributes on Patient. |
| FR-02 | Practitioner | name, specialty (state) | Confirmed — attributes on Practitioner. |
| FR-03 | Appointment | `book()` (behaviour); links to Patient + Practitioner (state) | Confirmed — Appointment class with associations to Patient/Practitioner. |
| FR-04 | Practitioner ↔ Appointment | `hasConflict(date, time)` (behaviour) | Confirmed — Practitioner/Appointment collaboration; no separate `ScheduleEngine` class (see AI Design Review Record). |
| FR-05 | Appointment | `cancel()` (behaviour); status (state) | Confirmed — Appointment class. |
| FR-06 | Appointment | status retained, record not deleted (state) | Confirmed — status persists rather than the record being removed. |
| FR-07 | Patient | search by ID/name (behaviour, supported by identifying attributes) | Confirmed — enabled by patientId/name already on Patient. |
| FR-08 | Practitioner | `viewUpcomingAppointments()` (behaviour) | Confirmed — Practitioner class, collaborates with Appointment. |
| FR-09 | Appointment | `updateDateTime()` (behaviour) | Confirmed — Appointment class. |
| FR-10 | Appointment | status (state: booked/cancelled/completed) | Confirmed — status attribute on Appointment. |
| FR-11 | Patient ↔ Appointment | `viewAppointmentHistory()` (behaviour) | Confirmed — Patient collaborates with Appointment. |
| FR-12 | *(none — see Clinic, optional)* | day's-appointments report (derived, not stored state) | Not modelled as its own class — a report is a derived view over existing Appointment records, not a new domain concept. |

---

## CRC Cards

### Patient

| Responsibilities | Collaborators |
|---|---|
| Store own identifying and contact details (FR-01) | Appointment |
| Provide own appointment history, including cancelled appointments (FR-06, FR-11) | |

### Practitioner

| Responsibilities | Collaborators |
|---|---|
| Store own name and specialty (FR-02) | Appointment |
| Provide own list of upcoming appointments (FR-08) | |
| Check whether a proposed time conflicts with an existing appointment (FR-04) | |

### Appointment

| Responsibilities | Collaborators |
|---|---|
| Hold its own date, time, status and links to one patient and one practitioner (FR-03, FR-10) | Patient |
| Change its own status when cancelled, without being deleted (FR-05, FR-06) | Practitioner |
| Support being updated to a new date/time (FR-09) | |

### Optional class — Clinic

| Responsibilities | Collaborators |
|---|---|
| Represent the single clinic location assumed for v1 (not requested by any FR) | Practitioner (if adopted) |
| If ever confirmed, would group Practitioners (and by extension Appointments/reports) under one location | Appointment (if adopted) |

---

## UML Class Diagram

*Include defensible relationships and multiplicities.*

![SmartCare domain model class diagram](classdiagram.png)

- `Patient` **1 — 0..\*** `Appointment` (FR-03, FR-11; Stage 2 assumption)
- `Practitioner` **1 — 0..\*** `Appointment` (FR-04, FR-08; Stage 2 assumption)
- `Clinic` **1 — 0..\*** `Practitioner`, drawn dashed/greyed — not part of the confirmed model, shown only to indicate where it would attach if adopted later.

---

## Design Rationale

*Explain class selection, responsibility allocation and key relationships.*

Three classes made it into the confirmed model — Patient, Practitioner and Appointment — because every one of them has both state and behaviour directly traceable to a requirement (see the trace table above). Everything else considered during the tutorial and lab either turned out to be an attribute of one of these three (Name, Status), an implementation detail rather than a domain concept (Database), a state/behaviour rather than a class (Cancellation), or a class with no confirmed requirement behind it (Clinic — kept as an explicitly optional card rather than silently dropped or silently included).

Responsibility allocation followed a "who actually owns this data" test rather than an "what sounds like a good software design" test. Booking, cancelling and updating (FR-03, FR-05, FR-09) sit on Appointment because Appointment is the thing whose state actually changes. Conflict-checking (FR-04) sits on Practitioner in collaboration with Appointment, because a conflict is fundamentally a question about a *practitioner's* existing schedule. This is also why the six AI-proposed "Manager/Controller/Engine" classes were rejected or folded in rather than accepted (see AI Design Review Record below): each one took a legitimate, requirement-backed behaviour and relocated it to a class no requirement asked for.

Both confirmed relationships are plain one-to-many associations, not inheritance or composition. Inheritance was explicitly ruled out for Patient–Appointment (an appointment is not a *kind of* patient), and composition was avoided because no requirement describes appointments being destroyed when a patient record is — FR-06 in fact requires the opposite, that appointments persist independently in history. Clinic remains outside the confirmed model for the same evidence standard applied throughout the assignment: the v1 assumption is a single location, and no FR asks the system to model, own or report across multiple locations.

---

## AI Design Review Record

| AI suggestion | Evidence | Decision | Reason | Model change |
|---|---|---|---|---|
| `PatientManager` | Cited FR-01, FR-07, FR-11 — real requirements, but they describe Patient's own behaviour, not a manager class. | **Reject** | Duplicates Patient's own responsibilities; leaves Patient anaemic for no functional gain. | None — Patient keeps create/search/history responsibilities. |
| `PractitionerManager` | Cited FR-02, FR-04, FR-08 — same issue. | **Reject** | Same anti-pattern as above. | None — Practitioner keeps its responsibilities. |
| `AppointmentManager` | Cited FR-03–FR-06, FR-09, FR-10 — genuinely appointment-related, but not evidence for a separate class. | **Modify** | Fold the cited behaviour into Appointment itself, consistent with NFR-02, instead of adding an unrequired class. | Appointment gains `book()`, `cancel()`, `updateDateTime()`, `getStatus()`. |
| `ClinicController` | Cited FR-12 — that FR asks for a report, not clinic-level control. | **Reject** | No FR asks for cross-cutting clinic coordination; v1 assumes a single location. | None — Clinic stays as the optional/unconfirmed class only. |
| `NotificationManager` | Cited FR-05/FR-06 — neither mentions notifying anyone. | **Reject** | Reintroduces the "SMS/email reminders" scope already rejected for lack of evidence in Stage 2. | None. |
| `ScheduleEngine` | Cited FR-04, FR-08 — conflict-checking is genuine, but not evidence for a separate engine class. | **Modify** | Conflict-checking is a natural Practitioner–Appointment collaboration, not a distinct class. | Practitioner gains `hasConflict(date, time)`. |
| Association (not inheritance) between Patient/Practitioner and Appointment | Directly reflects the Stage 2 assumption and FR-03/FR-04/FR-08/FR-11. | **Accept** | Correctly reflects confirmed requirements; no reason to model as inheritance or composition. | Formalised as `Patient 1 — 0..* Appointment` and `Practitioner 1 — 0..* Appointment`. |

