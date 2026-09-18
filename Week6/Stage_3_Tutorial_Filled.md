# Assignment 2-Case Study
## Stage 3 Tutorial Activities
### From Requirements to Domain Models

---

## Candidate Concepts

| Candidate | Class? | Reason |
|---|---|---|
| Patient | **Yes** | Core domain object with its own identity, attributes (name, contact details – FR-01) and behaviour (search FR-07, view own appointment history FR-11). |
| Practitioner | **Yes** | Core domain object with its own attributes (name, specialty – FR-02) and behaviour (view own appointments FR-08, reject conflicting bookings FR-04). |
| Appointment | **Yes** | Links a Patient and a Practitioner; has its own attributes (date, time, status) and behaviour (book, cancel, update – FR-03–FR-06, FR-09, FR-10). |
| Name | **No** | An attribute of Patient and Practitioner (FR-01, FR-02), not a concept with its own identity or behaviour. |
| Clinic | **Optional** | No FR asks for a Clinic class, and the Stage 2 assumption states a single clinic location for v1. Kept as a separate "optional class" in case the clinic ever expands to multiple locations. |
| Database | **No** | An implementation/persistence detail, not a domain concept — no requirement describes "database" behaviour. |
| Cancellation | **No** | A state transition and behaviour of Appointment (status changes to "cancelled" per FR-05/FR-06), not a class of its own. |
| Status | **No** | An attribute of Appointment (booked/cancelled/completed — FR-10), not a class of its own. |

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

---

## Relationship Reasoning

**Patient to Appointment: which relationship and why?**

A plain **association**, not composition. Appointment is linked to exactly one Patient, but nothing in the requirements says an Appointment's lifecycle is owned by the Patient (deleting a patient record is never mentioned, and FR-06 actually requires appointments to be *retained*, not destroyed alongside anything). An association captures "has a link to" without assuming an ownership/lifecycle rule that isn't evidenced.

**Practitioner to Appointment: what multiplicity?**

Practitioner **1 — 0..\*** Appointment (one practitioner can have many appointments; each appointment has exactly one practitioner). This comes directly from the Stage 2 assumption "each appointment involves exactly one patient and one practitioner."

**Should Appointment inherit from Patient?**

**No.** Inheritance means "is a" — an Appointment is not a kind of Patient, it's a separate concept that *references* a Patient. Modelling it as inheritance would be a classic AI/novice mistake (confusing "has a relationship with" for "is a type of"). The correct relationship is association, as above.

**Does Clinic need to own every object?**

**No.** No FR asks for a Clinic-level owner of Practitioners, Patients or Appointments, and the single-location assumption means there's nothing yet for a Clinic to distinguish between. Making Clinic own everything would add structural complexity the requirements don't support — the same reasoning that keeps Clinic as an optional class rather than a confirmed one (see AI Model Critique below, `ClinicController`).

---

## AI Model Critique

Critique AI proposals: `PatientManager`, `PractitionerManager`, `AppointmentManager`, `ClinicController`, `NotificationManager`, `ScheduleEngine`.

| AI-proposed class | Verdict | Reason |
|---|---|---|
| `PatientManager` | Reject | Duplicates responsibilities that belong on Patient itself (FR-01, FR-07, FR-11). Pulling all real behaviour into a separate "Manager" class is a known over-design pattern — it leaves Patient as an anaemic data bag for no functional gain. |
| `PractitionerManager` | Reject | Same anti-pattern — duplicates Practitioner's own responsibilities (FR-02, FR-04, FR-08). |
| `AppointmentManager` | Reject/fold in | Booking, cancelling and conflict-checking are real, FR-backed behaviours (FR-03–FR-06, FR-09), but nothing requires them to sit outside Appointment. They belong on Appointment itself. |
| `ClinicController` | Reject | No FR asks for cross-cutting, clinic-level control, and v1 assumes a single location — the same over-design flagged in the Relationship Reasoning question above. |
| `NotificationManager` | Reject | No FR supports notifications/reminders at all. Stage 2's AI Requirements Review already rejected "patient email/SMS reminders" for lack of evidence — this re-introduces the same unsupported scope at the design level. |
| `ScheduleEngine` | Reject/fold in | Conflict-checking is genuinely required (FR-04), but it's a natural collaboration between Practitioner and Appointment, not a separate "engine" class with no requirement asking for it. |

**Pattern:** every one of these six proposals follows the same shape — take a real, FR-backed behaviour and move it into a new "Manager"/"Controller"/"Engine" class that no requirement asked for. The behaviour is often legitimate; the *extra class* is not. This is the same evidence test used throughout the assignment: not whether a suggestion sounds like good software design, but whether it traces back to a confirmed requirement.
