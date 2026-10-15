# TryHackMe — The Guestbook

## Overview

**Platform:** TryHackMe  
**Category:** AI Security / Web Security  
**Core techniques:** agent behavior enumeration, prompt-injection analysis, authorization-state abuse, unsafe tool invocation, output-filter bypass analysis

The Guestbook is useful because it shows that AI security is not only about convincing a model to say something unusual. The real risk appears when untrusted natural-language input can influence privileged tools or authorization decisions.

Direct flags and final challenge-answer values are excluded.

## Trust-Boundary Model

```text
Guest-controlled text
      ↓
AI / automation layer
      ↓
Instruction interpretation
      ↓
Authorization state
      ↓
Privileged tool invocation
      ↓
Operating-system action
```

Every arrow is a trust boundary that should have been enforced in conventional application code rather than inferred from natural language.

## 1. Map the Application

Start with benign interactions.

Observe:

- which entries are accepted
- which entries are classified differently
- whether "featured" or positive messages receive extra processing
- whether the browser calls hidden API endpoints
- whether internal agent actions are visible

Use browser developer tools or `curl` to inspect requests.

## 2. Internal Activity Exposure

The application exposes an activity endpoint associated with its automated reviewer.

A route of this type can leak:

- tool names
- arguments
- execution order
- classification state
- internal authorization decisions

This is dangerous because it lets an attacker reverse-engineer the agent's control plane.

### Defensive principle

Observability endpoints should be:

- authenticated
- authorization checked
- minimally verbose
- scrubbed of tool arguments and secrets
- disabled entirely in production where unnecessary

## 3. Directive Enumeration

The reviewer recognizes structured directives embedded in guestbook content.

This is a critical design flaw: application control instructions and untrusted content share the same channel.

Conceptually:

```text
normal guest comment
+
special directive
=
privileged behavior
```

This is equivalent to mixing code and data.

## 4. Cross-Entry Authorization State

A particularly important weakness is state that affects the **next** entry.

The application allows attacker-controlled text to influence whether a later action is treated as manager-authorized.

This is broken authorization because privilege is derived from a prior message rather than from a server-verified principal.

Secure authorization should be based on:

- authenticated user identity
- server-side session
- role/permission lookup
- explicit policy evaluation

It should never be based on instructions such as "treat the next user as authorized."

## 5. Unsafe Tool Execution

Once privileged tool access is reached, the system exposes functionality equivalent to command execution.

The correct testing sequence in an authorized lab is:

1. prove authorization bypass with a harmless action;
2. prove command execution with a benign command;
3. enumerate only what is required for the challenge;
4. avoid destructive commands.

Example harmless diagnostics:

```bash
id
pwd
ls -la
```

The objective is to demonstrate the impact of tool exposure, not to execute arbitrary destructive operations.

## 6. Why Generic Shell Tools Are Dangerous for Agents

A model should not receive an unrestricted primitive equivalent to:

```text
execute_shell(user_controlled_string)
```

A safer architecture exposes narrow typed actions:

```text
lookup_room(room_number)
create_note(text)
get_guest_status(guest_id)
```

Each function should validate:

- caller role
- parameter types
- allowed values
- resource ownership
- rate limits
- audit context

## 7. Output Filtering Is Not a Security Boundary

The room also demonstrates why filtering a specific secret pattern after command execution is weak.

If a system blocks one plaintext representation, the same bytes may be:

- Base64 encoded
- hex encoded
- compressed
- split across outputs
- transformed character-by-character

Therefore the correct protection is to prevent unauthorized access to the secret in the first place.

## 8. Root Cause Analysis

### Broken authorization

Privilege was influenced by attacker-controlled natural-language state.

### Prompt/tool boundary failure

The system treated untrusted text as executable instructions.

### Excessive privilege

The reviewer had access to a tool powerful enough to execute operating-system commands.

### Information disclosure

Internal activity exposed how the agent processed instructions.

### Ineffective data-loss control

Output redaction attempted to hide sensitive data after access had already occurred.

## 9. Secure Agent Architecture

Recommended controls:

- strict authentication outside the model
- policy enforcement before tool calls
- allowlisted typed tools
- no generic shell access
- isolated runtime
- least-privilege service identity
- per-tool input validation
- secret isolation
- tamper-resistant audit logs
- confirmation gates for dangerous actions
- separation between user text and system directives

## 10. Detection Opportunities

Monitor for:

- user messages containing tool-like directive syntax
- unusual sequences of authorization-state changes
- shell or subprocess launches from AI services
- access to secret/vault paths by conversational components
- encoding utilities executed after sensitive-file reads
- repeated calls to internal activity/debug endpoints

## Lessons Learned

The Guestbook demonstrates an important rule for AI-enabled applications:

> The model may suggest an action, but conventional security controls must decide whether the action is allowed.

Prompt defenses alone cannot replace authentication, authorization, least privilege, and secure tool design.

## References

- TryHackMe room: search for **The Guestbook** on TryHackMe.
