# vani-probability — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `f64_normal_pdf(x, mean, sd)`  `f64_normal_cdf(x, mean, sd)`
> `rand_normal(mean, sd)`  `rand_uniform(lo, hi)`  `rand_i64(lo, hi)`
> `rand_f64()`  `rand_bool(p)`  `rand_choice(v)`
> `f64_erf`  `f64_erfc`  `f64_tgamma`  `f64_lgamma`
> `i64_factorial`  `i64_binomial`  `i64_perm`
> `f64_geometric_mean`  `f64_harmonic_mean`  `f64_quadratic_mean`  `f64_l1_norm`

---

## Compiler dependencies (items blocked on vani-compiler changes)

Track progress in `vani-compiler/docs/TODO_CURRENT.md` under "Vec\<f64\> builtin parity".

| Ticket | Compiler change needed | Unblocks in this package |
|--------|----------------------|--------------------------|
| F64-1 | `sort_by` on `Vec<f64>` via `fn(f64,f64)->i64` comparator | `median`, `quantile`, `iqr`, `mode` |
| F64-2 | `vec_median`, `vec_kth_smallest`, `vec_argmin`, `vec_argmax`, `vec_min`, `vec_max` on `Vec<f64>` | cleaner implementations of the above |
| F64-3 | `vec_fold`, `vec_map`, `vec_filter` on `Vec<f64>` | style preference; all are doable with manual loops now |

> **Key finding:** `for x in ref xs` over `Vec<f64>` **already works** (confirmed against
> `checker.rs:11661`). `xs[i]` indexing, `set(mut ref xs, i, v)`, `while`, and scalar f64
> arithmetic all work. All descriptive stats and distributions can be written **right now**
> with manual for-loops. Only `median` / `quantile` / `mode` are blocked on F64-1 (sort).
>
> **Interim workaround for median:** implement insertion sort via `set(mut ref xs, j, v)` +
> manual swap — O(n²) but correct. Annotate as `// blocked: replace with sort_by_f64 once F64-1 lands`.

---

## Can implement NOW (no compiler changes needed)

### Descriptive statistics

