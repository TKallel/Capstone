# References

Literature underpinning the methods used in this project. Each entry notes
where it is relevant, so the list reflects what actually informed the work
rather than a general reading list.

---

## Bayesian optimization — foundations

**Rasmussen, C. E. and Williams, C. K. I. (2006).** *Gaussian Processes for
Machine Learning.* MIT Press.
The standard reference for Gaussian Process regression. Chapter 4 covers the
Matérn family of kernels used throughout Functions 1–7, and explains why the
smoothness parameter ν matters. Freely available at
<http://gaussianprocess.org/gpml/>.

**Jones, D. R., Schonlau, M. and Welch, W. J. (1998).** Efficient global
optimization of expensive black-box functions. *Journal of Global
Optimization*, 13(4), 455–492.
The paper that established surrogate-based optimization for expensive
functions, and the origin of the Expected Improvement criterion in its modern
form. Directly relevant to the acquisition function used here.

**Mockus, J. (1975).** On Bayesian methods for seeking the extremum. In
*Optimization Techniques IFIP Technical Conference*, Springer, 400–404.
The earlier formulation of Expected Improvement that Jones et al. built on.

**Kushner, H. J. (1964).** A new method of locating the maximum point of an
arbitrary multipeak curve in the presence of noise. *Journal of Basic
Engineering*, 86(1), 97–106.
The earliest statement of the problem this project addresses: finding a
maximum from few, noisy evaluations.

**Frazier, P. I. (2018).** A tutorial on Bayesian optimization.
arXiv:1807.02811.
The most accessible overview of the field, and a good starting point for
anyone reading this repository without prior background.

**Shahriari, B., Swersky, K., Wang, Z., Adams, R. P. and de Freitas, N.
(2016).** Taking the human out of the loop: a review of Bayesian optimization.
*Proceedings of the IEEE*, 104(1), 148–175.
A broad survey comparing acquisition functions and surrogate choices. Useful
for the decision between UCB and EI discussed in the model card.

**Garnett, R. (2023).** *Bayesian Optimization.* Cambridge University Press.
A recent full-length treatment covering theory and practice.

---

## Acquisition functions

**Srinivas, N., Krause, A., Kakade, S. M. and Seeger, M. (2010).** Gaussian
process optimization in the bandit setting: no regret and experimental design.
*Proceedings of the 27th International Conference on Machine Learning*,
1015–1022.
Introduces GP-UCB with theoretical guarantees, and analyses how the
exploration weight β should be set. Relevant to the UCB used in the early
rounds and to Function 8.

**Bull, A. D. (2011).** Convergence rates of efficient global optimization
algorithms. *Journal of Machine Learning Research*, 12, 2879–2904.
Analyses when Expected Improvement converges and how quickly. Relevant to the
limitation noted in the model card that no bound is available on how far the
results sit from the true optimum.

---

## High-dimensional problems

**Binois, M. and Wycoff, N. (2022).** A survey on high-dimensional Gaussian
process modeling with application to Bayesian optimization. *ACM Transactions
on Evolutionary Learning and Optimization*, 2(2), 1–26.
<https://doi.org/10.1145/3545611>
Directly relevant to Function 8. Reviews why standard GP regression degrades
as dimensionality rises, and surveys the structural assumptions used to
compensate.

**Wang, Z., Hutter, F., Zoghi, M., Matheson, D. and de Freitas, N. (2016).**
Bayesian optimization in a billion dimensions via random embeddings. *Journal
of Artificial Intelligence Research*, 55, 361–387.
One established alternative to abandoning the GP in high dimensions:
projecting into a low-dimensional subspace. An approach this project did not
take, and a reasonable direction for future work on Function 8.

**Eriksson, D., Pearce, M., Gardner, J., Turner, R. D. and Poloczek, M.
(2019).** Scalable global optimization via local Bayesian optimization.
*Advances in Neural Information Processing Systems*, 32.
The TuRBO method, which fits GPs within local trust regions rather than
globally. A more principled solution to the problem that motivated the
ensemble approach used for Function 8.

---

## Neural network ensembles and uncertainty

**Lakshminarayanan, B., Pritzel, A. and Blundell, C. (2017).** Simple and
scalable predictive uncertainty estimation using deep ensembles. *Advances in
Neural Information Processing Systems*, 30.
The basis for the Function 8 approach: training several networks and using
their disagreement as an uncertainty estimate.

**Ovadia, Y., Fertig, E., Ren, J., Nado, Z., Sculley, D., Nowozin, S., Dillon,
J. V., Lakshminarayanan, B. and Snoek, J. (2019).** Can you trust your model's
uncertainty? Evaluating predictive uncertainty under dataset shift. *Advances
in Neural Information Processing Systems*, 32.
Directly relevant to the weakness identified in Function 8. Benchmarks how
uncertainty estimates behave when test inputs move away from the training
distribution — the exact situation in which this project found the ensemble's
disagreement to be uninformative.

