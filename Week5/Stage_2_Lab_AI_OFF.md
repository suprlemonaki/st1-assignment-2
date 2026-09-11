# SmartCare Lab - AI OFF



## Part A - Client Brief: AI OFF

SmartCare uses spreadsheets and paper records. Staff report duplicate bookings, difficulty finding patient information, inconsistent appointment status and limited appointment history. Management wants a small, maintainable patient, practitioner and appointment system.

## Part B - Stakeholders and Scope: AI OFF

### Stakeholders

| Stakeholder | Need | Evidence |
|---|---|---|
| Reception / admin staff | A straightforward way to book, reschedule and search for appointments without duplicate entries. | Client reported duplicate bookings and difficulty finding patient information. |
| GPs | Clear, up-to-date visibility of their own schedule and availability. | Client reported limited visibility of practitioner availability. |
| Patients | A simple way to book appointments and trust that their information is accurate and private. | Implied by the clinic managing patient records and appointments on their behalf. |
| Clinic management | Basic operational reports and a system that stays simple rather than becoming complex. | Client explicitly asked for a "small, maintainable" system and reported difficulty producing basic reports. |

### In scope (v1)

- Managing patient records
- Managing practitioner (GP) records
- Booking, viewing and cancelling appointments for a single clinic location

### Out of scope (v1)

- Facial-recognition login — not mentioned anywhere in the brief, and adds complexity and privacy risk; also a big risk if an info leak happens
- AI-generated diagnosis or treatment-plan recommendations
- Online payment (reject for now — could revisit later)
- Insurance processing (reject for now — could revisit later)

### Provisional (not yet confirmed)

- Appointment status tracking - likely needed to solve the clinic's inconsistent-status problem, but not yet directly confirmed by the client.

## Part C - Functional Requirements: AI OFF

- **FR-01:** The system shall allow staff to create a new patient record with name and contact details.
- **FR-02:** The system shall allow staff to create a new practitioner (GP) record with name and specialty.
- **FR-03:** The system shall allow staff to book an appointment for a patient with a specific practitioner at a specific date and time.
- **FR-04:** The system shall prevent a new appointment being booked for a practitioner who already has an appointment at that date and time.
- **FR-05:** The system shall allow staff to cancel an existing appointment.
- **FR-06:** The system shall retain cancelled appointments in the appointment history rather than deleting them.
- **FR-07:** The system shall allow staff to search for a patient by ID or name.
- **FR-08:** The system shall allow a practitioner to view their own list of upcoming appointments.
- **FR-09:** The system shall allow staff to update an existing appointment's date or time.
- **FR-10:** The system shall display the current status of an appointment (e.g. booked, cancelled, completed).
- **FR-11:** The system shall allow staff to view a patient's appointment history.
- **FR-12:** The system shall produce a basic report listing all appointments for a given day.

## Part D - Non-Functional Requirements: AI OFF

- **NFR-01:** The system shall remain responsive (respond to a search within 2 seconds) for a course-scale dataset of up to a few hundred patients.
- **NFR-02:** Core business logic (booking, cancelling, searching) shall be implemented as independently testable functions, separate from any display/output code.
- **NFR-03:** The system shall validate that required fields (patient name, practitioner, time) are provided before an appointment is saved.
- **NFR-04:** The system's code shall be organised into small, documented functions so it can be maintained and extended by another developer.
- **NFR-05:** Patient and appointment data shall only be modifiable by authorised staff, consistent with healthcare privacy expectations.
- **NFR-06:** The system shall be usable by reception staff after no more than a short walkthrough, without requiring technical training.

## Part E - User Stories and Acceptance Criteria: AI OFF

- **US-01:** As a receptionist, I want to book an appointment for a patient with a specific practitioner, so that I can schedule their visit without double-booking the practitioner.
- **US-02:** As a receptionist, I want to cancel an existing appointment, so that the time slot is no longer counted as booked.
- **US-03:** As a practitioner, I want to view my own upcoming appointments, so that I know my schedule for the day.
- **US-04:** As a receptionist, I want to search for a patient by name or ID, so that I can quickly find their record instead of searching paper files.
- **US-05:** As a member of clinic management, I want to see a basic report of a day's appointments, so that I can review clinic operations without manually checking paper records.
- **US-06:** As a patient, I want to have my cancelled appointments remain in the record, so that staff have an accurate history if I need to be seen again.

### Acceptance criteria (3 of the 6 stories, including one negative scenario)

**US-01 — book an appointment (positive)**
```
GIVEN a patient and a practitioner with no conflicting appointment
WHEN staff book an appointment for that patient at a specific date and time
THEN the appointment is added and appears when the practitioner's schedule is viewed
```

**FR-04 — reject a double-booking (negative/failure scenario)**
```
GIVEN a practitioner already has an appointment at a specific date and time
WHEN staff attempt to book another appointment for that practitioner at the same date and time
THEN the system rejects the booking and shows an error instead of creating a duplicate
```

**US-02 / US-06 — cancel an appointment (positive)**
```
GIVEN a patient has a booked appointment
WHEN staff cancel that appointment
THEN the appointment's status changes to cancelled and it remains visible in the patient's appointment history
```
