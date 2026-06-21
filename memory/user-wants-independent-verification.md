---
name: user-wants-independent-verification
description: "User is appropriately skeptical and wants claims verified independently, not self-reported"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: aa4bfd5d-c6b3-4f32-ac00-4dca6460bfe2
---

The user drives changes step by step and is skeptical of tools "marking their own homework" — e.g. when `rdwr_vn100 bench` both configured and counted the stream and reported 200 Hz, he asked "So I should take your word for it?" He also corrects/double-checks reasoning himself (caught a slice misread, questioned baud assumptions).

**Why:** this is flight hardware (rocket flight controller); wrong conclusions are costly. Healthy skepticism is warranted.

**How to apply:** offer independent verification for empirical claims (e.g. cross-check a measurement with the vendor SDK or a from-scratch pyserial parser, in a different language/codebase than the thing under test). Don't oversell; state confidence honestly and show the raw evidence. Affirm correct understanding, name mistakes plainly. Related: [[rdwr-vn100-project]].
