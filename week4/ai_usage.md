# AI Usage Documentation

> **Note on tool used:** this lab asks for a UC-approved GenAI tool such as Microsoft Copilot for Parts C and D, because Part C in particular is meant to be a live, back-and-forth tutoring conversation. The exchange below was produced with Claude while drafting this submission, as an example of the process and a starting point — swap in your own transcript from Copilot (or whichever approved tool you used) before you submit, since Part C specifically wants your own conversation and your own answers to the tutor's questions.

## Part C - Use AI as Tutor (AI ON)

**Prompt used** (as suggested in the handout):
> Act as a Python tutor. I am learning introductory software technology. Here is a small appointment-booking function. 1. Explain what the code does. 2. Identify three limitations. 3. Suggest improvements. 4. Do not rewrite the whole application. 5. Ask me two questions to test my understanding.

**Example AI tutor response (illustrative):**
1. *Explanation:* `book_appointment` checks that a patient name was given, builds a dictionary with the patient, practitioner and time, and appends it to a shared `appointments` list; `display_appointments` loops over that list and prints each entry, or a message if it's empty.
2. *Three limitations:* only `patient_name` is validated; nothing is saved once the program exits; two appointments can be booked for the same practitioner at the same time with no warning.
3. *Suggested improvements:* validate `practitioner_name` and `appointment_time` too; check for a clashing practitioner/time before adding a new appointment; consider saving appointments to a file so they survive between runs.
4. *Questions to test understanding:*
   - What would happen if `appointments` were reset inside `book_appointment` instead of being a module-level list — why does that matter?
   - Why does `book_appointment` raise an exception for a blank name instead of just printing a warning and continuing?

*(Replace this section with your own Copilot conversation and your own answers to its questions.)*

## Part D - Generate an Alternative (AI ON)

**Prompt used:**
> Create a simple, beginner-friendly Python function that stores patient name, practitioner name and appointment time. Do not use a database. Do not use a GUI.

**AI-generated code:**
```python
appointments_ai = []

def add_appointment(patient_name, practitioner_name, appointment_time):
    """Add a new appointment to the appointments list."""
    new_appointment = {
        "patient_name": patient_name,
        "practitioner_name": practitioner_name,
        "appointment_time": appointment_time
    }
    appointments_ai.append(new_appointment)
    print(f"Appointment booked for {patient_name} with {practitioner_name} at {appointment_time}.")

def show_appointments():
    """Print all booked appointments."""
    if len(appointments_ai) == 0:
        print("No appointments have been booked yet.")
    else:
        for i, appt in enumerate(appointments_ai, start=1):
            print(f"{i}. {appt['patient_name']} - {appt['practitioner_name']} - {appt['appointment_time']}")
```

## Part F - Verify Behaviour (results used in comparison.md)

Both the human-written and AI-generated versions were run with the same four test cases:

| Test input | Human version (`smartcare_v01.py`) | AI version |
|---|---|---|
| Normal appointment | Booked correctly | Booked correctly |
| Blank patient name | `ValueError: Patient name cannot be empty` | Silently accepted (booked with an empty name) |
| Same practitioner/time booked twice | `ValueError: Dr. John Doe already has an appointment at 2024-07-20 10:00 AM` | Silently accepted (double-booked with no warning) |
| `patient_name=None` | `ValueError: Patient name cannot be empty` | Silently accepted (booked with `None` as the name) |
| `appointment_time=None` | Silently accepted — not yet checked | Silently accepted (booked with `None` as the time) |

This is genuine, executed output, not a prediction — it's what running each function actually produced.

## Part G - Improve One Thing

Chosen improvement: reject a new booking if the same practitioner already has an appointment at that exact time.

```python
for existing in appointments:
    if existing["practitioner"] == practitioner_name and existing["time"] == appointment_time:
        raise ValueError(
            f"{practitioner_name} already has an appointment at {appointment_time}"
        )
```

This was chosen over adding a check for `appointment_time=None` because double-booking a practitioner is the more clinically meaningful failure — it directly causes the kind of scheduling conflict SmartCare's staff described, whereas a missing time is a data-entry problem more than a scheduling one.
