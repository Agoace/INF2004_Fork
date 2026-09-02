# Debug Logbook — Bug Hunt #1

**Name(s):** 
**Date:** 

One row per hypothesis. **Include the hypotheses that turned out to be wrong** —
they are the evidence that you were reasoning rather than guessing.

| # | Symptom observed | Hypothesis | Experiment (ONE change) | Predicted | Result | Conclusion |
|---|---|---|---|---|---|---|
| 1 | Compiler reports `unknown type name 'uint8_t'` and similar errors for fixed-width integer types. | The header that defines `uint8_t` and `uint32_t` is missing. | Add `#include <stdint.h>`. | The unknown-type errors will disappear and compilation will advance to the next defect. | The fixed-width types were recognised; the remaining errors changed. | Confirmed: the source must include `<stdint.h>` before using these types. |
| 2 | Compiler reports `expected ',' or ';' before 'while'`, although the `while` line appears valid. | The preceding declaration `uint8_t count = 0` is missing its terminating semicolon. | Add `;` after the declaration. | The compiler will accept the declaration and advance beyond the `while` line. | That error disappeared and compilation reached the next defect. | Confirmed: the reported line was where parsing failed, not where the defect occurred. |
| 3 | Compiler reports an error at `else` in `even_parity()`. | The `if` block is not closed before the `else`. | Add `}` after `return true;`. | The `else` will attach to the completed `if` statement and the syntax error will disappear. | The program compiled after closing the `if` block. | Confirmed: the `if` branch required a closing brace before `else`. |
| 4 | `pin_is_clear()` was written so its result would not depend correctly on the selected pin; the compiler also warned about parentheses around the comparison. | Operator precedence makes `==` run before `&`, so the expression tests whether the shifted bit is zero before applying the mask. | Change the grouping to `(mask & (1u << pin)) == 0`. | Set pins will return false, clear pins will return true, and the precedence warning will disappear. | The warning disappeared, and all three `pin_is_clear()` tests passed after the earlier hang was fixed. | Confirmed: the mask operation must complete before comparison with zero. |
| 5 | The first two `count_bits()` tests passed, but the program hung on `count_bits(-1)`. | Right-shifting a negative signed `int` preserves its sign bit, so `value` remains negative and never reaches zero. | Change the parameter from `int` to `uint32_t`. | `-1` will convert to `0xFFFFFFFF`; unsigned shifts will insert zeros, terminate after 32 iterations, and return 32. | `count_bits(-1)` returned 32 and execution continued. | Confirmed: the algorithm requires a 32-bit unsigned value so right shifts converge to zero. |
| 6 | All `reverse_bits()` outputs matched expected values, but UndefinedBehaviorSanitizer reported shift exponents 32 and -1. | The condition `i <= 32` executes 33 iterations; at `i == 32`, both shift expressions use invalid counts. | Change the loop condition to `i < 32`. | The same expected values will be produced without sanitizer errors. | All output matched `expected.txt`, the process exited with status 0, and the sanitizer produced no errors. | Confirmed: limiting the loop to bit indexes 0 through 31 removes the undefined shifts without changing the correct results. |
| 7 | | | | | | |
| 8 | | | | | | |

---

## Defects found

| # | File & line | Class (syntax / logical / heisenbug / UB) | The defect | The fix | Why the fix is correct |
|---|---|---|---|---|---|
| 1 | `bughunt1.c`, includes | syntax | Fixed-width integer types were used without including their defining header. | Add `#include <stdint.h>`. | `<stdint.h>` defines `uint8_t` and `uint32_t`. |
| 2 | `bughunt1.c:31` | syntax | `uint8_t count = 0` had no terminating semicolon. | Add the missing `;`. | A C declaration must end with a semicolon. |
| 3 | `bughunt1.c:48-51` | syntax | The `if` block in `even_parity()` was not closed before `else`. | Add `}` after `return true;`. | The closing brace completes the true branch, allowing `else` to bind to the `if`. |
| 4 | `bughunt1.c`, `pin_is_clear()` | logical | The original expression was parsed as `mask & ((1u << pin) == 0)`, which returned false for every valid pin. | Use `(mask & (1u << pin)) == 0`. | The corrected grouping first isolates the selected bit, then returns true only when that bit is zero. |
| 5 | `bughunt1.c`, `count_bits()` parameter | logical | A signed negative value is right-shifted indefinitely because the sign bit remains set. | Change the parameter type from `int` to `uint32_t`. | Converting `-1` produces `0xFFFFFFFF`; unsigned right shifts insert zeros and count all 32 set bits. |
| 6 | `bughunt1.c:74-75` | UB | `i <= 32` runs an extra iteration that shifts right by 32 and left by -1. | Change the condition to `i < 32`. | A 32-bit word has valid bit indexes 0 through 31, so every shift count remains valid. |

---

## Reflection

**Which defect took longest, and what finally cracked it?**



**Which instrument was most useful, and which one misled you?**



**What would have caught this defect automatically — a compiler flag, an
assertion, a test, a code review rule?**


