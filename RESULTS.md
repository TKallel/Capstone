# Results

**5 of 12 submission rounds were completed.** The project allowed one query per
function per week across twelve rounds; five were made. Everything below
therefore describes an unfinished search rather than a converged one.

---

## Where each function ended up

| Function | Inputs | Points | Best at start | Best after 5 rounds | Change |
|---|---|---|---|---|---|
| 1 | 2 | 15 | 7.71e-16 | 7.71e-16 | — |
| 2 | 2 | 15 | 0.6112 | 0.6112 | — |
| 3 | 3 | 20 | −0.0348 | −0.0348 | — |
| 4 | 4 | 35 | −4.026 | −3.789 | **+5.9%** |
| 5 | 4 | 25 | 1088.9 | 2757.0 | **+153.2%** |
| 6 | 5 | 25 | −0.7143 | −0.3227 | **+54.8%** |
| 7 | 6 | 35 | 1.365 | 2.195 | **+60.8%** |
| 8 | 8 | 45 | 9.5985 | 9.5985 | — |

Four of the eight improved. Four did not.

---

## Progress round by round

Which rounds produced a new best:

| Function | R1 | R2 | R3 | R4 | R5 |
|---|---|---|---|---|---|
| 1 | — | — | — | — | — |
| 2 | — | — | — | — | — |
| 3 | — | — | — | — | — |
| 4 | ✓ | — | — | — | — |
| 5 | — | — | ✓ | ✓ | ✓ |
| 6 | — | — | — | — | ✓ |
| 7 | ✓ | ✓ | — | ✓ | ✓ |
| 8 | — | — | — | — | — |

**Function 7** was the most consistent, improving in four of five rounds.
**Function 5** produced the largest gain and was still accelerating when
submissions stopped — its last three rounds added +181, +351 and +1137.
**Function 6** was flat for four rounds and then jumped 55% in the fifth.

---

## Why four functions stalled

Each model was tested by hiding one observation, refitting on the rest,
predicting the hidden point, and repeating for every point in turn.

Two measures are reported. **RMSE** is the average size of the prediction
error, in the units of that function's target — an absolute measure. **R²** is
the same information expressed as a proportion, from 1.0 (perfect) through 0.0
(no better than always guessing the average) to negative (worse than guessing).

| Function | RMSE | RMSE if guessing the mean | Ratio | R² | Improved? |
|---|---|---|---|---|---|
| 3 | 0.117 | 0.112 | **1.04** | −0.087 | no |
| 1 | 94.60 | 94.84 | **1.00** | 0.005 | no |
| 2 | 0.213 | 0.225 | **0.94** | 0.110 | no |
| 5 | 1.131 | 1.915 | 0.59 | 0.651 | yes |
| 6 | 0.335 | 0.591 | 0.57 | 0.679 | yes |
| 7 | 0.167 | 0.351 | 0.48 | 0.774 | yes |
| 4 | 0.177 | 0.572 | **0.31** | 0.904 | yes, once |

RMSE alone cannot be compared across functions, because each target sits on a
different scale — Function 1's RMSE of 94.6 looks enormous but its values span
roughly −248 to −2. The **ratio** column fixes this by dividing each model's
error by the error of simply predicting the mean. Below 1 means the model beats
guessing; above 1 means it does worse.

**Every function with a ratio near or above 1 failed. Every function below 0.6
improved. No exceptions.**

Function 3's model is wrong by 0.117 on a target whose whole range is 0.36 — and predicting the mean would
have been *more* accurate. Function 4's is wrong by 0.177.

The three failing models belong to the three smallest data sets — 15, 15 and 20
points. They could not tell one region of the space from another, so the search
had nothing to follow and drifted to the corners.

Function 8 has no score here because it uses a different kind of model. The model was barely less certain
about places it had never seen, so three of its five rounds re-measured
almost the same point.

**Function 4 is the exception worth noting.** It has the best model of all
eight and still stalled — the exploration setting was too aggressive, sending
four of five queries to the edges of the search space where the scores were ten
times worse.

---

## What the five rounds showed

**Boundary queries almost never worked.** Fifteen of the forty queries had at
least one coordinate at exactly 0 or 1. Only **one of those fifteen** produced
a new best. Of the twenty-five queries that stayed in the interior, **eight**
did.

