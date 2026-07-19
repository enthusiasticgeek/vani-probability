# vani-probability — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `f64_normal_pdf(x, mean, sd)`  `f64_normal_cdf(x, mean, sd)`
> `rand_normal(mean, sd)`  `rand_uniform(lo, hi)`  `rand_i64(lo, hi)`
> `rand_f64()`  `rand_bool(p)`  `rand_choice(v)`
> `f64_erf`  `f64_erfc`  `f64_tgamma`  `f64_lgamma`
> `i64_factorial`  `i64_binomial`  `i64_perm`
> `f64_geometric_mean`  `f64_harmonic_mean`  `f64_quadratic_mean`  `f64_l1_norm`

---

## Companion repo assessment

### vani-matrix — is it a prerequisite?

**NOT required for v0.2.0 or v0.3.0.** Markov chain transition matrices can be encoded
as flat row-major `Vec<f64>` of length n²: `T[i*n + j]` = P(j | i). All Markov and Bayes
operations work with this encoding via manual loops. Same for a flat covariance matrix.

**Required for v0.4.0** (multiple linear regression, PCA, Kalman filter). Those need:
- `mat_mul_rect(A, B, m, k, n)` — rectangular multiply
- `mat_inv_n(A, n)` — Gauss-Jordan inversion
- `mat_transpose(A, rows, cols)`
- `cholesky(A, n)` — Cholesky L factor for sampling

**Recommendation:** Create `vani-matrix` repo after v0.2.0 ships, before starting v0.4.0.
vani-probability v0.4.0 will add it as a `[deps]` entry in vani.toml.

### vani-special — regularised incomplete functions

Required by v0.3.0 (CDFs for t, chi², beta, gamma, F distributions).
Two options:

| Option | Pros | Cons |
|--------|------|------|
| A: implement inside vani-probability as private helpers | No new repo, simpler | Other packages (e.g. vani-calculus) can't reuse |
| B: new `vani-special` repo, add as `[deps]` | Reusable, clean separation | One more repo to maintain |

**Recommendation:** Option A for v0.3.0 (private helpers `_gamma_reg`, `_beta_reg`
prefixed with `_` to signal internal use). Extract to vani-special if a second consumer
appears.

---

## Compiler dependencies

Track progress in `vani-compiler/docs/TODO_CURRENT.md` under "Vec\<f64\> builtin parity".

| Ticket | Compiler change needed | Unblocks in this package |
|--------|----------------------|--------------------------|
| F64-1 | `sort_by` on `Vec<f64>` via `fn(f64,f64)->i64` comparator | `quantile`, `iqr`, `mode`, `spearman_r`, `mann_whitney_u_stat`; cleaner `median` |
| F64-2 | `vec_median`, `vec_kth_smallest`, `vec_argmin`, `vec_argmax` on `Vec<f64>` | style preference only |
| F64-3 | `vec_fold`, `vec_map`, `vec_filter` on `Vec<f64>` | style preference; all doable with manual loops |

> **Interim workaround for median:** insertion sort on a Vec copy — O(n²) but correct.
> Replace with `sort_by_f64` once F64-1 lands.

---

## v0.1.0 — Implemented ✓

### Descriptive statistics (10 functions)
- [x] `mean`, `variance`, `std_dev`, `median` (insertion-sort workaround), `skewness`, `kurtosis`
- [x] `sample_min`, `sample_max`, `weighted_mean`, `weighted_variance`

### Discrete distributions (8 functions)
- [x] `binomial_pmf`, `binomial_cdf`, `poisson_pmf`, `poisson_cdf`
- [x] `geometric_pmf`, `geometric_cdf`, `negative_binomial_pmf`, `hypergeometric_pmf`

### Continuous distributions (11 functions)
- [x] `exponential_pdf`, `exponential_cdf`, `sample_exponential`
- [x] `uniform_pdf`, `uniform_cdf`
- [x] `t_pdf`, `beta_pdf`, `chi_squared_pdf`, `gamma_pdf`, `laplace_pdf`, `log_normal_pdf`

