# Gaussian Mixture Model & Iterative Selection Bias for Redshift Distributions

A Python-based framework designed for cosmic shear analysis to fit multi-component Gaussian Mixture Models (GMM) to individual tomographic redshift data bins, perform statistical model selection via the Bayesian Information Criterion (BIC), and iteratively reconstruct an overarching power-law selection bias function from wide-field surveys.

The code is modularly structured around data management, optimization algorithms, and specialized physical models:

* `main.py`: The application entry point. Configures data pathways, weights, priors, and orchestrates the global iterative loops.
* `fit.py`: Contains the `Fit` driver class which coordinates individual narrow-bin optimizations, BIC calculations, and global bias loops.
* `model.py`: Defines the foundational `Model` object which handles superpositions of components, normalization via numerical integration, and parameter mapping.
* `gauss.py`: Atomic component class representing an individual Gaussian distribution $G(A, m, s)$ with embedded parameter setters.
* `bias.py`: Implements the power-law selection bias function $A \cdot (1+z)^a$ along with Monte Carlo error propagation for the model's derived mean redshift.
* `mcmc.py`: Wraps the `emcee` ensemble sampler to compute the log-likelihood and execute parallelized burn-in/sampling phases utilizing covariance matrices.
* `best_model.py`: Automates the model parsing loop to determine the optimal component counts by auditing relative $\Delta\text{BIC}$ values.
* `folder.py`: Internal I/O utility to structurally build run directories and prevent file-handling conflicts.
