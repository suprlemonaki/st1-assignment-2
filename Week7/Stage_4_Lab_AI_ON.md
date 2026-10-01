# Assignment 2 - Case Study
## Stage 4 Lab Activities — Implementing the SmartCare Domain Layer
### AI ON

---

## Part D — Implement Appointment: AI ON

**Suggested AI prompt used:**
> Act as a Python pair programmer. Implement only the Appointment class from the approved SmartCare UML. Use type hints and an AppointmentStatus enum. Cancelled appointments remain as objects. Do not add database, UI, notification or service classes. Protect status transitions and explain any decision not directly visible in the UML.

**AI-generated code (illustrative, unreviewed):**

```python
import sqlite3
from enum import Enum


class AppointmentStatus(Enum):
    SCHEDULED = "scheduled"
    CANCELLED = "cancelled"
    COMPLETED = "completed"


class PatientRecord:
    def __init__(self, patient_id, name):
        self.patient_id = patient_id
        self.name = name


class Appointment(PatientRecord):
    def __init__(self, appointment_id, patient, practitioner, date, time):
        super().__init__(patient.patient_id, patient.name)
        self.appointment_id = appointment_id
        self.patient = patient
        self.practitioner = practitioner
        self.date = date
        self.time = time
        self.status = AppointmentStatus.SCHEDULED  # public attribute
        self.notifier = NotificationManager()

    def cancel(self):
        self.status = AppointmentStatus.CANCELLED
        conn = sqlite3.connect("smartcare.db")
        conn.execute(
            "UPDATE appointments SET status = ? WHERE id = ?",
            (self.status.value, self.appointment_id),
        )
        conn.commit()
        self.notifier.send_cancellation(self.patient)


class NotificationManager:
    def send_cancellation(self, patient):
        print(f"Notifying {patient.name} of cancellation...")
```

The AI's own explanation (illustrative): *"I used inheritance from PatientRecord so Appointment has direct access to patient details, added a NotificationManager so patients are informed when their appointment is cancelled, and persisted the change immediately with SQLite so the cancellation isn't lost."* — none of these three justifications trace back to a confirmed requirement, which is exactly what Part E checks.

---

## Part E — Review Generated Code

| Check | Finding | Verdict |
|---|---|---|
| Model consistency | `Appointment(PatientRecord)` — inheritance isn't in the approved UML; Appointment associates with Patient, it doesn't inherit from anything patient-related | Fail |
| Unsupported features | SQL persistence and `NotificationManager` — neither is in scope; no FR asks for a database layer, and notifications were already rejected in Stage 3 for lack of evidence | Fail |
| Public state mutation | `self.status` is a plain public attribute, freely reassignable from outside the class | Fail |
| Unnecessary inheritance | Same as model consistency above — an "is-a" relationship where only "has-a" (association) is justified | Fail |
| Invented dependencies | `NotificationManager()` instantiated directly inside `__init__`, with no requirement behind it | Fail |
| Error handling | No validation of required fields in `__init__`; no guard against illegal transitions (nothing stops cancelling twice, or cancelling a COMPLETED appointment) | Fail |

Six checks, six failures — the AI's own stated justifications (direct patient access, keeping patients informed, not losing the cancellation) all sound reasonable in isolation, but none of them trace back to a confirmed requirement, which is the same evidence test used throughout every AI-ON activity in this assignment.

---

## Part F — Manual Behaviour Checks

Running the checks the handout asks for — *create valid objects, test invalid input, cancel a scheduled appointment, attempt an illegal repeated transition* — directly against the flawed AI-generated code above, to get concrete evidence rather than relying on the review alone:

```python
patient = PatientRecord("P1", "Jane Citizen")  # only works because Appointment inherited from it —
                                                # a real Patient object doesn't fit this constructor at all
appt = Appointment("A1", patient, practitioner, None, None)  # no error — missing date/time silently accepted

appt2 = Appointment("A2", patient, practitioner, date(2026, 10, 1), time(9, 0))
appt2.cancel()
print(appt2.status)          # CANCELLED — cancel() itself "works"...
appt2.cancel()                # ...but calling it again is accepted with no error
appt2.status = AppointmentStatus.SCHEDULED  # direct mutation bypasses cancel() entirely — also accepted
```

**Result:** every one of the four checks the handout asks for exposes a real, runnable failure — not just a style objection. Missing required fields are silently accepted, a cancelled appointment can be "re-cancelled," and status can be reassigned directly with no method call at all. This is the empirical evidence backing Part E's review.

---

## Part G — Refactor

