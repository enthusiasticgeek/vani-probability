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
| F64-1 | `sort_by` on `Vec<f64>` via `fn(f64,f64)->i64` comparator | `quantile`, `iqr`, `mode`; replace insertion-sort workaround in `median` |
| F64-2 | `vec_median`, `vec_kth_smallest`, `vec_argmin`, `vec_argmax`, `vec_min`, `vec_max` on `Vec<f64>` | cleaner implementations of the above |
| F64-3 | `vec_fold`, `vec_map`, `vec_filter` on `Vec<f64>` | style preference; all are doable with manual loops now |

> **Key finding:** `for x in ref xs` over `Vec<f64>` **already works** (confirmed against
> `checker.rs:11661`). `xs[i]` indexing, `set(mut ref xs, i, v)`, `while`, and scalar f64
> arithmetic all work. All descriptive stats and distributions can be written **right now**
> with manual for-loops. Only `quantile` / `mode` are blocked on F64-1 (sort).
>
> **Interim workaround for median:** insertion sort via `set(mut ref xs, j, v)` —
> O(n²) but correct. Replace with `sort_by_f64` once F64-1 lands.

---

## Implemented in v0.1.0

### Descriptive statistics

- [x] `mean` — accumulate sum, divide by n; O(n) WCET = n × 12 cycles
- [x] `variance` — two-pass (mean then Σ(xᵢ−m)²); Bessel-corrected; O(2n)
- [x] `std_dev` — calls `variance` + builtin `sqrt`
- [x] `median` — insertion-sort a copy, return middle; O(n²); blocked: replace with sort_by_f64 once F64-1 lands
- [x] `skewness` — Fisher-Pearson: n/((n−1)(n−2)) × Σ((xᵢ−m)/s)³
- [x] `kurtosis` — adjusted excess kurtosis; n(n+1)/((n-1)(n-2)(n-3)) × Σ((xᵢ-m)/s)⁴ − 3(n-1)²/((n-2)(n-3))
- [x] `sample_min` — scan loop
- [x] `sample_max` — scan loop
- [x] `weighted_mean(xs, ws)` — Σ(wᵢ xᵢ) / Σwᵢ
- [x] `weighted_variance(xs, ws)` — reliability-weighted Bessel-corrected variance

### Blocked on F64-1 (sort on Vec<f64>) — deferred

- [ ] `quantile(xs, q)` — linear interpolation between sorted neighbors
- [ ] `iqr(xs)` — Q3 − Q1
- [ ] `mode(xs)` — most frequent value

### Discrete distributions

- [x] `binomial_pmf` — `i64_binomial(n,k)` × pow(p,k) × pow(1-p,n-k)
- [x] `binomial_cdf` — cumulative sum of PMF up to k
- [x] `poisson_pmf` — pow(λ,k) × exp(-λ) / i64_factorial(k)
- [x] `poisson_cdf` — cumulative sum via while loop
- [x] `geometric_pmf` — pow(1-p,k-1) × p
- [x] `geometric_cdf` — 1 − pow(1-p,k)  *(added beyond original plan)*
- [x] `negative_binomial_pmf` — i64_binomial(k+r-1,k) × pow(p,r) × pow(1-p,k)  *(added beyond original plan)*
- [x] `hypergeometric_pmf` — three i64_binomial calls  *(added beyond original plan)*

### Continuous distributions

> `f64_normal_pdf` and `f64_normal_cdf` are builtins — not reimplemented.

- [x] `exponential_pdf` — guard x ≥ 0; λ × exp(-λx)
- [x] `exponential_cdf` — 1 − exp(-λx)
- [x] `sample_exponential` — inverse CDF: −log(rand_f64()) / λ  *(added beyond original plan)*
- [x] `uniform_pdf` — 1/(b-a) for a ≤ x ≤ b, else 0
- [x] `uniform_cdf` — (x-a)/(b-a) clamped to [0,1]
- [x] `t_pdf(df, x)` — uses `f64_tgamma` builtin
- [x] `beta_pdf(α, β, x)` — B(α,β) via Γ functions; uses `f64_tgamma`
- [x] `chi_squared_pdf(k, x)` — pow × exp / (pow × Γ)
- [x] `gamma_pdf(shape, rate, x)` — uses `f64_tgamma`  *(added beyond original plan)*
- [x] `laplace_pdf(μ, b, x)` — exp(-|x-μ|/b) / (2b)  *(added beyond original plan)*
- [x] `log_normal_pdf(μ, σ, x)` — uses `f64_normal_pdf` + `log` builtins  *(added beyond original plan)*

**Hard (requires regularised incomplete beta — deferred):**
- [ ] `t_cdf`, `beta_cdf`, `chi_squared_cdf` — need `beta_incomplete`

### Correlation and regression

- [x] `covariance` — two-pass; numerically stable
- [x] `pearson_r` — covariance / (std_dev × std_dev)
- [x] `ols_slope` — covariance(xs,ys) / variance(xs)
- [x] `ols_intercept` — mean(ys) − slope × mean(xs)
- [x] `ols_r_squared` — pearson_r²  *(added beyond original plan)*
- [ ] `spearman_r` — blocked on F64-1 (rank-transform requires sort)

### Information theory

- [x] `shannon_entropy(ps)` — −Σ p log₂(p) [bits]; uses builtin `log2`
- [x] `kl_divergence(ps, qs)` — Σ p log(p/q) [nats]; uses builtin `log`
- [x] `cross_entropy(ps, qs)` — −Σ p log(q) [nats]; uses builtin `log`
- [x] `renyi_entropy(ps, alpha)` — log(Σ pᵢ^α) / (1−α) [nats]; α→1 limit handled  *(added beyond original plan)*
- [ ] `mutual_information(ps_xy, ps_x, ps_y)` — double sum; deferred

### Hypothesis testing

- [x] `t_stat_one_sample` — (mean − μ₀) / (std_dev / sqrt(n))
- [x] `t_stat_two_sample` — Welch: (x̄−ȳ) / √(s²ₓ/nₓ + s²ᵧ/nᵧ)
- [x] `chi_squared_stat` — Σ (obs−exp)² / exp
- [x] `z_stat` — (mean − μ₀) / (σ / sqrt(n))  *(added beyond original plan)*
- [ ] z-test p-value — wraps `f64_normal_cdf` builtin; deferred

---

## Safety / WCET annotations

- [x] `#[bounded_stack(bytes=N)]` added to all 43 public functions (worst-case call-chain depth)
- [ ] `#[wcet(cycles=N)]` — add to leaf functions with no loops once cycle counts are audited

---

## Hard items (deferred — ~2 weeks, after core is done)

### Regularised incomplete beta `beta_incomplete(a, b, x)`

Required by `t_cdf`, `beta_cdf`, `chi_squared_cdf`. No compiler builtin exists.

- [ ] Implement Lentz's continued-fraction algorithm for `beta_incomplete(a, b, x)`
      — ~80 lines of vāṇī; requires careful convergence guard and `while` loop
- [ ] Add symmetry relation: if `x > (a+1)/(a+b+2)` use `1 - beta_incomplete(b, a, 1-x)`
- [ ] `t_cdf(df, t)` — `1 - beta_incomplete(df/2, 0.5, df/(df+t²)) / 2`
- [ ] `beta_cdf(α, β, x)` — wraps `beta_incomplete(α, β, x)`
- [ ] `chi_squared_cdf(k, x)` — regularised incomplete gamma `gamma_lower(k/2, x/2) / Γ(k/2)`
      (also needs `gamma_incomplete` — similar algorithm, lower priority)
