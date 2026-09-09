# Stage 1 Tutorial Activity - Why Software Engineering Still Matters

*Stage 1 | Introducing Software Technology Case Study with Python and Guided AI use*

## Learning goals

- Explain why software engineering is broader than coding.
- Identify stakeholders in a simple software problem.
- Recognise missing requirements.
- Critically evaluate AI-generated feature suggestions.
- Explain why AI output should not automatically be treated as correct.

## Activity 1 - Think-Pair-Share (10 minutes)

If ChatGPT or Copilot can produce a 100-line Python application very quickly, what knowledge does a software engineer still need?

1. Understanding the needs of the user and not just what they typed, gathering requirements and stakeholder communication
2. System design skills are still needed so it is secure, maintainable, and scales farther than a demo
3. Critical judgement to avoid any issues relating to laws, rules and ethics/morals, where the engineer is needing to test, verify, and take responsibility for the AI-generated code and whether it is safe and correct.

## Activity 2 - Is This Software Engineering? (10 minutes)

Scenario A: A student writes a 50-line Python calculator.
Scenario B: A team develops a payroll system used by 5,000 employees.
Scenario C: An AI assistant generates a simple appointment application from one prompt.

| Scenario | Programming? | Software engineering? | Why? |
|---|---|---|---|
| A | Yes | No | A small, single person script with no requirements analysis and stakeholders as well as no maintenance to manage long term |
| B | Yes | Yes | Requires requirements gathering, stakeholder management, testing, security, and ongoing maintenance for thousands of user |
| C | Yes | No | Code was generated with no analysis of stakeholder needs or verification of correctness. That's code generation, not engineering. |

## Activity 3 - SmartCare Problem Analysis (20 minutes)

Client statement: SmartCare Community Clinic currently uses spreadsheets and paper records to manage patients and appointments. The clinic wants new software to improve these processes.

### Task 1 - Identify stakeholders

| Stakeholder | What do they need? |
|---|---|
| Reception / admin staff | An easy way to book, view, and reschedule appointments |
| Doctors / GPs | Fast and reliable access to patient records and their own daily schedule |
| Patients | A simple way to book appointments and confidence that their personal and medical information is kept accurate and protected |
| Clinic management | Reporting on clinic operations and assurance the new system meets healthcare privacy regulations |

### Task 2 - Identify current problems

1. Duplicate appointment bookings happen because there is no shared system tracking who has booked what and when
2. Patient records are spread across paper files and spreadsheets, making them easy to lose and hard to find
3. Appointment status information is inconsistent, so staff can't reliably tell if a booking is confirmed, cancelled, or completed
4. There is limited visibility of doctor availability, and cancellations are handled manually with no reliable appointment history

### Task 3 - Ask client questions

1. What basic operational reports does management need out of the system?
2. What statuses should an appointment have (e.g. booked, confirmed, cancelled, completed)?
3. How should doctor availability be displayed and kept up to date?
4. Who should be allowed to cancel or modify an appointment, and should the history be kept afterwards?
5. Since this system will be built iteratively across stages, which features are must-haves for this first simple version versus later stages?

## Activity 4 - Critique an AI Response (15 minutes)

An AI assistant suggests: appointment management; facial-recognition login; AI diagnosis recommendations; patient search; online payment; practitioner schedule view; insurance processing; automatic treatment-plan generation.

| Suggestion | Client evidence? | In scope? | Decision |
|---|---|---|---|
| Appointment management | Yes (directly maps to duplicate bookings, inconsistent status, manual cancellations) | Yes | **Include** — Core to the stated scope: patient, practitioner and appointment management. |
| Facial recognition login | No | No | **Reject** — Not mentioned anywhere in the brief, and the client explicitly said they don't want a complex system. Not to mention a big risk if a info leak happens |
| AI diagnosis recommendations | No | No | **Reject** — Outside the stated scope (patient/practitioner/appointment management only) and too risky for a clinical tool. |
| Patient search | Yes (directly maps to "difficulty locating patient records") | Yes | **Include** — Solves one of the clinic's named operational problems. |
| Online payment | No | No | **Reject for now** — Not mentioned in the brief; outside the initial patient/practitioner/appointment scope, could revisit later. |
| Practitioner schedule view | Yes (directly maps to "limited visibility of practitioner availability") | Yes | **Include** — Solves one of the clinic's named operational problems. |
| Insurance processing | No | No | **Reject for now** — Not mentioned, and outside the simple patient/practitioner/appointment scope. |
| Treatment-plan generation | No | No | **Reject** — Not mentioned, involves clinical judgement, and the clinic explicitly doesn't want a complex hospital system. |

### Exit question

Write one activity that a software engineer must perform and that cannot safely be delegated entirely to AI.

Deciding what the system should NOT do, for example, rejecting AI-generated treatment-plan or diagnosis features, this requires professional judgement, accountability, and understanding of context and risk that AI cannot and should not be trusted with.