### Regression & correlation (5 functions)
- [x] `covariance`, `pearson_r`, `ols_slope`, `ols_intercept`, `ols_r_squared`

### Information theory (4 functions)
- [x] `shannon_entropy` [bits], `kl_divergence` [nats], `cross_entropy` [nats], `renyi_entropy` [nats]

### Hypothesis testing (4 functions)
- [x] `t_stat_one_sample`, `t_stat_two_sample` (Welch), `chi_squared_stat`, `z_stat`

### Safety annotations
- [x] `#[bounded_stack(bytes=N)]` on all 42 functions (worst-case call-chain depth)

---

## v0.2.0 — Implemented ✓: Bayes, Markov chains, time series

No new compiler features or companion repos required. All use flat `Vec<f64>` encoding.

### Conditional probability and Bayesian inference

- [x] `conditional_prob(p_ab, p_b)` — P(A|B) = P(A∩B)/P(B); guard P(B)>0
- [x] `law_of_total_prob(priors, likelihoods)` — P(B) = Σᵢ P(B|Aᵢ)P(Aᵢ); dot product of two Vecs
- [x] `bayes_posterior(priors, likelihoods)` — unnormalised posterior Vec: pᵢ × lᵢ for each hypothesis
- [x] `normalize_probs(ps)` — normalise a Vec to sum to 1.0; used after `bayes_posterior`
- [x] `bayes_log_posterior(log_priors, log_likelihoods)` — log-space to avoid underflow on many features
- [x] `naive_bayes_classify(log_priors, log_likelihoods_flat, n_classes, n_features)` — returns argmax class index

### Discrete-time Markov chains

Transition matrix T stored row-major flat: `T[i*n + j]` = P(state j | state i).
Row sums must equal 1.0. Initial state distribution π₀ is a Vec<f64> of length n.

- [x] `markov_step(pi, T, n)` — one step: new_pi[j] = Σᵢ pi[i] × T[i*n+j]; returns new Vec
- [x] `markov_run(pi, T, n, steps)` — apply `markov_step` k times; returns final distribution
- [x] `markov_stationary(T, n, tol, max_iter)` — power iteration from uniform π₀; returns π∞
- [x] `markov_is_absorbing_state(T, n, state)` — returns T[state*n+state] == 1.0
- [x] `markov_tv_distance(pi, qi, n)` — total variation distance: 0.5 × Σ|πᵢ−qᵢ|
- [x] `markov_mixing_steps(T, n, eps, max_iter)` — steps from uniform until TV(π_k, π∞) < eps
- [x] `markov_entropy_rate(T, n, pi)` — H = −Σᵢ πᵢ Σⱼ T[i,j] log T[i,j] [nats]

### Time series and streaming statistics

- [x] `moving_average(xs, window)` — simple MA; returns Vec of length n−window+1
- [x] `ema(xs, alpha)` — exponential moving average; returns Vec same length as xs
- [x] `autocorrelation(xs, lag)` — sample ACF at lag k: corr(xᵢ, xᵢ₊ₖ)
- [x] `acf_vec(xs, max_lag)` — ACF for lags 0..max_lag; returns Vec of length max_lag+1
- [x] `diff_series(xs)` — first difference: yᵢ = xᵢ₊₁ − xᵢ; returns Vec of length n−1
- [x] `cumsum(xs)` — cumulative sum; returns Vec same length as xs

### Online / streaming statistics (stateless helpers for Welford's algorithm)

No Vec storage needed — user maintains scalars across stream events.

- [x] `welford_mean_update(old_mean, count, x)` — returns updated mean
- [x] `welford_m2_update(old_m2, old_mean, new_mean, x)` — returns updated M2 (sum of squared deviations)
- [x] `welford_variance(m2, count)` — returns Bessel-corrected variance from M2 and count

### Extended regression and variance tests

