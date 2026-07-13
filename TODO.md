# vani-probability — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `f64_normal_pdf(x, mean, sd)`  `f64_normal_cdf(x, mean, sd)`
> `rand_normal(mean, sd)`  `rand_uniform(lo, hi)`  `rand_i64(lo, hi)`
> `rand_f64()`  `rand_bool(p)`  `rand_choice(v)`
> `f64_erf`  `f64_erfc`  `f64_tgamma`  `f64_lgamma`
> `i64_factorial`  `i64_binomial`  `i64_perm`
> `f64_geometric_mean`  `f64_harmonic_mean`  `f64_quadratic_mean`  `f64_l1_norm`

---

## Descriptive statistics

- [ ] `mean` — accumulate sum / n (use Kahan summation for precision)
- [ ] `variance` — Bessel-corrected; use Welford online algorithm to avoid two-pass
- [ ] `std_dev` — calls `variance` + builtin `sqrt`
- [ ] `median` — in-place sort copy (or partial sort) then index
- [ ] `skewness` — third standardised moment; Fisher's definition
- [ ] `kurtosis` — excess kurtosis (Fisher: subtract 3)
- [ ] `sample_min` / `sample_max` — scan loop
- [ ] `quantile(xs, q)` — linear interpolation between sorted neighbors
- [ ] `iqr(xs)` — inter-quartile range: Q3 - Q1
- [ ] `mode(xs)` — most frequent value (requires sort or hashmap)

## Discrete distributions
*(i64_binomial coefficient is builtin; PMF/CDF are new)*

- [ ] `binomial_pmf` — `i64_binomial(n,k)` × `pow(p,k)` × `pow(1-p,n-k)`
- [ ] `binomial_cdf` — cumulative sum of PMF up to k
- [ ] `poisson_pmf` — `pow(λ,k)` × `exp(-λ)` / `i64_factorial(k)`
- [ ] `poisson_cdf` — cumulative sum
- [ ] `geometric_pmf` / `geometric_cdf`
- [ ] `negative_binomial_pmf`
- [ ] `hypergeometric_pmf` — sampling without replacement
- [ ] Sampling functions: `sample_binomial`, `sample_poisson`, `sample_geometric`
      (build on `rand_bool` and `rand_i64` builtins)

## Continuous distributions
*(Normal is already a builtin — wrap f64_normal_pdf/cdf; add the rest)*

- [ ] `exponential_pdf` / `exponential_cdf` / `sample_exponential`
      (`sample` via inverse CDF: `-log(rand_f64()) / lambda`)
- [ ] `uniform_pdf` / `uniform_cdf` (wraps `rand_uniform` for sampling)
- [ ] `t_pdf` — Student's t; uses `f64_tgamma` builtin
- [ ] `t_cdf` — regularised incomplete beta function (needs `beta_incomplete`)
- [ ] `beta_pdf` / `beta_cdf` — uses `f64_tgamma` builtin
- [ ] `chi_squared_pdf` / `chi_squared_cdf`
- [ ] `gamma_pdf(shape, rate)` — uses `f64_tgamma` builtin
- [ ] `cauchy_pdf` / `cauchy_cdf`
- [ ] `laplace_pdf` / `laplace_cdf`
- [ ] `log_normal_pdf` — uses `f64_normal_pdf` builtin + builtin `log`
- [ ] Inverse CDF (percent-point) functions for each distribution

## Correlation and regression

- [ ] `covariance` — two-pass or online algorithm
- [ ] `pearson_r` — `covariance / (std_dev_x * std_dev_y)`
- [ ] `spearman_r` — rank-transform xs and ys, then `pearson_r` on ranks
- [ ] `ols_slope` / `ols_intercept` — simple linear regression
- [ ] `ols_r_squared` — coefficient of determination
- [ ] Multiple linear regression — normal equations (needs Vec<Vec<f64>> or matrix)
- [ ] `weighted_mean(xs, ws)` / `weighted_variance`

## Information theory

- [ ] `shannon_entropy` — uses builtin `log2`
- [ ] `kl_divergence` — uses builtin `log`
- [ ] `cross_entropy` — uses builtin `log`
- [ ] `mutual_information(ps_xy, ps_x, ps_y)`
- [ ] `renyi_entropy(ps, alpha)` — generalised entropy

## Hypothesis testing

- [ ] `t_stat_one_sample` — standardised departure from mu0
- [ ] `t_stat_two_sample` — Welch's t-statistic
- [ ] `chi_squared_stat` — goodness-of-fit statistic
- [ ] `z_stat` — z-score for known-variance test
- [ ] p-value approximation using `f64_normal_cdf` builtin for z-tests
- [ ] p-value from t-distribution CDF for t-tests (needs `t_cdf`)

## Safety / WCET annotations (future)

- [ ] Add `#[wcet(cycles=N)]` and `#[bounded_stack(bytes=N)]` to all public functions
      once implementations are complete.
- [ ] All loops over Vec<f64> inputs must have bounded length annotations or the
      WCET checker will report UNBOUNDED — document the expected budget formula.
