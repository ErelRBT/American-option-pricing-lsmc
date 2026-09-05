# American Option Pricing with Least-Squares Monte Carlo

A Python implementation of **American option pricing using the Least-Squares Monte Carlo (LSMC) method**, with custom random number generation, geometric Brownian motion simulation, polynomial regression and comparison with European Black-Scholes prices.

The project implements the main components of the pricing process from scratch in order to illustrate the numerical methods used in quantitative finance.

## Overview

American options differ from European options because they can be exercised at any time before maturity.

This early-exercise feature makes them more difficult to price analytically.

This project uses a simulation-based approach inspired by the **Longstaff-Schwartz Least-Squares Monte Carlo method**.

The algorithm:

1. Generates pseudo-random uniform variables.
2. Converts them into standard normal variables using the Box-Muller transformation.
3. Simulates asset-price paths under a geometric Brownian motion.
4. Computes the option payoff at maturity.
5. Works backward through time.
6. Estimates the continuation value using polynomial least-squares regression.
7. Compares immediate exercise with the estimated continuation value.
8. Determines the estimated American option price.
9. Compares the result with European option prices obtained from the Black-Scholes formula.

---

## Geometric Brownian Motion

The underlying asset is simulated under the risk-neutral geometric Brownian motion:

$$
dS_t = (r-q)S_t,dt + \sigma S_t,dW_t
$$

Using a discrete time step:

$$
S_t
\exp\left(
\left(r-q-\frac{1}{2}\sigma^2\right)\Delta t
+
\sigma\sqrt{\Delta t},Z
\right)
$$

where:

$$(S_t)$$ = asset price at time (t)

$$(r)$$ = risk-free interest rate

$$(q)$$ = dividend yield

$$(\sigma)$$ = volatility

$$(Z \sim N(0,1))$$

$$(\Delta t = T/M)$$

---

## Random Number Generation

Instead of directly relying on NumPy's normal random generator, the project implements the random-number-generation process explicitly.

### Linear Congruential Generator

Uniform pseudo-random variables are generated using:

$$
X_{n+1} = (aX_n+c) \mod m
$$

and transformed into values between 0 and 1.

```python
def lcg(seed, a, c, m, n):
    x = seed
    xi = []

    for i in range(n):
        x = (a*x + c) % m
        xi.append(x/m)

    return xi
```

### Box-Muller Transformation

The generated uniform variables are converted into standard normal variables using the Box-Muller transformation:

$$
Z_1 = \sqrt{-2\ln(U_1)}\cos(2\pi U_2)
$$

$$
Z_2 = \sqrt{-2\ln(U_1)}\sin(2\pi U_2)
$$

These variables are then used to simulate the stochastic component of the asset-price paths.

---

## Monte Carlo Path Simulation

The function `SimChemin()` generates \(N\) simulated price trajectories with \(M\) possible exercise dates.

The resulting matrix has the structure:

```text
                t0       t1       t2      ...       T

Path 1          S0      S1,1     S1,2     ...      S1,M
Path 2          S0      S2,1     S2,2     ...      S2,M
...
Path N          S0      SN,1     SN,2     ...      SN,M
```

Each trajectory follows the risk-neutral geometric Brownian motion.

---

## Option Payoffs

The project supports both calls and puts.

For a call:

$$
C_T = \max(S_T-K,0)
$$

For a put:

$$
P_T = \max(K-S_T,0)
$$

These are implemented through:

```python
def CALL(K, St):
    return max(St-K, 0)

def PUT(K, St):
    return max(K-St, 0)
```

---

## Least-Squares Monte Carlo

The main pricing function is:

```python
ValAmerican(N, M, deg, T, tdm, type)
```

At maturity, the cash flow of each trajectory is equal to the terminal option payoff.

The algorithm then moves backward through the simulated exercise dates.

At every date, it estimates the expected continuation value of the option through a polynomial regression:

$$
C(S_t)
\approx
\beta_0
+
\beta_1S_t
+
\beta_2S_t^2
+
...
+
\beta_dS_t^d
$$

The regression is solved using NumPy's least-squares implementation:

```python
beta, *_ = np.linalg.lstsq(A, Y, rcond=None)
```

For every simulated trajectory, the algorithm then compares:

$$
\text{Immediate Exercise Value}
$$

with:

$$
\text{Estimated Continuation Value}
$$

If immediate exercise is more valuable, the option is exercised. Otherwise, the simulated position is kept alive.

---

## In-the-Money Regression

The implementation allows the regression to use either:

* only **in-the-money (ITM)** paths;
* or **all simulated paths**.

This behaviour is controlled through:

```python
tdm = True
```

for in-the-money trajectories only, or:

```python
tdm = False
```

for all trajectories.

This makes it possible to compare how the regression sample affects the estimated option value.

---

## European Black-Scholes Benchmark

European call and put prices are also calculated using the Black-Scholes formula.

For the call:

$$
C = S_0e^{-qT}N(d_1) - Ke^{-rT}N(d_2)
$$

For the put:

$$
P = Ke^{-rT}N(-d_2) - S_0e^{-qT}N(-d_1)
$$

with:

$$
d_1 = \frac{ \ln(S_0/K) + (r-q+\frac{1}{2}\sigma^2)T }{ \sigma\sqrt{T} }
$$

and:

$$
d_2=d_1-\sigma\sqrt{T}
$$

The European prices provide a useful analytical benchmark for the simulation-based American option valuation.

---

## Parameters

The notebook currently uses the following market parameters:

```python
S0 = 1       # Spot price
K = 0.1      # Strike price
sig = 1      # Volatility
r = 0.017    # Risk-free rate
T = 1        # Maturity
q = 1        # Dividend yield
```

Simulation parameters:

```python
deg = 2      # Polynomial regression degree
N = 100      # Number of simulated paths
M = 30       # Number of exercise dates
lamb = 0.95  # Confidence level
```

These values can easily be modified in the notebook to test different market conditions and numerical configurations.

---

## Project Structure

```text
american-option-pricing-lsmc/
│
├── projet.ipynb
└── README.md
```


---

## Dependencies

The project uses:

* Python
* NumPy
* SciPy
* Pandas
* Matplotlib
* Jupyter Notebook

The main numerical computations rely primarily on **NumPy**, **SciPy** and Python's standard `math` library.

---

## Key Concepts Demonstrated

This project covers several concepts relevant to quantitative finance and computational mathematics:

* Monte Carlo simulation
* American option pricing
* European option pricing
* Black-Scholes model
* Least-squares regression
* Longstaff-Schwartz methodology
* Geometric Brownian motion
* Risk-neutral valuation
* Early exercise decisions
* Linear Congruential Generators
* Box-Muller transformation
* Polynomial basis functions
* Numerical option pricing

---

## Possible Improvements

Several extensions could further develop the project:

* Increase the number of Monte Carlo simulations.
* Study convergence as a function of the number of paths.
* Compare different polynomial degrees.
* Add alternative regression basis functions.
* Visualize simulated asset-price trajectories.
* Compare American and European option prices systematically.
* Compare results with a binomial-tree benchmark.
* Implement variance-reduction techniques.
* Vectorize the Monte Carlo simulation for better performance.
* Separate the notebook into reusable Python modules.
* Add unit tests.
* Improve the statistical construction of confidence intervals.

---

## Purpose

The objective of this project is educational: to implement and understand the numerical building blocks behind American option valuation rather than relying entirely on pre-built pricing libraries.

It combines probability, stochastic simulation, regression and derivative pricing within a single Python implementation.
