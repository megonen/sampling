# Adversary Attack Surface Calculator

**Live:** [adversary_calculator.html](https://megonen.github.io/da_calculators/adversary_calculator.html)

Interactive calculator for analysing the capabilities of an adversary who controls a fraction of DA nodes in the Logos Blockchain DA layer. Derives general formulas for two complementary attacks and shows how each protocol parameter shapes the adversary's effective capability as a function of the adversarial node fraction p_d.

---

## Background

This calculator assumes a **no-false-proof adversary**: adversarial nodes always respond with valid proofs when they hold the data, and stay silent when they do not. The attack strategy — respond or stay silent — is chosen by the adversary depending on the goal. The analysis of adversarial nodes that send false proofs (equivocation) is a separate topic.

Two attacks are modelled:

**Attack A — Available → Unavailable (Type II exploitation)**
The data is genuinely available. The adversary sets p_dis = 0 — controlled nodes withhold responses from targeted validators, causing them to observe fewer than τ successes and declare the data unavailable. This is a liveness threat.

**Attack B — Unavailable → Available (Type I exploitation)**
The adversary sets p_dis = 1 — it disperses data only to subnetworks where it controls at least one node, then responds with valid proofs. The adversary distributes data to at most K subnetworks so the data remains globally unrecoverable. It greedily selects the K subnetworks with the highest adversarial sampling probability p_samp(a, R, t) to maximise the chance of fooling the validator's sample. This is a safety threat.

---

## Parameters

| Parameter | Description | Default |
| --- | --- | --- |
| N | Total subnetworks = total columns in expanded data | 2048 |
| e | RS expansion factor | 2 |
| R | Nodes assigned to each subnetwork | 5 |
| S | Subnetworks sampled per validation round | 20 |
| τ | Acceptance threshold (declare available if ≥ τ successes out of S) | 13 |
| t | Nodes queried per subnetwork (1 ≤ t ≤ R); validator stops at first valid response | 5 |
| p_d | Adversarial node fraction (%) | 33 |

---

## Core Formulas

**Subnetwork false-failure probability** — probability all t queried nodes are adversarial (Attack A):

```
P_fail(a, R, t) = C(a,t) / C(R,t)    for a ≥ t,  else 0
```

**Adversarial sampling probability** — probability at least one of t queried nodes is adversarial (Attack B):

```
p_samp(a, R, t) = 1 − C(R−a, t) / C(R, t)    for a ≥ 1,  else 0
```

Note: P_fail and p_samp are not complements. P_fail is the probability all t draws are adversarial; p_samp is the probability at least one draw is adversarial. They are equal only when t = 1. For t > 1, P_fail ≤ p_samp.

**Effective false-failure probability** — under the binomial approximation (large n, fixed p_d):

```
P_fail_eff(p_d, t) = p_d^t
```

This is independent of R. The sum over occupancy telescopes exactly to p_d^t regardless of R.

**Attack A probability** — validator needs S−τ+1 failures:

```
P_A(p_d, t, S, τ) = Σ_{j=S−τ+1}^{S} C(S,j) · (p_d^t)^j · (1−p_d^t)^{S−j}
```

**Attack B probability** — adversary greedily selects K subnetworks with highest p_samp:

```
p_eff(p_d, R, t, K) = (1/N) · Σ_{chosen K subnets} p_samp(a, R, t)

P_B(p_d, R, t, N, S, τ) = Σ_{g=τ}^{S} C(S,g) · p_eff^g · (1−p_eff)^{S−g}
```

p_eff is bounded above by K/N = 1/e, producing a plateau in P_B at P(Bin(S, 1/e) ≥ τ).

---

## Regime Thresholds

Both P_A and P_B grow continuously with p_d. The thresholds below are descriptive heuristics, not hard boundaries:

| Zone | Condition | Note |
| --- | --- | --- |
| Attack A negligible | p_d ≪ (1−τ/S)^{1/t} | P_B may still be non-negligible at high t |
| Attack A active | p_d around (1−τ/S)^{1/t} | Liveness risk; P_B growing |
| Both significant | p_d large relative to crossover | Liveness + safety risk |
| Global recovery threat | p_d ≥ (1−1/e)^{1/R} | Hard boundary for recoverability only |

**Attack A mean-field crossover** — (1−τ/S)^{1/t}: the p_d where expected failures S·p_d^t = S−τ. Note P_A ≈ 0.4 at this crossover — it is a midpoint, not the onset. In the special case t = R this becomes (1−τ/S)^{1/R}.

**Global recovery threshold** — (1−1/e)^{1/R}: hard boundary where the adversary can threaten data recoverability. Always ≥ 1/2 (equality only at e=2, R=1). This is NOT the onset of Attack B — Attack B grows continuously and may be non-negligible well below this threshold.

---

## Tabs

### Attack A vs B
Shows both P_A and P_B as functions of p_d at current parameters. P_A and P_B values are shown directly — both grow continuously.

### τ effect
τ has opposite effects on the two attacks:
- Higher τ → Attack A easier (S−τ+1 failures needed; τ=S means only 1 failure needed)
- Higher τ → Attack B harder (τ hits needed; τ=S means all S hits needed)

### t effect
t is a tradeoff parameter:
- Higher t → Attack A harder (P_fail_eff = p_d^t decreases; crossover shifts right)
- Higher t → Attack B easier (p_samp increases for partially captured subnetworks)

### R effect
R is the primary structural defence for **Attack B** (p_samp and K-selection depend on R exponentially). For Attack A, P_fail_eff = p_d^t is R-independent — t is the primary Attack A defence parameter.

---

## Key Findings

- **τ = S** is the worst setting for Attack A (only 1 failure needed). The jointly optimal (τ*, t*) must balance both attacks.
- **P_fail_eff = p_d^t** — independent of R under the binomial approximation. t is the primary Attack A defence parameter.
- **R is primary for Attack B**, not Attack A. R affects p_samp and K-selection where capture probabilities scale exponentially.
- **Attack B has no hard onset threshold.** It grows continuously and can be non-negligible well below the global recovery threshold, especially at higher t. The plateau at P(Bin(S, 1/e) ≥ τ) is reached when p_eff saturates at K/N.

---

## Implementation

Built with React 18 and Recharts. P_fail_eff uses the closed-form p_d^t (binomial approximation). P_B uses the greedy K-selection upper bound with binomial sampling. τ and t value ranges scale automatically with S and R. No server required — open directly in any modern browser.