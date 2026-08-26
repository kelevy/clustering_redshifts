# Tomographic Redshift Distribution Modeling

Gaussian Mixture Model fitting of galaxy redshift distributions, with MCMC parameter estimation, Bayesian-Information-Criterion (BIC) model selection, and iterative galaxy-bias calibration.

This pipeline was built to characterise the redshift distributions **n(z)** of galaxies in tomographic weak-lensing bins, a key ingredient for cosmic shear analyses, where the shape and mean of n(z) directly propagate into cosmological parameter constraints.

---

## Overview

Given a set of narrow tomographic redshift bins (each described by a measured n(z) curve and its covariance matrix), the pipeline:

1. **Fits** each narrow bin with a superposition of Gaussian components using MCMC (`emcee`), for an increasing number of components.
2. **Selects** the best-fitting number of components per bin via the BIC, stopping once adding another Gaussian no longer meaningfully improves the fit (ΔBIC ≤ 2).
3. **Combines** the best-fit models from all narrow bins, weighted by galaxy number density, and compares the sum against an independently measured broad-bin n(z).
4. **Fits a galaxy-bias power law**, b(z) = A·(1+z)^a, that reconciles the weighted narrow-bin sum with the broad bin.
5. **Iterates** steps 1–4, feeding the recovered bias exponent back into the narrow-bin fits, until the bias parameter converges.

The output is, per tomographic bin, a smooth analytic (Gaussian-mixture) model of n(z) with quantified uncertainty, plus a global bias model relating narrow- and broad-bin redshift distributions.

---

## Scientific Background

Weak-lensing and CMB-lensing analyses split the source galaxy sample into tomographic bins to extract redshift information from the lensing signal. Because the true n(z) of each bin is only known from noisy, binned measurements, it is common to fit each bin with a flexible, low-dimensional analytic model. Here, a mixture of Gaussians rather than using the raw histogram directly. This:

- smooths out per data point noise while preserving the overall shape of the distribution,
- provides a continuous function that can be integrated (e.g. to compute the mean redshift and its uncertainty),
- allows a principled choice of model complexity (number of Gaussians) via the BIC, rather than an arbitrary fixed number,
- and enables a self-consistent estimate of the redshift-dependent galaxy bias, obtained by requiring that the weighted sum of narrow-bin models reproduces the independently measured broad-bin distribution.

---

## Repository Structure

```
redshift-gmm/
├── main.py              # Entry point: configures bins, priors, MCMC settings, and runs the fit
├── src/
│   ├── fit.py            # Fit class: ties together run / get_best_model / get_bias / iterate
│   ├── data.py            # Data class: loads n(z) + covariance matrix, handles NaNs, plotting
│   ├── gauss.py           # Gauss class: single Gaussian component (amplitude, mean, std)
│   ├── model.py            # Model class: sum of Gauss components, normalization, mean, plotting
│   ├── bias.py              # Bias class: weighted superposition of bin models × power-law bias
│   ├── mcmc.py              # MCMC class: emcee wrapper, log-likelihood, priors, BIC computation
│   ├── best_model.py       # Best_Model class: BIC-based model selection across Gaussian counts
│   └── folder.py            # Folder class: creates/manages the output directory structure
│
├── data/                # Input n(z) measurements (.ascii) and covariance matrices (.covmat)
├── results/             # Per-bin fit outputs: BIC tables, best-fit models (.bin), MCMC diagnostics
├── figures/             # Saved n(z) plots per bin, per iteration, and the final bias plot
├── requirements.txt
└── README.md
```

`fit.py`'s `Fit` class ties everything together and exposes three methods:

- `run()` — fits 1 to `max_nbre_gauss_fits` Gaussians to every narrow bin
- `get_best_model()` — selects the best number of components per bin via BIC
- `get_bias()` — fits the bias power law against the broad bin
- `iterate()` — repeats the above for a given number of iteration steps

---

## Installation

```bash
git clone https://github.com/kelevy/redshift-gmm.git
cd redshift-gmm
pip install -r requirements.txt
```

**Dependencies:** `numpy`, `scipy`, `matplotlib`, `emcee`, `corner`

---

## Usage

1. Place your narrow-bin and broad-bin n(z) files (`.ascii`, three columns: `z`, `n(z)`, `n(z)_err`) and their covariance matrices (`.covmat`) in `data/`.
2. Configure `main.py` with your bin file paths, per-bin galaxy weights, MCMC settings (walkers, burn-in, steps), and priors on the Gaussian and bias parameters.
3. Run from the project root:

```bash
python main.py
```

Results: per-bin BIC tables, best-fit models, and MCMC diagnostics are written to `results/`, organised by bin and iteration step; n(z) plots are saved to `figures/`.

### Example configuration (from `main.py`)

```python
fit = Fit(
    bins, broad_bin, data_color="red", weights=weights,
    figure_title=figure_title, max_nbre_gauss_fits=6, model_color="blue",
    walkers=150, walker_offset=1e-4, burn_in_steps=500, main_steps=10000,
    priors_gauss=priors_gauss, priors_bias=priors_bias,
    name_save_folder="results", bins_cov=bins_cov, broad_bin_cov=broad_bin_cov,
    normalize=True, mean=True
)
fit.iterate(10)
```