- [ ] `mean` — `for x in ref xs` accumulate sum, divide by `len(xs) as f64`
- [ ] `variance` — Welford online algorithm: single-pass, numerically stable; Bessel-corrected
- [ ] `std_dev` — calls `variance` + builtin `sqrt` (already stubbed correctly)
- [ ] `skewness` — third standardised moment; compute mean + std_dev first, then single pass
- [ ] `kurtosis` — excess kurtosis (fourth moment − 3); same pattern
- [ ] `sample_min` — scan loop (compiler's `vec_min` is i64-only, so manual loop required)
- [ ] `sample_max` — scan loop (same reason)
- [ ] `weighted_mean(xs, ws)` — `sum(xs[i]*ws[i]) / sum(ws[i])` via manual loop
- [ ] `weighted_variance(xs, ws)` — weighted Bessel-corrected variance

### Blocked on F64-1 (sort on Vec<f64>)

- [ ] `median` — sort copy, take middle; interim: manual insertion sort
- [ ] `quantile(xs, q)` — linear interpolation between sorted neighbors
- [ ] `iqr(xs)` — Q3 − Q1
- [ ] `mode(xs)` — most frequent value (requires sort + scan, or hashmap)

### Discrete distributions
*(i64_binomial coefficient is builtin; PMF/CDF are new)*

- [ ] `binomial_pmf` — `i64_binomial(n,k)` as f64 × `pow(p,k)` × `pow(1-p,n-k)`
- [ ] `binomial_cdf` — cumulative sum of PMF up to k (while loop)
- [ ] `poisson_pmf` — `pow(λ,k)` × `exp(-λ)` / `i64_factorial(k)` as f64
- [ ] `poisson_cdf` — cumulative sum via while loop
- [ ] `geometric_pmf` — `pow(1-p, k-1) * p`
- [ ] `geometric_cdf` — `1 - pow(1-p, k)`
- [ ] `negative_binomial_pmf` — `i64_binomial(k+r-1, k)` × `pow(p,r)` × `pow(1-p,k)`
- [ ] `hypergeometric_pmf` — three i64_binomial calls

### Continuous distributions

> `f64_normal_pdf` and `f64_normal_cdf` are builtins — do not reimplement.

- [ ] `exponential_pdf` — guard x ≥ 0; `λ × exp(-λx)`
- [ ] `exponential_cdf` — `1 - exp(-λx)`
- [ ] `sample_exponential` — inverse CDF: `-log(rand_f64()) / λ`
- [ ] `uniform_pdf` — guard a < b and a ≤ x ≤ b; `1 / (b-a)`
- [ ] `uniform_cdf` — `(x-a) / (b-a)` clamped
- [ ] `t_pdf(df, x)` — `Γ((df+1)/2) / (sqrt(df×π) × Γ(df/2)) × (1 + x²/df)^{-(df+1)/2}`; uses `f64_tgamma` builtin
- [ ] `beta_pdf(α, β, x)` — `pow(x,α-1) × pow(1-x,β-1) / B(α,β)` where B = Γ(α)Γ(β)/Γ(α+β); uses `f64_tgamma`
- [ ] `chi_squared_pdf(k, x)` — `pow(x,k/2-1) × exp(-x/2) / (pow(2,k/2) × Γ(k/2))`
- [ ] `gamma_pdf(shape, rate)` — uses `f64_tgamma`
- [ ] `laplace_pdf(μ, b, x)` — `exp(-|x-μ|/b) / (2b)`
- [ ] `log_normal_pdf` — uses `f64_normal_pdf` builtin + builtin `log`

**Hard (requires regularised incomplete beta — see §Hard below):**
- [ ] `t_cdf`, `beta_cdf`, `chi_squared_cdf` — all need `beta_incomplete`

### Correlation and regression

- [ ] `covariance` — online algorithm: two-pass or Welford for numerical stability
- [ ] `pearson_r` — `covariance(xs,ys) / (std_dev(xs) * std_dev(ys))`
- [ ] `spearman_r` — rank-transform xs and ys (requires sort — blocked on F64-1), then `pearson_r`
- [ ] `ols_slope` — `covariance(xs,ys) / variance(xs)`
- [ ] `ols_intercept` — `mean(ys) - ols_slope * mean(xs)`
- [ ] `ols_r_squared` — `pearson_r(xs,ys)²`

### Information theory

- [ ] `shannon_entropy(ps)` — `for p in ref ps { s = s - p * log2(p); }` using builtin `log2`
- [ ] `kl_divergence(ps, qs)` — `sum p * log(p/q)` using builtin `log`
- [ ] `cross_entropy(ps, qs)` — `-sum p * log(q)` using builtin `log`
- [ ] `mutual_information(ps_xy, ps_x, ps_y)` — double sum
- [ ] `renyi_entropy(ps, alpha)` — `log(sum pow(p,alpha)) / (1-alpha)` for alpha ≠ 1

### Hypothesis testing (statistics only — p-values need CDF)

- [ ] `t_stat_one_sample` — `(mean - μ0) / (std_dev / sqrt(n))`; uses builtin `sqrt`
- [ ] `t_stat_two_sample` — Welch: `(mean_x - mean_y) / sqrt(var_x/nx + var_y/ny)`
- [ ] `chi_squared_stat` — `sum (obs - exp)² / exp`
- [ ] `z_stat` — `(mean - μ0) / (σ / sqrt(n))`
- [ ] z-test p-value — wrap `f64_normal_cdf` builtin (no extra implementation needed)

---

## Hard items (~ 2 weeks, after core is done)

### Regularised incomplete beta `beta_incomplete(a, b, x)`

Required by `t_cdf`, `beta_cdf`, `chi_squared_cdf`. No compiler builtin exists.

- [ ] Implement Lentz's continued-fraction algorithm for `beta_incomplete(a, b, x)`
      — ~80 lines of vāṇī; requires careful convergence guard and `while` loop
- [ ] Add symmetry relation: if `x > (a+1)/(a+b+2)` use `1 - beta_incomplete(b, a, 1-x)`
- [ ] `t_cdf(df, t)` — `1 - beta_incomplete(df/2, 0.5, df/(df+t²)) / 2`
- [ ] `beta_cdf(α, β, x)` — wraps `beta_incomplete(α, β, x)`
- [ ] `chi_squared_cdf(k, x)` — regularised incomplete gamma `gamma_lower(k/2, x/2) / Γ(k/2)`
      (also needs `gamma_incomplete` — similar algorithm, lower priority)

---

## Safety / WCET annotations (after all implementations complete)

- [ ] Add `#[wcet(cycles=N)]` and `#[bounded_stack(bytes=N)]` to all public functions.
- [ ] All loops over `Vec<f64>` inputs are O(n) — document expected budget as `N × cycles_per_iter`.
- [ ] `beta_incomplete` continued-fraction: document max iterations as the bound.
      The WCET checker will report UNBOUNDED until `#[bounded(max_iter)]` is added.
