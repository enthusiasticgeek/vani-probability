# vani-probability

Probability and statistics library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler).

Provides descriptive statistics, discrete and continuous distributions, correlation, OLS regression, information theory, and hypothesis testing — as pure vāṇī source. Does **not** reimplement anything already available as a vāṇī compiler builtin.

## Add to your project

```toml
# vani.toml
[deps]
probability = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add probability
vanic build
```

## What's included (v0.1 stubs — see TODO.md for implementation status)

| Module | Functions |
|---|---|
| Descriptive | `mean`, `variance`, `std_dev`, `median`, `skewness`, `kurtosis`, `sample_min/max` |
| Discrete distributions | `binomial_pmf/cdf`, `poisson_pmf/cdf`, `geometric_pmf` |
| Continuous distributions | `exponential_pdf/cdf`, `uniform_pdf/cdf`, `t_pdf`, `beta_pdf`, `chi_squared_pdf` |
| Correlation | `pearson_r`, `covariance` |
| Regression | `ols_slope`, `ols_intercept` |
| Information theory | `shannon_entropy`, `kl_divergence`, `cross_entropy` |
| Hypothesis tests | `t_stat_one_sample`, `t_stat_two_sample`, `chi_squared_stat` |

## What this library does NOT provide

These are already vāṇī compiler builtins — use them directly with no import:

| Builtin | What it does |
|---|---|
| `f64_normal_pdf(x, mean, sd)` | Gaussian PDF |
| `f64_normal_cdf(x, mean, sd)` | Gaussian CDF |
| `rand_normal(mean, sd)` | Sample from N(mean, sd²) |
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
