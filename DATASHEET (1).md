# Datasheet: Black-Box Optimization Capstone Data Set

### Motivation

This data set records the search for the maxima of eight unknown black-box functions. It serves two purposes: it trains the models that choose each week's query, and it documents what was tried and what came back.

### Composition

It contains 215 input-output pairs across eight functions, with search spaces from 2D to 8D. Of these, 175 points were supplied at the start and 40 were added through weekly queries. Each row holds the input coordinates, the score returned, and a column marking which round the point came from. There are no missing values, but the true optimum of each function is unknown and no point was ever queried twice, so observation noise cannot be measured.

The largest gap is simply size. The project allowed 12 rounds of submission and 5 were completed, so the data set holds less than half the evidence the project could have produced. Every function is substantially under-sampled as a result — most severely Function 8, with 45 points in an eight-dimensional space.

### Collection Process

The starting points were supplied by the course. Query points were chosen by the author and submitted through the course platform, which returned the score a week later. The first round was chosen by eye from scatter plots; later rounds used a model to pick the most promising untried point. Five of the twelve available rounds were completed. The sampling is deliberately non-random, so the data is concentrated in regions the models already favoured.

### Preprocessing and Uses

The stored CSV files are unmodified — all transformations happen inside the notebooks. Four functions needed log transforms to compress very wide output ranges; the other four were used as they are. The data suits fitting surrogate models and reproducing the optimization trajectory. It is not suitable for training a general-purpose regressor or for drawing conclusions about the real drug, warehouse or recipe problems the function descriptions reference, which are labels on simulated functions.

### Distribution and Maintenance

The data set is public in the project GitHub repository as plain CSV files, released for academic use and maintained by the author. The black-box functions themselves belong to the course provider and are not included. Collection stopped after five of the twelve available rounds, so the data set is incomplete relative to what the project allowed.
