# Safety & Responsible Use

This document covers the safety features built into the Hypnosis Script Generator and guidelines for responsible use.

---

## Important Disclaimer

This tool is for **self-exploration and personal use only**. It is not a medical device, not a substitute for professional therapy, and not clinically validated. The AI-generated scripts are not reviewed by a licensed hypnotherapist.

**Do not use this tool if you have:**
- Epilepsy or photosensitive seizure disorders
- Dissociative disorders (DID, DPDR, dissociative amnesia)
- Psychosis or schizophrenia spectrum conditions
- Active PTSD with unprocessed trauma (especially with regression methods)
- A history of adverse reactions to hypnosis or meditation

**Do not use while:**
- Driving or operating machinery
- Under the influence of alcohol or drugs
- In a public or unsafe environment
- Supervising children or dependents

---

## Built-in Safety Features

### Content Warnings
Sessions using **advanced methods** (time distortion, hallucination, somnambulism, double induction, full-body anesthesia) or **authoritarian personas** (Commander, Drill Instructor, Dominatrix, Interrogator) trigger an explicit confirmation dialog before starting. The warning describes the potential for intense experiences including dissociation, hallucination, or strong emotional responses. Users must confirm to proceed.

### Emergency End
The **"end & re-alert"** button is always visible during sessions and cannot be hidden. It is also bound to the **Escape** key. When activated:
1. All narration stops immediately.
2. A **persona-matched re-alert script** plays, bringing the user back to full alertness in that persona's familiar voice.
3. The **5-4-3-2-1 grounding exercise** is displayed automatically.
4. A **30-second cooldown** prevents starting another session immediately.

### Grounding Exercise
After an emergency end, the post-session screen displays a structured grounding exercise:
- 5 things you can see
- 4 things you can touch/feel
- 3 things you can hear
- 2 things you can smell
- 1 thing you can taste

This sensory-anchoring technique is a standard clinical tool for re-grounding after dissociative experiences.

### Sleep Intention
When the "Sleep induction" intention is selected, the **emergence phase is suppressed** — there is no count-up, no re-alerting. The session ends in silence. The Configure tab shows a warning explaining this. This prevents unexpected jarring re-alerts for users who intend to fall asleep.

### Visual Safety
- All visual effects cap oscillation frequency at **≤2 Hz** per WCAG 2.3.1.
- No saturated red with opposing transitions.
- The `@media (prefers-reduced-motion)` CSS query disables all animations automatically.
- A manual **"None"** visual option provides a black screen with only the fixation dot.
- The moiré effect uses medium greys (#888/#000) rather than pure black-white to stay within contrast guidelines.

### Wake Lock
`navigator.wakeLock.request('screen')` prevents the device from sleeping during a session. This is critical because a screen timeout during deep trance could leave the user in a state with no guidance to return. The wake lock re-acquires automatically if the tab becomes hidden and then visible again.

---

## Guidance for Specific Categories

### Advanced Methods
Time distortion, positive/negative hallucination, somnambulism, and full-body anesthesia are **deep-trance phenomena** that require prior experience with basic trance. The tool does not enforce this — it is the user's responsibility to build foundational experience before attempting advanced techniques.

### Authoritarian Personas
The Commander, Headmistress, Interrogator, Drill Instructor, Dominatrix, and Surgeon use **direct, commanding language**. This style of delivery can be powerful for users who respond to structure and authority, but it may be distressing for others. Content warnings are shown before sessions using these personas.

### Pain Management
The tool includes pain-related intentions (Pain management/comfort) and methods (Glove Anesthesia, Pain Dial). These are **complementary techniques** — the AI is instructed to always include language like "This works alongside your medical care, not instead of it." Hypnotic analgesia should never be used to mask symptoms that require medical attention.

### Regression
The Age Regression method is included for accessing **resource states** (childhood wonder, past confidence). It is explicitly **not designed for trauma work**. The method description states: "Not for trauma work (that requires a live therapist)."

---

## What the AI Does NOT Do

- The AI does not have access to any personal information about the user.
- The AI does not adapt based on real-time physiological signals.
- The AI does not make clinical assessments or diagnoses.
- The AI does not store or learn from previous sessions.
- Each script generation is independent — the AI has no memory between calls.

---

## Data Privacy as a Safety Feature

All data stays on the device. There is no server, no account, no cloud sync. This means:
- No one can access your session history, mood data, or custom personas remotely.
- Shared config URLs contain only the session configuration (method, persona, intention text) — not scripts, history, or personal data.
- Backup files are plain JSON on your filesystem — they go where you put them.

This architecture means there is no external attack surface and no data breach risk beyond physical device access.

---

## If Something Feels Wrong

1. **Press Escape or click "end & re-alert"** — this is always available.
2. **Follow the grounding exercise** that appears after emergency end.
3. **Open your eyes, stand up, move around** — physical movement re-engages the body.
4. **Drink water** — a simple orienting action.
5. **Wait for the 30-second cooldown to pass** before considering another session.
6. If you feel persistently dissociated, disoriented, or distressed after a session, contact a mental health professional.

---

## Reporting Safety Issues

If you encounter a situation where the tool behaves unsafely — emergency end fails, content warnings don't appear, or the grounding exercise doesn't display — please report it as a bug with console logs. Safety features are the highest-priority fix category.
