---
name: fc-edit-requires-permission
description: Do not edit fc.py (flight-controller code) without explicit permission; show the diff first
metadata: 
  node_type: memory
  type: feedback
  originSessionId: aa4bfd5d-c6b3-4f32-ac00-4dca6460bfe2
---

`../fc/src/fc.py` (→ `fc-current.py`) is the user's rocket flight-controller code. It lives outside the `rdwr_vn100` repo we've been working in.

**Why:** it's outward-facing flight software; an unreviewed change is high-stakes. The user has been validating each step deliberately.

**How to apply:** propose `fc.py` changes and **show the diff before applying**; get explicit go-ahead before writing. Prefer proving things in the `rdwr_vn100` sandbox first, then porting. Related: [[rdwr-vn100-project]].
