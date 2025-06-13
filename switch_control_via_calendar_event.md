# Automation Explanation

## What does the automation do?

**The automation controls a smart plug based on a calendar event.**

- The plug turns **on** at the start of a specific calendar event (e.g., "Watering").
- The plug turns **off** after the event ends, but only if it was turned on by the automation, or exactly at the event’s end even if it was turned on manually.
- If you manually turn the plug **off** during the event, the automation won’t turn it back on.
- If you manually turn the plug **on** outside the event time, the automation won’t turn it off.

---

## How does the automation work?

### 1. Trigger

The automation triggers whenever the calendar state changes — meaning when an event starts or ends.

---

### 2. Condition on the event title

It checks if the event that changed state matches the specific event name you want to respond to (like "Watering"). If not, it does nothing.

---

### 3. Actions — what happens next

The actions split into two branches depending on whether the event is **active (`on`)** or **finished (`off`)**.

---

#### a) When the event **starts** (state `on`):

- It checks if the plug is currently **off**.
- It checks if the plug was **not manually turned off** during this event (using an `input_boolean` flag).
- If both are true, it **turns the plug on** and sets a flag (`input_boolean`) indicating the plug was turned on by the automation.

---

#### b) When the event **ends** (state `off`):

- If the automation had turned the plug on (flag is `on`), it **turns the plug off** and resets all flags.
- If the plug was turned on **manually** (automation did not turn it on), but it’s still on, it will **turn off the plug only if the current time matches exactly the event’s end time**.
- It also resets the manual-off flag.

---

### 4. Additional automation for manual turn-off during event

Since the main automation doesn’t track manual turn-offs, there’s a separate helper automation that:

- Detects when you manually turn the plug off during an active event,
- Sets a flag indicating manual turn-off,
- Resets the automation-on flag (because automation should no longer turn the plug back on).

---

## Why do we need this?

Without these flags and checks, the automation could:

- Turn the plug back **on at event start even if you manually turned it off** (which you might not want).
- Turn the plug **off when you manually turned it on outside the event**.
- Make the plug behave unexpectedly, causing confusion.

---

## Summary table

| Event State   | Plug State                   | Automation Action                                      |
|---------------|------------------------------|-------------------------------------------------------|
| Event starts  | Plug off, not manually turned off | Turn plug on, set automation-on flag              |
| Event starts  | Plug off, manually turned off     | Do nothing                                          |
| Event running | Plug manually turned off           | Set manual-off flag, automation won’t turn plug on |
| Event ends    | Plug turned on by automation       | Turn plug off, reset flags                          |
| Event ends    | Plug turned on manually             | Turn plug off only exactly at event end time       |

---
