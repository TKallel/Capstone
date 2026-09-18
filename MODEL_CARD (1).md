# Model Card: Adaptive Surrogate-Based Optimization Strategy

### Overview

**Model Name:** Adaptive Surrogate-Based Black-Box Optimization Strategy (v1.0)
**Type:** Gaussian Process regression for Functions 1–7; SVM classifier plus neural network ensemble for Function 8.
The approach fits a cheap stand-in model to the observations so far, scores candidate points on how promising they look, and submits the best one as the next weekly query.

### Intended Use

Suitable for expensive black-box functions in low to moderate dimensions, where only tens of evaluations are affordable and inputs are continuous. It should be avoided when evaluations are cheap, when inputs are discrete, and in safety-critical settings — it offers no guarantees and no bound on how far results sit from the true optimum.

### Details

Functions 1–7 use a Gaussian Process, which predicts a score for any untried point and says how confident it is. That confidence is what makes sensible exploration possible.

Two things were tuned per function. First, how adventurous to be: pushed harder where the problem description warned of many local optima, kept tight where a single peak was described. Second, whether to reshape the outputs before modelling — four functions had ranges too wide to fit in one go, and Function 1's values span about 200 orders of magnitude, which the model reads as a column of zeros unless compressed.

Function 8 uses a different model altogether. With 45 points spread through eight dimensions the Gaussian Process has too little to work with, so an SVM splits the space into promising and poor halves, and eight neural networks model the promising half. Where the networks disagree, that stands in for uncertainty.

### Performance

Measured by best value found, since no ground truth exists. These figures cover 5 of the 12 rounds the project allowed, so they describe an unfinished search.

Four of eight functions improved — Function 5 by 153%, Function 7 by 61%, Function 6 by 55%, Function 4 by 6%. Functions 1, 2, 3 and 8 did not improve at all. Cross-validation explains why: every function whose model scored below R² 0.2 failed, and every function above 0.6 succeeded, with no exceptions. The stalled functions were not exploring badly — they were following models that could not tell one region from another.

### Assumptions and Limitations

The model assumes each surface varies at the same rate everywhere and has no sudden jumps, neither of which can be checked from the data. Assumed noise levels turned out to be wrong: four of five functions found far less noise in the measurements than was expected. For Function 8, ensemble disagreement was assumed to measure uncertainty and does not — it barely tracks distance from known data, which is why three of five rounds re-queried almost the same point. The largest limitation is budget. The project allowed 12 rounds and 5 were completed, so under half the available evaluations were used. Three of the four successes recorded their best value in the last completed round, and Function 5's gains were still accelerating — +181, +351, +1137 across its final three rounds. These are searches that stopped, not searches that converged, and the four stalled functions may simply not have reached the point where the method could help.

### Ethical Considerations

Every choice — kernel, transform, exploration setting — is recorded in the notebooks with the reasoning behind it, so a reviewer can follow the decisions rather than just the outcome. Negative results are reported in the same detail as positive ones; reporting only the four successes would misrepresent how reliable the approach is. Applied to real decisions, these assumptions would need validating rather than assuming, and a person would need to review each proposed query before it was run.