The reason is in how the candidate list is built. Each script creates its pool
of candidate points with `np.linspace(0, 1, n_grid)` along every axis, so 0 and
1 are always on the menu, and exact corners are always available. The model
then picks the best candidate from whatever pool it is given — it cannot
suggest a point that is not in the list. 

**Improvement clustered.** The two functions that improved most show
consecutive queries in one tight region, each refining the last. The four that
failed show queries scattered with no continuity between rounds.

**The searches were cut short.** Three of the four successes recorded their
best value in round 5 — the last round completed, with seven still available.
Equally, five rounds is too few to conclude that the four stalled functions are
genuinely resistant to this approach.

---

## Rebalancing exploration and exploitation

Every round came down to one choice: refine near the best point found so far
(**exploitation**), or try somewhere unknown that might be better
(**exploration**). Each function had a setting controlling that balance.

| Function | Setting | Changed mid-project? | Improved? |
|---|---|---|---|
| 1 | β = 1.96 | no | no |
| 2 | ξ = 0.01 | no | no |
| 3 | ξ = 0.05 | no | no |
| 4 | ξ = 0.10 | no | once |
| 5 | ξ = 0.05 → **0.01** | **yes** | yes, 3 rounds |
| 6 | ξ = 0.05 → **0.01** | **yes** | yes |
| 7 | ξ = 0.05 → **0.01** | **yes** | yes, 4 rounds |
| 8 | β = 0.2 | no | no |

**The three functions where the setting was changed are the three that improved
most.** Functions 5, 6 and 7 each started balanced, found a promising region,
then tightened to exploit it. Every function left at a fixed value stalled or
improved only once.

The lesson is not that one value is better than another. It is that the right
balance changes as the search goes on, and a value chosen at the start will be
wrong for most of the run.

**Function 4 explored too hard**
— ξ = 0.10 sent four of five queries to the edges of the space, even though it
had the best model of the eight and plenty of good information to use.
**Function 8 exploited too hard** — β = 0.2 meant rounds 1, 3 and 4 measured
effectively the same point, then round 5 leapt to a corner.

**Functions 1 and 3 only looked like they were exploring.** Their models could
not tell one region from another, so every unknown point seemed equally
uncertain and the search just picked whichever candidate was furthest from the
data — always a corner. 

## What I would do differently

Tie the balance to the model rather than to the problem description:

- **While the model cannot predict** (RMSE at or above the guessing baseline),
  ignore the acquisition function and spread points to build coverage. There is
  nothing yet to exploit and no informed exploration to be done.
- **Once the model predicts reasonably** (RMSE comfortably below baseline),
  start balanced at ξ ≈ 0.05.
- **After a clear new best**, tighten to ξ ≈ 0.01 and refine — which is exactly
  what was done on Functions 5, 6 and 7, and exactly what worked.
- **If several rounds pass with no improvement**, loosen again rather than
  continuing to exploit a region that has stopped yielding.

**1. Check the model before acting on the point it suggests.**
The validation above takes a few lines to run. On Functions 1, 2 and 3 it would
have shown, after the first round, that the model's predictions were no better
than guessing the average. Rounds 2 to 5 on those functions all submitted the
point the model picked, and all five of those models were unreliable.

**2. When the model is unreliable, choose points to spread coverage instead.**
Functions 1 and 2 had only 10 starting points across a 2D space, and Function 3
had 15 across 3D. A model fitted to that little data cannot rank one region
above another, so following it is close to picking at random. Choosing points
that sit far from everything already measured at least maps the space, and
gives the next model something to learn from.
**3. Build the candidate list differently.**
The model does choose the query — but only from the list of candidates the code
hands it. That list comes from `np.linspace(0, 1, n_grid)` on every axis, which
guarantees the exact corners of the space are always available. Replacing the
grid is required.

**4. On Function 8, stop re-measuring the same point.**
Rounds 3 and 4 landed 0.023 and 0.017 away from an earlier query and returned
9.355 and 9.349 against 9.351 — three measurements of effectively one location.
Note that querying close to a previous point is not wrong in itself: Function 5
round 4 sat 0.056 from round 3 and improved from 1269 to 1620, and Function 7
round 4 did the same. The difference is whether the repeat produces new
information. On Function 8 it did not, because the model there had no working
sense of where it was uncertain.

---

