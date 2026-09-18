# Assignment 2 - Case Study
## Stage 3 Lab Activities — SmartCare Domain Modelling
### AI ON

---

## Part E — AI Design Review

**Prompt used:**
> Act as a software design reviewer. Based only on the confirmed functional requirements from SmartCare v0.2 (FR-01 to FR-12), suggest candidate classes and their relationships for the SmartCare domain model. For every suggested class, state which requirement ID(s) support it, or say clearly that no requirement supports it. Do not invent new requirements or scope.

**AI response (illustrative):**

1. `PatientManager` — cited as supported by FR-01, FR-07, FR-11 (create, search, view history).
2. `PractitionerManager` — cited as supported by FR-02, FR-04, FR-08.
3. `AppointmentManager` — cited as supported by FR-03, FR-04, FR-05, FR-06, FR-09, FR-10.
4. `ClinicController` — cited as supported by FR-12 ("coordinates day-level reporting across the clinic").
5. `NotificationManager` — cited as supported by FR-05, FR-06 ("notifies patients when an appointment is cancelled").
6. `ScheduleEngine` — cited as supported by FR-04, FR-08.
7. Suggests modelling Patient–Appointment and Practitioner–Appointment as one-to-many associations (not inheritance), with each Appointment referencing exactly one Patient and one Practitioner — cited as supported by FR-03, FR-04, FR-08, FR-11.

---

## Part F — Compare and Decide

| AI suggestion | Evidence? | Decision | Reason | Model change |
|---|---|---|---|---|
| `PatientManager` | The cited FRs (FR-01, FR-07, FR-11) are real, but they describe things *Patient* does, not a separate manager — the class-level suggestion isn't actually supported by its own citations. | **Reject** | Duplicates Patient's own responsibilities; leaves Patient anaemic for no functional gain. | None — Patient keeps create/search/history responsibilities. |
| `PractitionerManager` | Same issue — FR-02, FR-04, FR-08 describe Practitioner's own behaviour, not a manager class. | **Reject** | Same anti-pattern as above. | None — Practitioner keeps its responsibilities. |
| `AppointmentManager` | FR-03–FR-06, FR-09, FR-10 are genuinely appointment-related, but nothing requires the behaviour to live outside Appointment. | **Modify** | Fold the cited behaviour into Appointment itself, consistent with NFR-02 (business logic as small, testable functions), instead of adding an unrequired class. | Appointment gains `book()`, `cancel()`, `updateDateTime()`, `getStatus()` — already reflected on the CRC card and class diagram. |
| `ClinicController` | FR-12 asks for a *report*, not clinic-level control — the citation doesn't actually support a controller class. | **Reject** | No FR asks for cross-cutting clinic coordination; v1 assumes a single location. | None — Clinic stays as the optional/unconfirmed class only. |
| `NotificationManager` | Not evidence-based — FR-05/FR-06 are about cancelling and retaining appointments, nothing about notifying anyone. | **Reject** | Reintroduces the "SMS/email reminders" scope already rejected for lack of evidence in Stage 2's AI Requirements Review. | None. |
| `ScheduleEngine` | FR-04 conflict-checking is genuine, but no FR asks for a separate engine class. | **Modify** | Conflict-checking is a natural collaboration between Practitioner and Appointment, not a distinct class. | Practitioner gains `hasConflict(date, time)`; shown as a Practitioner–Appointment collaboration, not a new class. |
| Association (not inheritance) between Patient/Practitioner and Appointment | Directly reflects the Stage 2 assumption that each appointment involves exactly one patient and one practitioner, and FR-03/FR-04/FR-08/FR-11 already assume this shape. | **Accept** | Correctly reflects confirmed requirements; no reason to model as inheritance or composition. | Formalised in the class diagram as `Patient 1 — 0..* Appointment` and `Practitioner 1 — 0..* Appointment`. |

**Pattern:** every "Manager/Controller/Engine" suggestion cited a real requirement but claimed it supported a *class* the requirement never actually asked for — a subtler version of the "AI suggested it" problem from Stage 2, since this time the AI *did* cite evidence, just evidence that didn't support the specific design decision it was attached to. Checking the citation against what it's actually being used to justify is what the compare/decide step is for.

---

## Part G — Python Skeletons

