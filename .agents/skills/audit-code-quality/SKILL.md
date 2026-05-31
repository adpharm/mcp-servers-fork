---
name: audit-code-quality
description: Audit the quality of code to optimize for our 4 pillars of engineering.
---

Please audit your implmentation for adherence to our very important engineering pillars:

1. Simple (not necessarily "easy", because often complex = easy and simple = harder)
2. Best-practice-following (no cheap wins or "we can do that later")
3. DRY (minimize code duplication, SSOT where possible)
4. Type-safe (let typescript relieve the maintanance burden, type-casting only as a last resort, definitely no `as any`, leverage /typescript-advanced-types skill where applicable)

Note:

- If your implementation for extensive in nature, use agents as you see fit.
- The implementation you're auditing may already have been audited and fixed. You must not assume that it has not been fixed, nor that it has been fixed.
- If the fixes you suggest are unequivocally correct, just do it.