**Gal, Y. and Ghahramani, Z. (2016).** Dropout as a Bayesian approximation:
representing model uncertainty in deep learning. *Proceedings of the 33rd
International Conference on Machine Learning*, PMLR 48, 1050–1059.
Shows that dropout at prediction time approximates Bayesian inference in a
deep network, giving a principled route to uncertainty estimates from neural
networks. Relevant as an alternative to the ensemble approach used for
Function 8: where ensemble disagreement proved uninformative here, MC dropout
provides a theoretically grounded substitute and would be a sensible next
thing to try.

**Cortes, C. and Vapnik, V. (1995).** Support-vector networks. *Machine
Learning*, 20(3), 273–297.
The SVM classifier used as the first stage of the Function 8 model.

---

## Sampling and experimental design

**Sobol, I. M. (1967).** On the distribution of points in a cube and the
approximate evaluation of integrals. *USSR Computational Mathematics and
Mathematical Physics*, 7(4), 86–112.
The sequence used to generate candidate points, replacing the uniform grid
that had been causing the acquisition function to favour corners.

**Owen, A. B. (1998).** Scrambling Sobol' and Niederreiter–Xing points.
*Journal of Complexity*, 14(4), 466–489.
Describes the scrambling used in the implementation, which avoids structural
artefacts in the raw sequence.

**Stein, M. L. (1999).** *Interpolation of Spatial Data: Some Theory for
Kriging.* Springer.
The source of the widely followed recommendation to prefer Matérn kernels over
the squared-exponential, on the grounds that the latter assumes unrealistic
smoothness. This motivated the Matérn ν = 5/2 choice here.

---

## Interpretability and transparency

**Jo, N., Aghaei, S., Benson, J., Gómez, A. and Vayanos, P. (2023).** Learning
optimal fair decision trees: trade-offs between interpretability, fairness,
and accuracy. *Proceedings of the 2023 AAAI/ACM Conference on AI, Ethics, and
Society (AIES)*, 181–192. Also available as arXiv:2201.09932 under the title
*Learning Optimal Fair Classification Trees*.
Informed the treatment of interpretability in this project. The paper proposes
*decision complexity* as a measure allowing interpretability to be compared
across different model classes, and quantifies a "price of interpretability" —
about 4.2 percentage points of out-of-sample accuracy against the best
complex models in their benchmarks. The framing is directly relevant here,
where a Gaussian Process offers an inspectable kernel and calibrated
uncertainty while the neural network ensemble used for Function 8 gives
neither. That substitution bought the ability to model an 8-dimensional
surface at the cost of interpretability, and the uncertainty test in that
notebook suggests the trade was not worthwhile.

**Gebru, T., Morgenstern, J., Vecchione, B., Vaughan, J. W., Wallach, H.,
Daumé III, H. and Crawford, K. (2021).** Datasheets for datasets.
*Communications of the ACM*, 64(12), 86–92.
The framework followed in `DATASHEET.md`.

**Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson,
B., Spitzer, E., Raji, I. D. and Gebru, T. (2019).** Model cards for model
reporting. *Proceedings of the Conference on Fairness, Accountability, and
Transparency*, 220–229.
The framework followed in `MODEL_CARD.md`.

---

## Applications

**Snoek, J., Larochelle, H. and Adams, R. P. (2012).** Practical Bayesian
optimization of machine learning algorithms. *Advances in Neural Information
Processing Systems*, 25.
The paper that popularised Bayesian optimization for hyperparameter tuning —
the scenario Function 7 simulates.

---

## Software

**Pedregosa, F. et al. (2011).** Scikit-learn: machine learning in Python.
*Journal of Machine Learning Research*, 12, 2825–2830.
Provides `GaussianProcessRegressor`, `MLPRegressor` and `SVC`.

**Harris, C. R. et al. (2020).** Array programming with NumPy. *Nature*, 585,
357–362.

**Virtanen, P. et al. (2020).** SciPy 1.0: fundamental algorithms for
scientific computing in Python. *Nature Methods*, 17, 261–272.
Provides the Sobol sampler and the normal distribution functions used in
Expected Improvement.

**Hunter, J. D. (2007).** Matplotlib: a 2D graphics environment. *Computing in
Science & Engineering*, 9(3), 90–95.

---

## A note on use

These references support and explain the methods applied here; they were not
all consulted in equal depth during the project itself. The entries most
directly relevant to decisions made are Rasmussen & Williams (kernel choice),
Jones et al. (Expected Improvement), Shahriari et al. (the choice between
acquisition functions), Binois & Wycoff (why the GP was abandoned at eight
dimensions), Ovadia et al. and Gal & Ghahramani (why the ensemble's
uncertainty estimate proved unreliable, and what to try instead), Jo et al.
(how to think about the interpretability given up in that substitution), and
Gebru et al. and Mitchell et al. (the documentation frameworks).
