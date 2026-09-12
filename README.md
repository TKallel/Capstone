# Black-Box Optimization Capstone

Optimizing eight unknown functions using Bayesian optimization, with one guess allowed per function per week.

## What is this project?

Imagine you are given a machine with a few dials on it. You turn the dials, press a button, and the machine gives you back a single score. You cannot open the machine. You cannot see how it works. All you can do is try different dial settings and see what score comes out.

Your goal is to find the dial settings that give the highest possible score.

Now add one more rule: **you only get to try one setting per week.** Every guess is expensive, so you need to make each one count.

That is a **black-box optimization** problem, and this project solves eight of them at once.

### Why does this matter?

This is not just a puzzle. It is exactly what happens in real life when:

- **Tuning a machine learning model** — each training run takes hours or days
- **Designing a drug** — each lab experiment costs money and time
- **Running a factory or warehouse** — you cannot shut it down to test every layout
- **Developing a recipe or material** — each batch takes time to make and test

In all of these cases you cannot brute-force every option. You need a smart way to decide what to try next.

---

## The setup

| Item | Detail |
|---|---|
| Number of functions | 8 |
| Input values | Continuous numbers between 0 and 1, to six decimal places |
| Dimensions (number of dials) | 2 up to 8, depending on the function |
| Starting data | Around 10 known points per function (varies) |
| Output | One number — the score for that input |
| Queries allowed | One per function, per week |
| Goal | Maximize the score |

### What makes it hard

- **Very little data.** Ten points is almost nothing when you have eight dials to set.
- **Slow feedback.** You wait a full week to find out whether your guess was good.
- **No structure to lean on.** You do not know if the function is smooth, bumpy, or has many peaks.
- **Noise.** Some functions give slightly different answers for the same input.
- **Growing difficulty.** The 8-dimensional function is far harder than the 2-dimensional one.

---

## The core idea: explore vs exploit

Every week you face the same decision:

> **Exploit** — try something near the best point you have found so far, hoping to squeeze out a slightly better score.
>
> **Explore** — try somewhere you know nothing about, hoping to discover a much better region you have completely missed.

Exploit too much and you polish a small hill while a mountain sits unexplored nearby. Explore too much and you wander forever without ever refining a good answer.

The whole project is about managing this trade-off intelligently, and shifting the balance as more data arrives.

---

## How the approach evolved

### Week 1 — Eyeballing it

I started by plotting the data and picking points that looked promising by hand.

This worked for getting a feel for the problem, but it fell apart quickly. You cannot visually inspect an 8-dimensional space, and "this looks good" is not a repeatable method. There was also no way to measure how confident I should be about any region.

### Week 2 onwards — Bayesian optimization

I switched to a proper framework with two pieces:

**1. A surrogate model — the stand-in**

Since the real function is too expensive to query, I built a cheap imitation of it from the data I already had. This is called a **surrogate model**. I used a **Gaussian Process (GP)**.

The useful thing about a GP is that it gives you two answers for any input:

- **A prediction** — what score it thinks you would get
- **An uncertainty** — how sure it is about that prediction

The uncertainty is what makes exploration possible. The model can effectively say *"I have no idea what happens over here"* — and that is a signal worth acting on.

**2. An acquisition function — the decision rule**

This turns the surrogate's predictions into an actual choice. I used two:

- **UCB (Upper Confidence Bound)** — score each candidate as `prediction + β × uncertainty`. A large β means explore more. Used early, when data was thin.
- **EI (Expected Improvement)** — asks *"how much better than my current best is this point likely to be?"* Used later, and preferred for noisy or multi-peaked functions because it does not chase regions that are merely uncertain.

---

## What I learned function by function

Each function needed different handling. This is the most important takeaway from the project: **there is no single configuration that works everywhere.**

### Function 1 (2D) — Fix the data before blaming the model

My first model did not work. I assumed the problem was the model and kept adjusting it. It was not.

Almost every output value was microscopically close to zero — think 0.0000000000000001 — except two values that were noticeably larger. To the model, all those tiny values looked identical to each other and identical to zero. There was no pattern to learn.

**The fix:** a signed log transform, which squashes a huge range of numbers into a manageable one while keeping the positive/negative distinction intact. After the transform the two meaningful points stood out clearly, and the model immediately started targeting the right region.

**Lesson:** when a model fails, check whether the data is even readable by it first.