```python
from __future__ import annotations
from datetime import date as Date, time as Time
from enum import Enum


class AppointmentStatus(Enum):
    SCHEDULED = "scheduled"
    CANCELLED = "cancelled"
    COMPLETED = "completed"


class IllegalStatusTransition(Exception):
    """Raised when an Appointment status change isn't allowed."""


# Who decides whether SCHEDULED can become CANCELLED? Appointment does (Tutorial Activity 3) —
# enforced here, not left to whatever code happens to call cancel().
_ALLOWED_TRANSITIONS: dict[AppointmentStatus, set[AppointmentStatus]] = {
    AppointmentStatus.SCHEDULED: {AppointmentStatus.CANCELLED, AppointmentStatus.COMPLETED},
    AppointmentStatus.CANCELLED: set(),  # terminal — retained per FR-06, never reopened
    AppointmentStatus.COMPLETED: set(),  # terminal
}


class Appointment:
    """SmartCare Appointment. FR-03 (book), FR-05/FR-06 (cancel, retained), FR-09 (update), FR-10 (status)."""

    def __init__(self, appointment_id: str, patient: "Patient", practitioner: "Practitioner",
                 date: Date, time: Time) -> None:
        if patient is None or practitioner is None:
            raise ValueError("An appointment requires both a patient and a practitioner (NFR-03).")
        if date is None or time is None:
            raise ValueError("An appointment requires a date and a time (NFR-03).")
        if practitioner.has_conflict(date, time):
            raise ValueError("Practitioner already has an appointment at that date/time (FR-04).")

        self._appointment_id = appointment_id
        self._patient = patient
        self._practitioner = practitioner
        self._date = date
        self._time = time
        self._status = AppointmentStatus.SCHEDULED

        patient._record_appointment(self)
        practitioner._record_appointment(self)

    @property
    def appointment_id(self) -> str:
        return self._appointment_id

    @property
    def status(self) -> AppointmentStatus:
        return self._status

    @property
    def date(self) -> Date:
        return self._date

    @property
    def time(self) -> Time:
        return self._time

    def _transition_to(self, new_status: AppointmentStatus) -> None:
        if new_status not in _ALLOWED_TRANSITIONS[self._status]:
            raise IllegalStatusTransition(
                f"Cannot move an appointment from {self._status.value} to {new_status.value}."
            )
        self._status = new_status

    def cancel(self) -> None:
        """FR-05: cancel; FR-06: the object is never deleted, only its status changes."""
        self._transition_to(AppointmentStatus.CANCELLED)

    def complete(self) -> None:
        """Not explicit in the v0.3 UML operations list — added here since COMPLETED was
        already a valid status value (FR-10) with no way to legally reach it. See the
        Domain Implementation Workbook, Section 5 (Updated UML)."""
        self._transition_to(AppointmentStatus.COMPLETED)

    def update_date_time(self, new_date: Date, new_time: Time) -> None:
        """FR-09. Only legal while still SCHEDULED."""
        if self._status != AppointmentStatus.SCHEDULED:
            raise IllegalStatusTransition("Only a scheduled appointment's date/time can be updated.")
        if self._practitioner.has_conflict(new_date, new_time):
            raise ValueError("Practitioner already has an appointment at that date/time (FR-04).")
        self._date = new_date
        self._time = new_time
```

**Removed entirely:** inheritance from `PatientRecord`, the SQL call, `NotificationManager`. **Added:** required-field validation, the conflict check at booking time (FR-04), a private `_status` with a read-only property, and an explicit transition table so illegal moves raise `IllegalStatusTransition` instead of silently succeeding.

**Confirming the fix** — rerunning the same four checks from Part F against the refactored class:

```python
Appointment("A3", None, practitioner, date(2026, 10, 1), time(9, 0))
# ✅ raises ValueError: "An appointment requires both a patient and a practitioner (NFR-03)."

appt3 = Appointment("A4", patient, practitioner, date(2026, 10, 1), time(10, 0))
appt3.cancel()
appt3.cancel()
# ✅ raises IllegalStatusTransition: "Cannot move an appointment from cancelled to cancelled."

appt3.status = AppointmentStatus.SCHEDULED
# ✅ raises AttributeError — status has no setter, direct mutation is no longer possible
```

Every failure from Part F is now a raised exception instead of a silent success.

---

## Part H — AI Engineering Log

| Element | Record |
|---|---|
| Prompt used | See Part D — "Implement only the Appointment class... Use type hints and an AppointmentStatus enum... Protect status transitions..." |
| AI-generated contribution | The full Part D code: `AppointmentStatus` enum, `PatientRecord`, `Appointment(PatientRecord)`, `NotificationManager` |
| Kept | The three-member `AppointmentStatus` enum shape (SCHEDULED/CANCELLED/COMPLETED) — matches FR-10 and is a genuine improvement in type-safety over a raw string |
| Rejected | Inheritance from `PatientRecord`, the SQL call inside `cancel()`, the `NotificationManager` dependency, public `status` mutation, missing field validation, missing transition guard — six items, matching Tutorial Activity 4's critique exactly |
| Verification evidence | Part E's static review (6/6 checks failed) plus Part F's runtime evidence (each failure reproduced as an actual silent bug) plus Part G's re-run showing each one now raises the correct exception |

---

## Reflection

The part I modified most heavily was the status handling, the AI's version stored `status` as a plain public attribute, which meant nothing actually stopped an illegal transition; it just looked protected because the value happened to be an enum member. Switching to a private `_status` with a read-only property and an explicit transition table was the real fix, not the enum itself.

I rejected the inheritance from `PatientRecord` outright. This is the same is-a/has-a mistake the Stage 3 Tutorial already flagged for the plain Patient class, so seeing it resurface here, just with a differently-named class, was a useful reminder that renaming something doesn't change what kind of relationship it actually is. The `NotificationManager` and the SQL call were even easier calls: neither traces to any requirement at all, and the notification manager specifically had already been rejected once before, in Stage 3's AI Design Review Record, for exactly the same reason.

The approved design constrained the AI in a very direct way: because Part A had already locked in the UML, CRC responsibilities and confirmed associations before any code was written, reviewing the generated code wasn't a matter of judging whether it "looked reasonable", it was a checklist of whether it matched something already decided. Every rejected element failed that check for a specific, nameable reason, not a vague one. That's the difference between reviewing AI output against your own prior design versus asking the AI to design as it goes, the second one has nothing to be checked against.