- [x] `f_stat(xs, ys)` — F-statistic for variance ratio: var(xs)/var(ys); requires var(ys)>0
- [x] `pooled_variance(xs, ys)` — pooled sample variance for equal-variance t-test
- [x] `welch_df(xs, ys)` — Satterthwaite degrees of freedom for Welch t-test
- [x] `standardize(xs)` — z-score transform: returns new Vec of (xᵢ−m̄)/s
- [x] `mutual_information_discrete(ps_xy, ps_x, ps_y, nx, ny)` — Σᵢⱼ p(x,y)log(p(x,y)/(p(x)p(y))); flat joint

### Tests and examples for v0.2.0
- [x] `tests/test_bayes.vani` — conditional_prob, law_of_total, posterior, normalize
- [x] `tests/test_markov.vani` — step, stationary, entropy_rate, mixing_steps
- [x] `tests/test_timeseries.vani` — MA, EMA, ACF, diff, cumsum, Welford online
- [x] `tests/test_regression2.vani` — f_stat, pooled_variance, welch_df, standardize
- [x] `examples/markov_weather.vani` — 3-state (Sun/Cloud/Rain) Markov chain demo
- [x] `examples/bayes_medical.vani` — Bayesian diagnosis (prior + test sensitivity/specificity)
- [x] `examples/timeseries_demo.vani` — MA + EMA + ACF on a synthetic signal

---

## v0.3.0 — Implemented ✓: Special functions → CDFs and p-values

Implements regularised incomplete gamma and beta as **private helpers** (prefixed `_`).
No new companion repos required.

### Regularised incomplete gamma P(a, x) = γ(a,x)/Γ(a)

Needed by: `chi_squared_cdf`, `gamma_cdf`, `poisson_cdf_exact`.

- [x] `_gamma_reg_series(a, x)` — series expansion for x ≤ a+1: P(a,x) = e^{−x} x^a / Γ(a) × Σ xⁿ/∏(a+k)
- [x] `_gamma_reg_cf(a, x)` — continued-fraction via Lentz for x > a+1: Q(a,x) = 1−P(a,x)
- [x] `_gamma_reg(a, x)` — router: uses series for small x, CF for large x; guard x≤0→0, a≤0→0
- [x] `chi_squared_cdf(k, x)` — P(k/2, x/2) using `_gamma_reg`
- [x] `gamma_cdf(shape, rate, x)` — P(shape, rate×x)
- [x] `exponential_cdf_exact(lambda, x)` — already closed-form; mark as complete via existing formula

### Regularised incomplete beta I_x(a,b)

Needed by: `t_cdf`, `beta_cdf`, `f_cdf`, `binomial_cdf_exact`.

- [x] `_beta_cf(a, b, x)` — Lentz's modified continued fraction; ~40 iterations to convergence
- [x] `_beta_reg(a, b, x)` — uses CF with symmetry relation I_x(a,b) = 1 − I_{1-x}(b,a) when x > (a+1)/(a+b+2)
- [x] `t_cdf(df, t)` — 1 − I_{df/(df+t²)}(df/2, 1/2) / 2; two-tailed via sign(t)
- [x] `beta_cdf(alpha, beta, x)` — I_x(alpha, beta) via `_beta_reg`
- [x] `f_cdf(df1, df2, x)` — I_{df1×x/(df1×x+df2)}(df1/2, df2/2) via `_beta_reg`

### p-values using existing test statistics + new CDFs

- [x] `t_pvalue_two_tailed(t, df)` — 2 × (1 − t_cdf(df, |t|)); uses `t_cdf`
- [x] `chi_squared_pvalue(stat, df)` — 1 − chi_squared_cdf(df, stat)
- [x] `z_pvalue_two_tailed(z)` — 2 × (1 − f64_normal_cdf(|z|, 0, 1)); uses compiler builtin
- [x] `f_pvalue(f, df1, df2)` — 1 − f_cdf(df1, df2, f)