### Function 2 (2D) — Choosing the right decision rule

The description hinted the function had multiple peaks and you could get stuck in the wrong one. I switched from UCB to Expected Improvement, which handles this better — it naturally abandons a region once that region is well understood, instead of circling it forever.

### Function 3 (3D) — Minimizing by maximizing

The goal was to reduce drug side effects. Since the output was already expressed as a negative value, maximizing it directly is the same as minimizing harm. No extra work needed — but worth confirming rather than assuming.

### Functions 4 and 5 (4D) — Taming extreme ranges

Function 4's outputs ranged from about -48 to -4, and Function 5's from -3.8 all the way up to 1269. A single model cannot fit both the enormous values and the tiny ones well at the same time — it ends up doing a poor job on at least one end.

Log transforms solved this in both cases. Function 5 needed an extra step: shifting all values above zero first, since you cannot take the log of a negative number.

These two functions also differed in character. Function 4 was described as having many local optima, so I pushed exploration harder. Function 5 was described as having a single peak, so I let the model settle and refine instead.

### Function 6 (5D) — Sometimes do nothing

The outputs sat in a narrow, well-behaved range. No transform was needed. Adding one would have been unnecessary complexity.

**Lesson:** techniques are tools, not rituals. Apply them when the data asks for them.

### Function 7 (6D) — Rare successes among many failures

Most configurations scored near zero, with only a handful performing well. Untransformed, the model would treat the few good results as flukes and ignore them. A log transform brought the successes into proportion so the model could learn from them properly.

### Function 8 (8D) — Changing the model entirely

This is where the Gaussian Process stopped being the right tool.

With 43 data points spread across 8 dimensions, the space is almost entirely empty — every point is far away from every other point. A GP works by fitting a single "length-scale" parameter describing how quickly the function changes across space, and with data this sparse there simply is not enough signal to fit it reliably. This is the **curse of dimensionality**.

**The replacement — two models working in sequence:**

1. **An SVM classifier** looks at all 43 points and splits the space into "promising" and "poor" regions, using the median score as the cutoff. It does not need to estimate uncertainty or fit a kernel, so high dimensions do not break it. This narrows the search down to about half the candidates.

2. **A neural network ensemble** — eight small neural networks, each trained slightly differently — then models the score surface *within* the promising region only. Where the eight networks agree, confidence is high. Where they disagree, that disagreement becomes the uncertainty estimate that drives exploration.

The next point is then chosen in two stages: first keep only candidates the SVM considers promising, then among those pick the one maximizing `ensemble mean + 0.2 × ensemble disagreement`.

Worth being honest here: the neural network ensemble is doing most of the work. The SVM filter helps, but it is a useful addition rather than the main reason this approach outperforms the GP at this dimensionality.

---

## Summary table

| Function | Dimensions | Model | Key adjustment |
|---|---|---|---|
| 1 | 2D | GP + UCB | Signed log transform to expose tiny values |
| 2 | 2D | GP + EI | EI to handle multiple peaks |
| 3 | 3D | GP + EI | No transform; maximize the negative directly |
| 4 | 4D | GP + EI | Log transform; strong exploration for many local optima |
| 5 | 4D | GP + EI | Shift-then-log; moderate exploration for a single peak |
| 6 | 5D | GP + EI | No transform — the data was already well behaved |
| 7 | 6D | GP + EI | Log transform to amplify rare high scores |
| 8 | 8D | SVM + NN ensemble | GP replaced entirely; UCB on ensemble disagreement |

---

## Key takeaways

**Check the data before tuning the model.** The hardest problem in this project (Function 1) was a data scaling issue, not a modeling issue. I lost time adjusting kernels when the real fix was a transform.

**Match the method to the problem.** The same Gaussian Process that worked beautifully in 2D was unreliable in 8D. Knowing when your tool stops working is as valuable as knowing how to use it.

**Read the problem description.** Hints like "many local optima" or "a single peak" directly determined how much exploration I configured. Free information, easily ignored.

**Uncertainty is not a nuisance — it is the plan.** Knowing what you do not know is what tells you where to look next.

**Simplicity has value.** Function 6 needed no transform at all. Adding one would have been effort spent making things worse.

---

## Repository structure

```
.
├── README.md
├── data/                 # Observations for each function
├── scripts/              # One optimization script per function

```

## Requirements

```
numpy
scipy
scikit-learn
matplotlib
```

