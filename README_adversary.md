# Adversary Attack Surface Calculator

**Live:** [adversary_calculator.html](https://megonen.github.io/da_calculators/adversary_calculator.html)

Interactive calculator for analysing the capabilities of an adversary who controls a fraction of DA nodes in the Logos Blockchain DA layer.

---

## Background

This calculator assumes a **no-false-proof adversary**: adversarial nodes always respond with valid proofs when they hold the data, and stay silent when they do not. This assumption is grounded in the binding property of the polynomial commitment scheme.

Two attacks are modelled:

**Attack A -- Available to Unavailable (Type II exploitation)**
Liveness threat. The adversary withholds responses (p_dis = 0). Protocol stage exploited: sampling.

**Attack B -- Unavailable to Available (Type I exploitation)**  
Safety threat. The adversary selectively disperses data (p_dis = 1) to at most K subnetworks. Protocol stages exploited: dispersal + sampling.

---

## Functional Parameter Dependencies

```
(N, e) -> K=N/e -> (p_d, R, t, K) -> (q_A, q_B) -> (S, tau) -> (P_A, P_B)
Encoding    Budget    Dispersal         Per-subnet    Sampling    Validator
```

| Attack | Per-subnet | Depends on | Validator probability | Depends on |
| --- | --- | --- | --- | --- |
| A | q_A = p_d^t | p_d, t | P_A = P(Bin(S, q_A) >= S-tau+1) | q_A, S, tau |
| B | q_B = p_eff(p_d,R,t,K,N) | p_d, R, t, K=N/e | P_B = P(Bin(S, q_B) >= tau) | q_B, S, tau |

---

## Key Thresholds

| Threshold | Formula | Meaning |
| --- | --- | --- |
| Attack A crossover | (1-tau/S)^{1/t} | p_d where expected failures = failure budget |
| Attack B feasibility | 1-(1-1/e)^{1/R} | Min p_d for adversary to populate K subnetworks |
| Attack B suppression | tau > S/e | P_B exponentially suppressed; bounded by Bin(S, 1/e) tail |
| Global recovery | (1-1/e)^{1/R} | Hard threshold for data recoverability |

---

## Key Findings

- **tau = S** is the worst setting for Attack A (only 1 failure needed)
- **q_A = p_d^t** is R-independent under binomial approximation
- **R is primary for Attack B**, not Attack A
- **tau > S/e exponentially suppresses Attack B** — P_B is nonzero but bounded by P(Bin(S,1/e) ≥ tau), which drops rapidly as tau exceeds S/e
- **Attack B feasibility threshold** is much lower than global recovery threshold

---

## Implementation

Built with React 18 and Recharts. No server required.
