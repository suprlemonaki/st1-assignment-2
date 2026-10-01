# Assignment 2 – Case Study
## Stage 4 Tutorial Activities
### Object-Oriented Design Decisions


---

## Activity 1 — Encapsulation Review

| Class | Protected state / invariant | Public operations |
|---|---|---|
| Patient | `patient_id` immutable once set; `name` and `contact_details` must be non-empty (validated on creation — NFR-03) | `view_appointment_history()` |
| Practitioner | `practitioner_id` immutable; `name` and `specialty` must be non-empty (NFR-03); internal appointment list never exposed for direct mutation | `view_upcoming_appointments()`, `has_conflict(date, time)` |
| Appointment | `status` can only move through legal transitions (SCHEDULED → CANCELLED, SCHEDULED → COMPLETED; nothing leaves CANCELLED or COMPLETED); status is never publicly settable, only readable | `cancel()`, `complete()`, `update_date_time()`, `status` (read-only) |

In each case the *state* is kept private and the *public operations* are the only sanctioned way to change it - nothing outside the class can put an object into an invalid state.

---

## Activity 2 — Composition or Inheritance?

**Appointment and Patient →** ☒ Composition/association ☐ Inheritance
**Reason:** Appointment *has a* link to a Patient, it isn't *a kind of* Patient. It's a plain association, not even composition — nothing suggests an appointment's existence is owned by the patient's lifecycle (FR-06 requires appointments to be retained independently, in history).

**Appointment and Practitioner →** ☒ Composition/association ☐ Inheritance
**Reason:** Same shape as above — Appointment references exactly one Practitioner (FR-04, FR-08) without being a type of Practitioner or owned by it.

**Doctor and Practitioner (hypothetical) →** ☐ Composition/association ☒ Inheritance
**Reason:** Unlike Appointment's links above, a Doctor genuinely *is a* Practitioner — a more specific kind that would share all of Practitioner's attributes and behaviour while potentially adding its own. That "is-a" shape is exactly what inheritance is for, in contrast to the "has-a" shape of the other three relationships on this page.

**Clinic and Appointment →** ☒ Composition/association ☐ Inheritance
**Reason:** Consistent with the Stage 3 answer that Clinic doesn't need to own every object — if Clinic is ever adopted, it would relate to Appointment via a plain association, not composition. Composition would imply an appointment's existence is tied to the clinic record itself, which conflicts with FR-06's requirement that appointments persist independently in history.

---

## Activity 3 — Responsibility Allocation

**Who decides whether SCHEDULED can become CANCELLED?**
Appointment itself. It's the object whose state is changing, so it's the one that should enforce which transitions are legal — not the UI, not a manager class, not whatever code happens to call `cancel()`.

**Who validates a patient name?**
Patient itself, at construction — the same encapsulation principle. Validating required fields at the point of creation is also what NFR-03 asks for.

**Should Appointment execute SQL? Why?**
No. Persistence is an infrastructure/data-access concern, not a domain-model concern. NFR-02 requires business logic to be independently testable and separate from display/output code — mixing SQL into `cancel()` couples the domain object to a specific storage mechanism and makes it impossible to test without a real database. (No FR asks for a database layer at all — "Database" was already ruled out as a domain concept back in Stage 3.)

**Should the UI decide whether a status transition is legal?**
No. The UI should only call `Appointment.cancel()` (or similar) and respect what the object allows or rejects. If the transition rule lived in the UI instead, it would have to be duplicated in every other place that might change an appointment's status later (a future API, a batch job, etc.), and NFR-02's "independently testable" business logic would no longer be true.

---

## Activity 4 — AI Code Critique

*AI generates an Appointment class with public status mutation, SQL inside `cancel()`, a `NotificationManager` dependency and inheritance from `PatientRecord`. Identify at least five design problems and corrections.*

| # | Problem | Correction |
|---|---|---|
| 1 | Public status mutation — `status` is a plain attribute, settable to anything from outside the class | Make `status` private/read-only; the only way to change it is through methods like `cancel()`/`complete()` that enforce legal transitions |
| 2 | SQL inside `cancel()` | Remove persistence entirely from the domain class — no FR asks for a database layer, and NFR-02 requires business logic separate from I/O |
| 3 | `NotificationManager` dependency | Remove it — no FR supports notifications/reminders; this was already rejected for lack of evidence in Stage 3's AI Design Review Record |
| 4 | Inheritance from `PatientRecord` | Replace with a plain association — Appointment *has a* Patient, it isn't *a kind of* Patient (see Activity 2) |
| 5 | `status` stored as a raw value with no enum, so nothing stops an invalid or misspelled status being assigned | Use a proper `AppointmentStatus` enum with a fixed set of members |
| 6 | No validation in the constructor — a required field (patient, practitioner, date or time) could be missing entirely | Validate required fields at construction and raise an exception if any are missing, per NFR-03 |

---

## Exit question

**Why can code be object-oriented syntactically but still have poor object-oriented design?**

Using the `class` keyword and defining methods only satisfies the *syntax* of OOP. Good OO *design* is about how responsibilities, state and dependencies are actually allocated: does an object protect its own invariants (encapsulation), is a relationship modelled as "is-a" or "has-a" correctly (inheritance vs association), and does the object depend only on things it genuinely needs? The Activity 4 class is a perfectly valid Python class — it runs — but it violates encapsulation (public status), misuses inheritance (Appointment isn't a PatientRecord), and pulls in responsibilities that were never asked for (SQL, notifications). None of those are syntax errors; they're design errors that syntax alone can't catch.
