---
title: Penalty method
course-code: NI-PON
cards-deck: NI-PON
created: 2022-05-23
tags: NI-PON
---

### Penalty method 
#card
Is a method for solving [Optimization problem](private/archive/school/optimization-problem.md) that is constrained with unconstrained $f'$, that
1. Contains function $f$ from the original problem
2. For each constrain there is penalization element, which is zero $\iff$ constrain is met, otherwise it's positive
3. Penalization elements are often multiplied in each step with some variable $\nu_k$. 
^1653308220084

#### Quadratic penalization
#card
$$Q(x, v) = f(x) + \frac{\nu}{2}\sum g(x)^2 + \frac{\nu}{2}\sum([h(x)]^+)^2$$
where $[y]^+ = max(y, 0)$
^1653308591980

#### Algorithm for the quadratic penalization
#card
1. declare $v_0 > 0$, and sequence $\tau_k$, $\tau_k \rightarrow 0$
2. for $k = \{0..1\}$ 
	1. find  approx minimum $x_k$ of the function $Q(x, \nu_k)$ with breaking condition
	   $||\nabla_xQ(x, \nu_k)|| \le \tau_k$,
	2. if the final condition for convergence is reached: return $x_k$
	3. otherwise set $\nu_{k+1} > \nu_k$
^1653309285137

#### $\mathcal{l}_1$ penalization
#card
names after the $\mathcal{l}_1$ norm
$$\varphi(x, \nu) = f(x) + \nu \sum |g(x)| + \nu \sum [h(x)]^+$$ 
^1653310664231
