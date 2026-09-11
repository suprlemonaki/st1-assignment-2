# SmartCare Lab - AI ON

## Part F - AI Requirements Review: AI ON

**Prompt used:**
> Act as a software requirements reviewer. Review the SmartCare requirements for ambiguity, inconsistency, missing clarification questions and testability. Do NOT invent new client requirements. For every suggestion, state whether it is based on evidence or is only a question/assumption requiring validation.

## Part G - VERIFY the AI Review

| AI suggestion | Evidence? | Decision | Reason | Verification |
|---|---|---|---|---|
| Add a confirmation step before cancelling an appointment. | Not evidence-based — a plausible UX suggestion, not something requested by any stakeholder. | Modify | Improves usability but doesn't need its own FR — folded into how FR-05 (cancel) is implemented. | Reviewed against existing FR-05; no new client evidence needed since it's an implementation choice, not new scope. |
| FR-04 should also check the same patient isn't double-booked, not just the practitioner. | Evidence-based — follows directly from the clinic's stated duplicate-booking problem. | **Accept** | Genuinely closes a gap; a patient being double-booked is the same category of problem as a practitioner being double-booked. | Traced back to the client's stated "duplicate appointment bookings" problem; added as a refinement to FR-04. |
| Add patient email/SMS reminders as a requirement. | Not evidence-based — not mentioned anywhere by the client. | Reject | Outside the confirmed scope of patient/practitioner/appointment management for a simple first version. | No stakeholder or client statement supports it; flagged as a possible future-stage feature instead. |
| NFR-01's response-time target is untestable without specifying a dataset size. | Evidence-based — a valid point about testability, not a new client requirement. | Accept (modified) | "Responsive" needed a measurable condition; the requirement was tightened rather than replaced. | Compared against the requirement wording; fixed by adding "for up to a few hundred patients," which is now testable. |
| The system should include an AI chatbot to answer patient questions. | Not evidence-based — no stakeholder asked for this. | Reject | Clearly outside scope, and the client explicitly said they want a simple system. | No evidence in the client brief or stakeholder table; rejected without further testing. |

Two of five were accepted, two rejected, one modified — deliberately not a rubber stamp. The two accepted suggestions (#2 and #4) share a pattern worth noticing: both pointed at something already implied by the client's own words (the duplicate-booking problem; the word "responsive") rather than adding anything new. The two rejected suggestions (#3 and #5) both added a feature with no stakeholder behind it at all — that's the actual test used throughout: not whether a suggestion sounds useful, but whether it traces back to evidence already in the brief.

## Part H - Finalise SmartCare v0.2

The final submission combines Parts B-E (see `Stage_2_Lab_AI_OFF.md`) with the AI review evidence above into `Stage_2_SmartCare_v02.md` — stakeholder analysis, scope, 12 FRs, 6 NFRs, 6 user stories, acceptance criteria (including one negative scenario), assumptions/open questions, and the AI review record.

## Reflection (150-250 words)

I wrote the stakeholder list, scope, functional and non-functional requirements, user stories and acceptance criteria before asking AI for anything, based only on the client brief's stated problems: duplicate bookings, hard-to-find records, inconsistent status, and poor visibility of practitioner availability. Reviewing my own draft afterwards, the AI review noticed something I'd missed: FR-04 only stopped a practitioner being double-booked, not a patient being double-booked in the same slot, even though both come from the same "duplicate bookings" problem the client actually described.

Where the AI overreached was suggesting new scope entirely off its own initiative: SMS reminders and an AI chatbot for patient questions. Neither is grounded in anything the client said, so I rejected both rather than folding them in just because they sounded plausible and helpful.

The one requirement that changed after review was NFR-01. The AI pointed out that "responsive" had no measurable condition, so it couldn't actually be tested. I kept the requirement but added a concrete dataset size so it has a real pass/fail condition.

Requirements need evidence because a specification isn't just a list of good ideas — it's a contract for what gets built and, eventually, tested against. A suggestion that "sounds right" but traces back to nothing a stakeholder said is scope invented by the engineer (or the AI), not scope the client asked for, and it's the engineer who is accountable if that invented feature turns out to be wasted effort or, worse, actively wrong for the clinic.


