# vani-probability

Probability and statistics library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler).

Provides descriptive statistics, discrete and continuous distributions, correlation, OLS regression, information theory, hypothesis testing, Bayesian inference, Markov chains, time series analysis, CDF/p-value hypothesis testing, multiple linear regression, PCA, and stochastic processes (depends on vani-matrix from v0.4 onward) — as pure vāṇī source. Does **not** reimplement anything already available as a vāṇī compiler builtin.

## Add to your project

```toml
# vani.toml
[deps]
probability = { registry = "kosh", version = "^0.4.2" }
```

```sh
vanic add probability
vanic build
```

From v0.4 onward this package depends on [`vani-matrix`](https://github.com/enthusiasticgeek/vani-matrix) (dense linear algebra) for the multiple-linear-regression, PCA, and Kalman-filter functions; `vanic add` resolves it transitively.

## What's included (v0.4.2 — see TODO.md for implementation status)

| Module | Functions |
|---|---|
| Descriptive | `mean`, `variance`, `std_dev`, `median`, `skewness`, `kurtosis`, `sample_min/max`, `weighted_mean/variance`, `quantile`, `iqr`, `mode` |
| Discrete distributions | `binomial_pmf/cdf`, `poisson_pmf/cdf`, `geometric_pmf/cdf`, `negative_binomial_pmf`, `hypergeometric_pmf` |
| Continuous distributions | `exponential_pdf/cdf`, `uniform_pdf/cdf`, `t_pdf`, `beta_pdf`, `chi_squared_pdf`, `gamma_pdf`, `laplace_pdf`, `log_normal_pdf` |
| Correlation & regression | `covariance`, `pearson_r`, `ols_slope/intercept/r_squared` |
| Information theory | `shannon_entropy`, `kl_divergence`, `cross_entropy`, `renyi_entropy` |
| Hypothesis tests | `t_stat_one_sample/two_sample`, `chi_squared_stat`, `z_stat` |
| Nonparametric tests | `spearman_r`, `mann_whitney_u_stat`, `kolmogorov_smirnov_stat` |
| Bayesian inference | `conditional_prob`, `law_of_total_prob`, `bayes_posterior`, `normalize_probs`, `bayes_log_posterior`, `naive_bayes_classify` |
| Markov chains | `markov_step/run/stationary`, `markov_is_absorbing_state`, `markov_tv_distance`, `markov_mixing_steps`, `markov_entropy_rate` |
| Time series | `moving_average`, `ema`, `autocorrelation`, `acf_vec`, `diff_series`, `cumsum` |
| Streaming stats | `welford_mean_update`, `welford_m2_update`, `welford_variance` |
| Extended regression | `f_stat`, `pooled_variance`, `welch_df`, `standardize`, `mutual_information_discrete` |
| Special functions & CDFs | `chi_squared_cdf`, `gamma_cdf`, `exponential_cdf_exact`, `t_cdf`, `beta_cdf`, `f_cdf` |
| p-values | `t_pvalue_two_tailed`, `chi_squared_pvalue`, `z_pvalue_two_tailed`, `f_pvalue` |
| Multiple linear regression | `mlr_fit`, `mlr_predict`, `mlr_residuals`, `mlr_r_squared`, `mlr_adj_r_squared` |
| Covariance & correlation matrices | `cov_matrix`, `correlation_matrix` |
| Dimensionality reduction | `pca_power_iter`, `pca_deflate` |
| Stochastic processes | `random_walk_1d`, `geometric_brownian`, `kalman_predict`, `kalman_update` |

## What this library does NOT provide

These are already vāṇī compiler builtins — use them directly with no import:

| Builtin | What it does |
|---|---|
| `f64_normal_pdf(x, mean, sd)` | Gaussian PDF |
| `f64_normal_cdf(x, mean, sd)` | Gaussian CDF |
| `rand_normal()` | Sample from N(0,1) -- scale/shift yourself for N(mean, sd²) |
| `rand_uniform(lo, hi)` | Uniform sample on [lo, hi] |
| `rand_f64()` | Uniform sample on [0, 1) |
| `rand_bool(p)` | Bernoulli sample |
| `rand_i64(lo, hi)` | Uniform integer sample |
| `f64_erf` / `f64_erfc` | Error function |
| `f64_tgamma` / `f64_lgamma` | Gamma / log-gamma |
| `i64_factorial` / `i64_binomial` / `i64_perm` | Combinatorics |
| `f64_geometric_mean` / `f64_harmonic_mean` / `f64_quadratic_mean` | Pythagorean means |

## License

MIT
