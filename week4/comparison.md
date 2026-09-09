# Part A - Understand the Problem (AI OFF)

**What data must be stored?**
Patient name, practitioner name, and appointment time for every appointment, plus somewhere to hold multiple appointments together (a list of records).

**What functions might be useful?**
A function to book/add an appointment, a function to display all appointments, and - beyond what's in the starter code - functions to cancel or edit an appointment, and to search by patient or practitioner.

**What could go wrong?**
A blank or missing patient name, a missing practitioner or time, two appointments booked for the same practitioner at the same time, inconsistent time formats, and appointments being lost entirely because nothing is saved outside the running program.

**What requirements are unclear?**
Whether `appointment_time` needs strict date/time validation, whether the same patient can be double-booked, whether cancelling or editing appointments is in scope, and whether data needs to persist between runs (a file or database) rather than living only in memory.

# Part B - Limitations found by running the human-written prototype

Both `task1` and `task1enhanced` were run successfully (see terminal output). Testing the enhanced version against edge cases surfaced these limitations:

1. **No validation on `practitioner_name` or `appointment_time`** — both silently accept `None` or a blank string; only `patient_name` was checked in the original code.
2. **Nothing is persisted** — all appointments live in an in-memory list and disappear the moment the program ends.
3. **No protection against double-booking** — the original code let two different patients book the same practitioner at the same time with no warning (fixed in `smartcare_v01.py` as the Part G improvement).
4. **`appointment_time` is just a free-text string** — there's no check that it's a real date/time or that it's in the future.
5. **No way to cancel, edit, or search** — the prototype can only add and display appointments.
6. **The appointment list is a bare module-level global** — `book_appointment` reads and mutates `appointments` directly rather than it being passed in or returned, which makes the function harder to test in isolation.

# Part E - Compare Human and AI Versions

| Question | Human version | AI version |
|---|---|---|
| Easy to understand? | Yes — short functions, explicit error messages | Yes — arguably even simpler; uses `print` instead of raising errors |
| Runs successfully? | Yes, on normal and edge-case input | Yes, on all input, including invalid input |
| Uses only required features? | Yes — list, dict, functions only, no DB/GUI | Yes — list, dict, functions only, no DB/GUI |
| Adds assumptions? | Assumes `appointment_time` is a sensibly formatted string but doesn't silently accept missing data | Assumes every input is always valid — no assumption is ever checked |
| Handles errors? | Yes — raises `ValueError` for a blank patient name and for a double-booked practitioner/time (Part G) | No — blank names, `None` values, and duplicate bookings were all accepted without any error |
| Could I explain it? | Yes, and I could justify each validation choice | Yes — the code itself is simple, but its silent lack of validation is easy to miss without testing it |

*(Verified by executing both versions with the Part F test cases — see `ai_usage.md` for the exact inputs and outputs.)*
