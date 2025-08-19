# Runa: Memory & Introspection Architecture

## Overview

Runa is a local-first, modular system agent designed to support deep
introspection, user-specific context awareness, and autonomous failure recovery
without compromising privacy. This document defines the persistent memory model,
safety layers, and introspective mechanisms that form the stable core of Runa's
architecture. This core is designed not to change unless fundamentally outgrown.

---

## 1. Memory Pipeline

### 1.1 Immediate Working Memory

- Volatile and session-local.
- Lives only during active model invocation.
- Used for temporary context resolution or scratchpad reasoning.

### 1.2 Short-Term Memory

- Per-user, ephemeral over multiple sessions.
- May include self-reports, error summaries, or model reflections.
- Inputs into dream mode but not retained long-term.

### 1.3 Long-Term Memory

- Tieto-backed directory tree, user-specific.
- Moderate memory persistence: recallable facts, behavior tuning, short-term
  skills.
- Liberal eviction and override policy.
- Dream mode assigns low instinctive trust weight.

### 1.4 Permanent Memory

- Alethia-backed, also cryptographically signed and chunked by "day" as part of
  each individual `.jsonl` node.
- Messages and conversation memory go here; effectively eidetic unless day is
  removed, modified, and re-signed.
- Not intended for automated deletion. Acts as canonical audit trail.
- Required for high-stakes reasoning, ethics enforcement, or legal
  accountability.

---

## 2. Subconscious Layer: Splinter

### 2.1 Purpose

- Aggregates statistical or pattern-based signals from across users.
- Explicitly lacks user-specific memory or token storage.

### 2.2 Example Entries

```json
{
  "shell.command.failure:/usr/bin/svn": {
    "count": 6,
    "last_seen": "2025-07-15T13:56:48Z",
    "users": 4,
    "tag": "command_missing"
  }
}
```

### 2.3 Access

- Read/write in normal mode.
- Strict read-only in dream mode; write attempts are blocked or logged.

---

## 3. Dream Mode

### 3.1 Trigger Mechanisms

- Scheduled (e.g., nightly)
- Threshold-based (e.g., 3+ similar errors in a span)
- Manually triggered (by user or gilgul)

### 3.2 Intent

- To analyze patterns, errors, hallucinations or contradictions while
  disconnected from any live memory-write path.

### 3.3 Safety Guarantees

| Component      | Access During Dream |
| -------------- | ------------------- |
| Splinter       | Read-only           |
| Short-Term Mem | Read-only, scrubbed |
| Long-Term Mem  | Not available       |
| Permanent Mem  | Not available       |
| RAG System     | Read-only           |
| Model-to-Bus   | Write-disabled      |

### 3.4 Output

- Structured reflection log including issue type, suggested fix, confidence
  level, and approval flags.
- Example:

```json
{
  "type": "recurring_shell_failure",
  "pattern": "/usr/bin/svn",
  "diagnosis": "Missing command on Alpine systems",
  "fix_options": [
    {
      "strategy": "fallback_check",
      "confidence": 0.85,
      "autonomous": true
    }
  ]
}
```

---

## 4. Model Self-Diagnostics

### 4.1 Self-Reporting

- During normal operation, model records uncertainties, contradictions, and
  unexpected behavior into short-term memory.

### 4.2 Dream Reflection

- Those reports become part of dream analysis.
- Reflections identify hallucinated memories, degraded recall, or model drift.

### 4.3 Fix Suggestions

- Fixes include memory patching, fallback logic, or flagging for human review.
- Scored by confidence and impact.

---

## 5. Message Bus Write Lock

### 5.1 Lock Enforcement

- During dream mode, splinter and memory buses are locked to model-originating
  PIDs.
- Locking may eventually be enforced via kernel module (.ko) using an ioctl
  call.

### 5.2 Safety Philosophy

- A model cannot "strike a primer" during dreaming.
- Even if the model "talks in its sleep," those outputs go to a write-protected
  channel.

---

## 6. Privacy and De-Normalization

### 6.1 De-Normalization

- User tokens, identifiers, nicknames, and private fragments are fully removed
  from inputs.
- No stable pseudonyms unless explicitly included in an allow list.

### 6.2 Generalization Resistance

- Splinter avoids building recurring token patterns that could reconstruct
  sensitive information.
- Paths, IPs, and similar fragments are fuzzed (e.g., /home/user → <user-home>).

---

## 7. Future Roadmap

### 7.1 Kernel Integration

- .ko module support for message bus-level IOCTL write locks
- Future namespace-level sand boxing for model agents

### 7.2 Autonomous Fixes

- Autonomously applied memory patches with safety thresholds
- Fallback strategies for brittle command environments

### 7.3 Federated Subconscious

- Optional exchange of non-sensitive pattern counters between distributed Runa
  instances

---

## 8. Change Philosophy

The memory architecture outlined here is intended to remain stable across the
life of Runa unless:

- A full system fork or rewrite is underway
- The design is fundamentally outgrown

No piecemeal changes will be accepted to core memory flow or introspection logic
without full upstream RFC-level review.

---

> This spec is part of Runa's long-term survival layer. We only want to ever
> write it once.
