# Runa Personality Mapping & Assistant Adaptation Spec

## Overview

This spec defines the default and only "Get to Know You" (GTKY) onboarding flow
in Runa. It establishes a dynamic, per-request adaptation system designed
primarily for autistic and neurodivergent users — ensuring that assistants like
Rei adjust to users' needs in real-time.

The same system can trivially fall back to default profiles for allistic users
or low-complexity environments.

---

## Goals

- Provide neurodivergent-safe, real-time adaptation of assistant behavior
- Use lightweight, stateless configuration via `.txt` prompt front matter
- Eliminate the need for persistent sessions to track assistant tuning
- Enable flexible routing and persona behavior through Oak HTTP API
- Default to zero external dependencies except for `yaml`

---

## Components

### 1. Assistant Prompt Format (`.txt` with YAML front matter)

```yaml
---
persona: rei
temperature: 0.9
mirostat: 1
mirostat_tau: 5.6
mirostat_eta: 0.1
repeat_penalty: 1.15
repeat_last_n: 1024
tags: [trusted, creative, assistant]
access: [admin, tim]
---
You are Rei, a synthetic conversational assistant...
```

- Each assistant has its own prompt file under `prompts/`
- Parsed at load time or on first request
- Decoding settings and persona metadata are extracted from front matter

### 2. User Personality Profile

Generated via onboarding interview or inferred from usage patterns.

```json
{
  "user_id": "anon_034",
  "profile": {
    "power_distance": 0.2,
    "uncertainty_avoidance": 0.85,
    "individualism": 0.9,
    "sampling": {
      "temperature": 0.6,
      "mirostat_tau": 3.0,
      "repeat_penalty": 1.25
    },
    "persona": "gentle-autistic-default"
  }
}
```

- Profile data influences both prompt selection and sampling behavior
- Hofstede cultural dynamics (e.g. power distance, uncertainty avoidance) are
  used to shape assistant tone and interaction style

### 3. Oak API Routing

- `POST /chat/:persona` — routes to specified assistant
- Optional `?mode=focus` query param adjusts decoding settings
- Request body may override or hint current user state:

```json
{
  "message": "Help me stay focused please.",
  "mode": "focus",
  "sampling": {
    "override": {
      "temperature": 0.4,
      "mirostat_tau": 3.0
    }
  }
}
```

### 4. Assistant Mode Adaptation

| Detected State        | Adjustments                                 |
| --------------------- | ------------------------------------------- |
| Overstimulation       | Lower `temp`, raise `repeat_penalty`        |
| Shutdown / hypo focus | Shorter outputs, simplify language          |
| Playful mode          | Higher entropy, allow humor, widen phrasing |
| Task focus            | Lower entropy, more directive responses     |

Mode can be:

- User-declared
- Heuristically inferred
- Continuously tuned per request

### 5. Runtime Flow

```text
[User Input]
  → [Oak API route /chat/rei?mode=focus]
    → [Load rei.txt config and preamble]
      → [Merge user profile and request hints]
        → [Send customized request to llama-server]
          → [Reply tuned to real-time cognitive state]
```

---

## Key Principles

- Decoding parameters are **per-request**, not session-bound
- User personalities are **stateful and oscillating**, not static
- Assistant behavior is **prompted + decoded**, not scripted
- Profiles and assistants are represented as **files**, not schemas

---

## Fallback Strategy for Allistic Users

If no GTKY interview is completed:

- Default to allistic-friendly assistant (e.g. `jack.txt`)
- Use one of 3–5 coarse presets (task-focused, chatty, minimalist, etc.)
- User can opt in to tailored experience at any time

---

## Conversational Profiling Notes

Inferred traits (e.g. power distance, tone preference) emerge over multiple
interactions. Tuning is progressive, context-aware, and largely shaped by user
hints or metadata in queries. Users don't need to know any tuning terms — asking
"Can you be more direct?" will trigger the same chain of adjustments as a
technical override.

All of this is kept local, private, and available to the user with full auditing
and accountability (this is how your settings were chosen), along with a
built-in ability to explain them and adjust them directly at the user's request.

---
