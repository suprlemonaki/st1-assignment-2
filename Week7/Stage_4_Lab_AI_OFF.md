# Assignment 2 - Case Study
## Stage 4 Lab Activities — Implementing the SmartCare Domain Layer
### AI OFF

---

## Part A — Revisit Approved UML

Confirming what's locked in from `Stage_3_SmartCare_v03_Domain_Model_Workbook` before writing any code:

| Class | Attributes | Operations | Notes going into implementation |
|---|---|---|---|
| Patient | patientId, name, contactDetails | viewAppointmentHistory() | No change from v0.3 |
| Practitioner | practitionerId, name, specialty | viewUpcomingAppointments(), hasConflict(date, time) | No change from v0.3 |
| Appointment | appointmentId, date, time, status | book(), cancel(), updateDateTime(date, time), getStatus() | `status` (booked/cancelled/completed, FR-10) is formalised as an `AppointmentStatus` enum for implementation — same three states, just made type-safe |

**Confirmed associations:** `Patient 1 — 0..* Appointment`, `Practitioner 1 — 0..* Appointment` (plain associations, not composition or inheritance — Stage 3 Relationship Reasoning, reconfirmed in Stage 4 Tutorial Activity 2).

**Confirmed invariants going into code:** required fields validated before an appointment is saved (NFR-03); a practitioner can't be double-booked (FR-04); cancelled appointments are retained, never deleted (FR-06); business logic stays independently testable and separate from persistence/display code (NFR-02). Clinic remains out of the confirmed model (optional class only).

Nothing here changes the approved v0.3 model — Part A is confirmation, not redesign.

---

## Part B — Implement Patient: AI OFF

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from appointment import Appointment  # avoids a circular import; see note below


class Patient:
    """SmartCare Patient. FR-01 (create), FR-07 (search), FR-11 (view history)."""

    def __init__(self, patient_id: str, name: str, contact_details: str) -> None:
        if not name or not name.strip():
            raise ValueError("Patient name is required (NFR-03).")
        if not contact_details or not contact_details.strip():
            raise ValueError("Patient contact details are required (NFR-03).")

        self._patient_id = patient_id
        self._name = name.strip()
        self._contact_details = contact_details.strip()
        self._appointments: list["Appointment"] = []

    @property
    def patient_id(self) -> str:
        return self._patient_id

    @property
    def name(self) -> str:
        return self._name

    @property
    def contact_details(self) -> str:
        return self._contact_details

    def _record_appointment(self, appointment: "Appointment") -> None:
        """Internal only — called by Appointment when it books itself (Part D).
        Not part of Patient's public API; nothing outside Appointment should call this."""
        self._appointments.append(appointment)

    def view_appointment_history(self) -> list["Appointment"]:
        """FR-11: every appointment, including cancelled ones (FR-06 — nothing is deleted)."""
        return list(self._appointments)
```

No AI needed — the class is a direct, mechanical translation of the CRC card and UML: private state, validated on entry, exposed only through read-only properties and `view_appointment_history()`.

---

## Part C — Implement Practitioner: AI OFF

```python
from __future__ import annotations
from datetime import date as Date, time as Time
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from appointment import Appointment, AppointmentStatus


class Practitioner:
    """SmartCare Practitioner (GP). FR-02 (create), FR-04 (conflict check), FR-08 (own schedule)."""

    def __init__(self, practitioner_id: str, name: str, specialty: str) -> None:
        if not name or not name.strip():
            raise ValueError("Practitioner name is required (NFR-03).")
        if not specialty or not specialty.strip():
            raise ValueError("Practitioner specialty is required (NFR-03).")

        self._practitioner_id = practitioner_id
        self._name = name.strip()
        self._specialty = specialty.strip()
        self._appointments: list["Appointment"] = []

    @property
    def practitioner_id(self) -> str:
        return self._practitioner_id

    @property
    def name(self) -> str:
        return self._name

    @property
    def specialty(self) -> str:
        return self._specialty

    def _record_appointment(self, appointment: "Appointment") -> None:
        """Internal only — called by Appointment when it books itself (Part D)."""
        self._appointments.append(appointment)

    def view_upcoming_appointments(self) -> list["Appointment"]:
        """FR-08: appointments that are still SCHEDULED."""
        from appointment import AppointmentStatus  # deferred import, see note below
        return [a for a in self._appointments if a.status == AppointmentStatus.SCHEDULED]

    def has_conflict(self, date: Date, time: Time) -> bool:
        """FR-04: True if this practitioner already has a SCHEDULED appointment at date/time."""
        from appointment import AppointmentStatus
        return any(
            a.status == AppointmentStatus.SCHEDULED and a.date == date and a.time == time
            for a in self._appointments
        )
```

**No database logic**, as the handout asks — `has_conflict()` checks the practitioner's own in-memory list, not a query.

**Why the deferred imports:** `AppointmentStatus` isn't implemented until Part D (AI ON). Because the `import` sits *inside* the method bodies rather than at the top of the file, Python doesn't need it to exist until `has_conflict()`/`view_upcoming_appointments()` are actually *called* — not when the class is *defined*. That lets Patient and Practitioner be written and reasoned about now, before Appointment exists, without the file failing to load.