### Tests and examples for v0.3.0
- [x] `tests/test_special.vani` — gamma_reg series + CF crosscheck, beta_reg symmetry, convergence
- [x] `tests/test_cdfs.vani` — chi_squared_cdf spot values, t_cdf vs tables, f_cdf
- [x] `tests/test_pvalues.vani` — z_pvalue, t_pvalue, chi_squared_pvalue known cases
- [x] `examples/hypothesis_full.vani` — end-to-end: raw data → stat → df → p-value → decision

---

## v0.4.0 — Planned: Matrix-dependent features

**Prerequisite: vani-matrix repo must exist and be added to `[deps]` in vani.toml.**

### Multiple linear regression (needs mat_inv_n)

- [ ] `mlr_fit(X_flat, y, n_obs, n_pred)` — β = (XᵀX)⁻¹ Xᵀy; returns coefficient Vec
- [ ] `mlr_predict(X_flat, beta, n_obs, n_pred)` — ŷ = X β
- [ ] `mlr_residuals(y, y_hat)` — r = y − ŷ
- [ ] `mlr_r_squared(y, y_hat)` — 1 − SS_res/SS_tot
- [ ] `mlr_adj_r_squared(y, y_hat, n_pred)` — adjusted R²

### Covariance matrix
- [ ] `cov_matrix(X_flat, n_obs, n_vars)` — returns flat n_vars × n_vars matrix; two-pass
- [ ] `correlation_matrix(X_flat, n_obs, n_vars)` — standardise cov_matrix rows+cols by std_devs

### Dimensionality reduction
- [ ] `pca_power_iter(cov_flat, n, max_iter, tol)` — first principal component via power iteration
- [ ] `pca_deflate(cov_flat, n, pc1)` — deflate matrix for second component extraction

### Stochastic processes
- [ ] `random_walk_1d(n_steps, sigma)` — returns Vec of n_steps positions; uses rand_normal
- [ ] `geometric_brownian(s0, mu, sigma, dt, n_steps)` — GBM path via Euler-Maruyama
- [ ] `kalman_predict(x, P, F, Q, n)` — Kalman filter predict step (flat matrices)
- [ ] `kalman_update(x, P, z, H, R, n, m)` — Kalman filter update step

### Tests and examples for v0.4.0
- [ ] `tests/test_mlr.vani` — fit y=2x1+3x2+1, verify coefficients, R²=1
- [ ] `tests/test_cov_matrix.vani` — identity input → identity cov; 2D correlated data
- [ ] `tests/test_stochastic.vani` — random_walk mean≈0, GBM positive paths
- [ ] `examples/portfolio_demo.vani` — GBM multi-asset simulation with correlations
- [ ] `examples/kalman_demo.vani` — 1D position tracking under noisy measurement

---

## Deferred (sort required — blocked on F64-1)

- [ ] `quantile(xs, q)` — linear interpolation between sorted neighbors
- [ ] `iqr(xs)` — Q3 − Q1
- [ ] `mode(xs)` — most frequent value (sort + scan)
- [ ] `spearman_r(xs, ys)` — rank-transform then pearson_r; both sort passes needed
- [ ] `mann_whitney_u_stat(xs, ys)` — nonparametric; needs sorted merge
- [ ] `kolmogorov_smirnov_stat(xs, ys)` — nonparametric; needs sorted empirical CDFs

---

## Safety / WCET annotations

- [x] `#[bounded_stack(bytes=N)]` on all v0.1.0 functions (worst-case call-chain depth)
- [x] `#[bounded_stack(bytes=N)]` on all v0.2.0 functions (budgets verified against `vanic check`'s
      exact worst-case-stack-depth diagnostics, not estimated by hand)
- [x] `#[bounded_stack(bytes=N)]` on all v0.3.0 functions (same verification method; the Lentz
      continued-fraction helpers needed noticeably larger budgets than the hand estimate)
- [ ] `#[wcet(cycles=N)]` on leaf functions with no loops — add after cycle audit
- [ ] v0.4.0+ functions: annotate after implementation and `vanic stack-depth` run
