# Assignment 2 Case Study
## Stage 2 Tutorial - From Problems to Requirements

*Week 5 | 60 minutes*

# Learning goals

- Analyse stakeholders.

- Distinguish functional and non-functional requirements.

- Recognise ambiguity and unsupported requirements.

- Define scope.

- Develop user stories and acceptance criteria.

- Critique AI-generated requirements.

# Activity 1 - Stakeholder Map

| Stakeholder             | Need                                                                                    | Potential conflict                                                                                                   |
|-------------------------|-----------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| Reception / admin staff | An easy way to book, reschedule, and search for appointments without duplicate entries. | Wants fast, simple data entry — could conflict with validation or security steps that slow booking down.             |
| GPs                     | Clear visibility of their own daily schedule and availability.                          | Wants flexibility to adjust their own schedule — could conflict with reception's need to control bookings centrally. |
| Patients                | A simple way to book appointments and trust that their data is accurate and private.    | Wants minimal friction (e.g. no login step) — could conflict with security/privacy requirements.                     |
| Clinic management       | Basic operational reports, and a system that stays simple rather than becoming complex. | May want more reporting features over time — could conflict with the client's stated wish to keep v1 simple.         |
| Privacy / compliance    | Patient data handled in line with healthcare privacy regulations.                       | Privacy and security controls could slow down the ease-of-use staff and patients want.                               |

# Activity 2 - Functional or Non-Functional?

☒ Functional ☐ Non-functional The system shall allow staff to cancel an appointment.

☐ Functional ☒ Non-functional The system should remain responsive for the course-scale dataset.

☒ Functional ☐ Non-functional The system shall retain cancelled appointments.

☐ Functional ☒ Non-functional Core business logic should be independently testable.

☒ Functional ☐ Non-functional The system shall search for a patient by ID.

# Activity 3 - Repair Ambiguous Requirements

The system should be easy to use.

Problem: “Easy to use” is subjective and unmeasurable — there's no way
to test whether it's been met. Clarification question: What specific
tasks must a first-time user complete without help, and within how many
steps?

Patient search should be fast.

Problem: “Fast” has no defined threshold, so the requirement can't be
verified. Clarification question: What is the maximum acceptable
response time for a patient search, and over how many records?

The system should securely manage data.

Problem: “Securely” doesn't say what security controls are actually
required, so it can't be tested. Clarification question: What specific
controls (e.g. login access, encryption, audit logging) does “securely”
require?

Appointments should normally be easy to cancel.

Problem: “Normally” and “easy” are both vague — it's unclear which cases
are exceptions or what counts as easy. Clarification question: Are there
cases where cancellation should be restricted (e.g. same-day, already
completed), and how many steps should cancelling take?

# Activity 4 - AI Requirements Audit

Classify each suggestion: Confirmed / Assumption requiring validation /
Unsupported / Out of scope.

| AI suggestion                             | Classification                  | Evidence / reason                                                                                                                                                  |
|-------------------------------------------|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Patients receive SMS reminders.           | Assumption requiring validation | Not mentioned in the client statement; plausible and low-risk, but needs confirming with the client before treating it as a requirement.                           |
| Facial recognition login.                 | Unsupported                     | Not mentioned anywhere in the brief, and adds complexity and privacy risk the client explicitly said they don't want — also a big risk if an info leak happens.                                              |
| Receptionists create appointments.        | Confirmed                       | Directly supported — reception staff are a named stakeholder whose role is to manage bookings.                                                                     |
| Online payment.                           | Out of scope                    | Not mentioned in the brief; outside the initial patient/practitioner/appointment scope for v1.                                                                     |
| Practitioners view schedules.             | Confirmed                       | Directly maps to the clinic's stated problem of limited visibility of practitioner availability.                                                                   |
| AI recommends treatments.                 | Unsupported                     | Not mentioned, involves clinical judgement, and conflicts with the client's wish for a simple, non-clinical system.                                                |
| Cancelled appointments remain in history. | Assumption requiring validation | Related to the clinic's stated problem of unreliable appointment history, but the client hasn't confirmed cancelled appointments must be kept rather than deleted. |

# Exit question

Why is 'AI suggested it' not sufficient evidence for a requirement?

Because an AI suggestion isn't evidence of what a stakeholder or the
problem actually needs — it's a plausible-sounding guess. A requirement
has to be traceable to something a stakeholder said or a problem they
described; otherwise the engineer is quietly inventing scope instead of
gathering it, and is accountable if that invented feature wastes effort
or misses what the client actually asked for.
