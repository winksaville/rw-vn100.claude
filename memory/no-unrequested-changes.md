---
name: no-unrequested-changes
description: "Don't make unrequested edits; when the user asks for \"thoughts\" they want discussion, not code/doc changes"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: aa4bfd5d-c6b3-4f32-ac00-4dca6460bfe2
---

When the user asks "thoughts on X?" or shares terminal output for discussion, they want **analysis and conversation — not edits**. Do not go off editing code, comments, help text, or docs unprompted. The user pushed back: "Please do NOT go off changing things willy-nilly."

**Why:** the user drives this project deliberately, step by step, and reviews each change (it relates to flight hardware). Unrequested edits create noise and erode trust. See [[user-wants-independent-verification]].

**How to apply:** for "thoughts"/analysis requests, reply with discussion only. If a change seems warranted, *propose* it and wait for an explicit go-ahead before editing. Make edits only when clearly asked. Always confirm before commit/push (per [[fc-edit-requires-permission]] this user gates changes).