Skeletons only — no full behaviour yet (that's Part H's job to check, not implement):

```python
class Patient:
    """Patient record. Attributes: FR-01. Behaviour: FR-07, FR-11."""

    def __init__(self, patient_id, name, contact_details):
        self.patient_id = patient_id
        self.name = name
        self.contact_details = contact_details

    def view_appointment_history(self):
        """FR-11: return this patient's appointments, including cancelled ones (FR-06)."""
        raise NotImplementedError


class Practitioner:
    """Practitioner (GP) record. Attributes: FR-02. Behaviour: FR-04, FR-08."""

    def __init__(self, practitioner_id, name, specialty):
        self.practitioner_id = practitioner_id
        self.name = name
        self.specialty = specialty

    def view_upcoming_appointments(self):
        """FR-08: return this practitioner's upcoming appointments."""
        raise NotImplementedError

    def has_conflict(self, date, time):
        """FR-04: check for an existing appointment at the same date/time."""
        raise NotImplementedError


class Appointment:
    """Appointment linking one Patient and one Practitioner. FR-03, FR-10."""

    def __init__(self, appointment_id, patient, practitioner, date, time, status="booked"):
        self.appointment_id = appointment_id
        self.patient = patient
        self.practitioner = practitioner
        self.date = date
        self.time = time
        self.status = status  # FR-10: booked / cancelled / completed

    def book(self):
        """FR-03: create the appointment, subject to Practitioner.has_conflict (FR-04)."""
        raise NotImplementedError

    def cancel(self):
        """FR-05/FR-06: set status to 'cancelled'; the record is kept, not deleted."""
        raise NotImplementedError

    def update_date_time(self, new_date, new_time):
        """FR-09: change the appointment's date/time."""
        raise NotImplementedError

    def get_status(self):
        """FR-10: report the current status."""
        raise NotImplementedError
```

Note what's deliberately absent: no `PatientManager`, `AppointmentManager`, `ClinicController`, `NotificationManager` or `ScheduleEngine` — consistent with the Reject/Modify decisions in Part F.

---

## Part H — Consistency Check

Model-code consistency only — full behaviour is intentionally not implemented yet.

| Class | In UML model? | In CRC card? | In code skeleton? | Consistent? |
|---|---|---|---|---|
| Patient | Yes | Yes | Yes — same attributes (`patient_id`, `name`, `contact_details`), same operation (`view_appointment_history`) | Yes |
| Practitioner | Yes | Yes | Yes — same attributes, plus both operations (`view_upcoming_appointments`, `has_conflict`) | Yes |
| Appointment | Yes | Yes | Yes — same attributes (incl. `status`), all four operations present as stubs | Yes |
| Clinic | Yes (optional/dashed) | Yes (Optional class card) | **Not coded** | Yes — intentional; Clinic is unconfirmed for v1, so it's correctly absent from the code, not a gap. |
| `PatientManager`/`AppointmentManager`/etc. | Not in model | Not in model | Not in code | Yes — consistently rejected/folded in everywhere, not just in one artifact. |

Every method body is a stub (`raise NotImplementedError`) rather than real logic — that's expected at this stage per the handout instruction, and is itself consistent across all three classes.

---

## Reflection (150–250 words)

The hardest modelling decision was Clinic. It's a plausible class — clinics conceptually "contain" practitioners and appointments — but nothing in the confirmed FRs asks for it, and the Stage 2 assumption explicitly states a single location for v1. It was tempting to include it anyway because it "felt" like it belonged in a clinic system, which is exactly the kind of reasoning the whole assignment has been pushing back against. Keeping it as a clearly-labelled optional class, rather than either forcing it in or leaving it out silently, felt like the right middle ground.

The AI over-designed in a very specific way this time: every "Manager/Controller/Engine" suggestion came with a requirement ID attached, so at first glance it looked evidence-based. Checking each citation against what it was actually being used to justify showed the requirement supported the *behaviour*, not the *extra class* — FR-04 is a real rule, but it doesn't ask for a `ScheduleEngine`, it asks for a conflict check, which Practitioner can do itself.

The evidence that shaped the final model was the Stage 2 assumption ("each appointment involves exactly one patient and one practitioner") and the FR list itself — both gave concrete, checkable answers to relationship and multiplicity questions instead of guesses.
