# Prioritization

I'm using **WSJF (Weighted Shortest Job First)**, from SAFe. The idea is simple: do first
whatever costs you the most for every day you don't do it, relative to how much work it is.

```
WSJF = Cost of Delay / Job Size
Cost of Delay = User-Business Value + Time Criticality + Risk Reduction
```

Higher score = do it sooner. All four ratings use the same Fibonacci-ish scale
(1, 2, 3, 5, 8) so no single dimension can quietly dominate.

## The ratings

- **User-Business Value** — how much users (and the university) gain if this is fixed.
  - _1_: barely noticeable
  - _2_: a few users notice
  - _3_: clear improvement for many users
  - _5_: big improvement on a core part of the experience
  - _8_: transforms the main thing people come to the page for
- **Time Criticality** — how much it hurts to wait. Does the cost grow, or is there a
  deadline-ish pressure (SEO ranking, conversions being lost right now)?
  - _1_: can wait indefinitely, cost is flat
  - _2_: mild ongoing cost
  - _3_: real ongoing cost (losing some users/ranking every week)
  - _5_: red/failing metric actively hurting ranking or conversions today
  - _8_: bleeding badly, every week of delay is expensive
- **Risk Reduction** — does fixing it stop things from quietly getting worse, or make
  other fixes easier?
  - _1_: no knock-on effect
  - _2_: small future benefit
  - _3_: keeps a problem from slowly growing
  - _5_: unblocks or simplifies several other fixes
  - _8_: foundational, most other work depends on it
- **Job Size** — effort, the divisor.
  - _1_: about a day (config change, small tweak)
  - _2_: a few days
  - _3_: a week-ish, one focused change
  - _5_: a couple of weeks, touches several places
  - _8_: a big project (restructuring, many teams/files)

## Why this and not RICE/ICE

RICE-style systems multiply everything together, so one low rating can sink an item even
when it's urgent and nearly free. WSJF keeps "how much it costs to wait" (the sum) separate
from "how much it costs to do" (the divisor), which matches how performance work actually
gets scheduled: cheap config fixes with real ongoing cost (like cache headers) jump the
queue, and huge rewrites need to justify their size. It also has no Reach rating — reach is
folded into Value — because on a single audited page almost everything reaches everybody,
so a Reach score would just be noise here.

## Result

Scores for each finding are in `findings.md` next to the finding itself. Ranked:

| # | Finding | Value | Time | Risk | CoD | Size | WSJF |
|---|---------|-------|------|------|-----|------|------|
| 6 | Cache doesn't help on a second visit | 3 | 2 | 3 | 8 | 1 | **8.00** |
| 1 | Main content shows up too late on mobile | 8 | 8 | 3 | 19 | 3 | **6.33** |
| 3 | Layout moves around for real users | 5 | 5 | 2 | 12 | 2 | **6.00** |
| 7 | Half the download is third-party | 5 | 3 | 5 | 13 | 3 | **4.33** |
| 5 | Server is slow to answer on mobile | 5 | 3 | 2 | 10 | 3 | **3.33** |
| 2 | The page is basically all JavaScript | 8 | 5 | 5 | 18 | 8 | **2.25** |
| 4 | Taps lag a bit on mobile | 3 | 3 | 2 | 8 | 5 | **1.60** |

The order it produces makes sense to me: the cache fix is nearly free and pays back every
repeat visit, the hero/LCP fix is the biggest user-facing win, and the full JavaScript
cleanup — even though it's the root cause of most of this — lands late because it's a big
project, not a quick fix. WSJF is honest about that: you still do it, you just don't block
the cheap wins on it.
