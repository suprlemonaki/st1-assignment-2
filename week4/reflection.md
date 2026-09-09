# Part H - Reflection (150-250 words)

Before using AI, I built the SmartCare prototype by hand: a booking function that stores patient name, practitioner name and appointment time in a dictionary, appended to a list, plus a display function to print them. I ran it, then tested it with a blank patient name, a duplicate practitioner/time booking, and None values, and found the code only checked for a blank patient name — everything else went through silently.

AI helped me see this more clearly by explaining what the code did line by line and naming limitations I'd already partly noticed but hadn't tested for yet, like the missing practitioner/time validation. It also made assumptions of its own: the AI-generated alternative assumed every input would already be valid, so it accepted blank names, None values, and double-bookings without complaint.

I verified this by actually running both versions against the same five test cases rather than trusting either one's docstrings or comments. That's how I confirmed the AI version's lack of validation, not from the AI's own description of itself.

The engineering work that remained mine: deciding which single improvement mattered most for a clinic (rejecting a double-booked practitioner, over a generic None check), writing and testing that fix, and judging that the AI version, while readable, wasn't safe to use as-is without the validation the human version had.