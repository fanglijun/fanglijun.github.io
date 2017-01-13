---
layout: post
title:  "The Open-Closed Principle"
date:   2017-01-12 23:33:58 +0800
categories: my2829
---

> SOFTWARE ENTITIES (CLASSES, MODULES, FUNCTIONS, ETC.) SHOULD BE OPEN FOR EXTENSION, BUT CLOSED FOR MODIFICATION.

### How?
- Abstraction is the Key.
    Give client an interface instead of an implementation.
- Strategic Closure
    It should be clear that no significant program can be 100% closed. The designer must choose the kinds of changes against which to close his design.
- Using a “Data Driven” Approach to Achieve Closure.
    The change part could be considered "data", not program. So, "data" could be put outside the program.

### Heuristics and Conventions based on the open-closed principle
- Make all Member Variables Private.
- No Global Variables -- Ever.
- RTTI is Dangerous. （run time type identification）

