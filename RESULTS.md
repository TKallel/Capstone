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

Read as absolute error, the failures are stark. Function 3's model is wrong by
0.117 on a target whose whole range is 0.36 — and predicting the mean would
have been *more* accurate. Function 4's is wrong by 0.177 on a range of 2.5.

The three failing models belong to the three smallest data sets — 15, 15 and 20
points. They could not tell one region of the space from another, so the search
had nothing to follow and drifted to the corners.

Function 8 has no score here because it uses a different kind of model. Its
equivalent check showed the same problem: the model was barely less certain
about places it had never seen, so three of its five rounds re-measured
almost the same point.

**Function 4 is the exception worth noting.** It has the best model of all
eight and still stalled — the exploration setting was too aggressive, sending
four of five queries to the edges of the search space where the scores were ten
times worse.

---

## What the five rounds showed

**Boundary queries almost never worked.** Any query with a coordinate pinned at
exactly 0 or 1 came back poor, across every function. Interior queries did
consistently better. This traces back to generating candidates on a uniform
grid, where the points furthest from any measurement are always at the edges.

**Improvement clustered.** The two functions that improved most show
consecutive queries in one tight region, each refining the last. The four that
failed show queries scattered with no continuity between rounds.

**The searches were cut short.** Three of the four successes recorded their
best value in round 5 — the last round completed, with seven still available.
Equally, five rounds is too few to conclude that the four stalled functions are
genuinely resistant to this approach.

---

## What I would do differently

1. **Check the model before trusting it.** The validation above takes a few
   lines and would have flagged Functions 1, 2 and 3 as unreliable before four
   rounds were spent following their recommendations.
2. **Spend early rounds on coverage when data is thin.** With 15 points, a
   query filling a gap is worth more than one following a model that cannot
   predict.
3. **Don't generate candidates on a uniform grid.** It biases the search toward
   the boundary, which is where most of the wasted queries went.
4. **Rule out repeat queries.** With one evaluation a week, re-measuring a known
   point wastes a round.

---

Full detail per function is in the notebooks. See `MODEL_CARD.md` for the
method and its limitations, and `DATASHEET.md` for the data.
