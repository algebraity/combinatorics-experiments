# Dyadic Sumet Asymptotics

## Introduction and motivation

We are interested in sets of the form

$$
A_n := \{i \cdot 2^j : i, j \in [n]\},
$$

where $[n] = \{1, 2, \ldots, n\} \subset \mathbb{Z}^+$, and their sumsets

$$
A_n + A_n.
$$

Pairs $(x_1, x_2), (x_3, x_4) \in A_n \times A_n$ producing the same sum correspond to solutions to the equation

$$
x_1 + x_2 = x_3 + x_4.
$$

The number of such quadruples is defined as the additive energy $E(A_n)$. Trivial solutions contribute $\Theta(|A_n|^2)$ to $E(A_n)$; any additional contribution corresponds to genuine additive coincidences.

High additive energy typically arises from strong additive structure, e.g. additive progressions, which $A_n$ seems to lack. Instead, $A_n$ is organized multiplicatively, with each element having a unique dyadic representation $u2^t$ with $u$ odd. Any additive coincidence imposes rigid constraints on the $2$-adic valuations of their summands, highly restricting the additive energy.

Thus, as $n \to \infty$, the additive energy should be negligible relative to $|A_n|^2$. Therefore $|A_n + A_n|$ is expected to be asymptotically equal to the number of unordered pairs from $A_n$, i.e.

$$
|A_n + A_n| \sim \frac{|A_n|^2}{2}.
$$

Our goal is to gather empirical evidence for this conjecture, and to form a conjecture regarding the rate at which $|A_n + A_n| \to \frac{|A_n|^2}{2}$ as $n \to \infty$.

---

## Computational methodology

Since $|A_n| \sim \Theta(n^2)$, computing the sum $x_1 + x_2$ for each $x_1, x_2 \in A_n$ would involve computing $\Theta(n^4)$ sums. This is relatively inefficient, and would make computing $|A_n + A_n|$ for large values of $n$ computationally infeasible. Fortunately, there is a more efficient technique requiring only $\Theta(n^3)$ arithmetic operations, which I call the "dyadic interval method."

### Dyadic interval method

For any $m \in \mathbb{Z}$, there is a unique representation $m = u \cdot 2^t$, where $u$ is odd and $t = v_2(m)$, the $2$-adic valuation of $m$. It is natural to group $x \in A_n$ by $u$ to compute $|A_n + A_n|$.

For each odd $u$, it is only necessary to know which exponents $t$ occur s.t. $u2^t \in A_n + A_n$. Thus:

$$
|A_n + A_n| = \sum_{u\text{ odd}}|\{t : u2^t \in A_n + A_n\}|.
$$

Doing so allows the four parameter set

$$
\{i2^j + i'2^{j'} : i, j, i', j' \in [n]\}
$$

to a family of sets defined by just three parameters.

Let $j_2 > j_1 > 0$, and take $a2^{j_1}, b2^{j_2} \in A_n$ such that

$$
a2^{j_1} + b2^{j_2} = 2^{j_1}(a + b2^{j_2 - j_1}).
$$

Let $d := j_2 - j_1$. Then for some odd $u$, and $\tau = v_2(a+b2^d)$, we have

$$
a2^{j_1} + b2^{j_2} = u2^{j_1 + \tau}.
$$

For a fixed triple $(a, b, d)$, $u$ is fixed, and $j_1 + \tau$ traverses the interval $[1+\tau, (n-d)+\tau]$ as $j_1$ varies. Thus, each triple $(a, b, d)$ contributes a contiguous block of exponents for a single odd $u$.

This gives rise to an efficient algorithm to compute $|A_n + A_n|$:

1. Choose a triple $(a, b, d)$.
2. Compute $u = \text{odd}(a + b2^d)$ and $\tau = v_2(a+b2^d)$.
3. Add $I = [1 + \tau, (n-d) + \tau]$ to the set of valid $t$'s for that $u$.
4. Sum these sets over all $u$.

The result is a set of buckets for exponents, with each bucket being represented by a unique odd number $u$. To compute each bucket, the algorithm must iterate over $a \sim n$, $b \sim n$, and $d \sim n$, resulting in an expected complexity of $\Theta(n^3)$ arithmetic operations. This is much more efficient than the "naive method," which is $\Theta(n^4)$.

---

## Implementation

The algorithm outlined in Section 2 has been implemented in the form of a robust kernel written in C for speed and efficiency. It is accessed by a Python program which uses it to compute $|A_n|$, $|A_n + A_n|$, $\frac{1}{2} - \frac{|A|^2}{2}$, and the density of coincidental sums for all values of $n$ in a specified range, with specified increments. Currently, both arithmetic and geometric increments are being considered. After all of the data is computed, it is exported to a CSV file for later analysis.

So far, $|A_n + A_n|$ and related values have been computed for all $n \in [1, 500] \cap \mathbb{Z}$, and for every $25$th $n$ in the range $[500, 1000]$. Since each computation is expected to be take $\Theta(n^3)$ operations and uses a significant amount of memory, computing higher values of $n$ is considered challenging or infeasible. It may be possible to compute $|A_n + A_n|$ for a few higher values, e.g. $n = 1100, 1200, \ldots, 2000$, but this may require a more efficient implementation, as even $n=1000$ is observed to use over 70GB of RAM at once to complete.

The programs used to compute $|A_n + A_n|$ have all been validated for $n \in [1, 100] \cap \mathbb{Z}$ by manually computing the sumset using Python `list` and `int` objects, which have no size limit. There is no evidence to suggest divergence at higher values of $n$, and all arithmetic is performed using GMP to avoid overflow.. It is therefore safe to conclude that the data produced via the "diadic interval method" code is accurate for all $n$ computed in this project.

All of the data, along with the code used to compute it, is included in this repository.

---

## Results

The patterns observed in the data so far support the hypothesis that

$$
|A_n + A_n| \sim \frac{|A_n|^2}{2}.
$$

Contained in the `data` folder are plots generated using the data from $n \in [i, 500] \cap \mathbb{Z}$, for different values of $i$, as well as less dense data for higher values of $n$. They show how

$$
\frac{|A_n + A_n|}{|A|^2}
$$

monotonically approaches $\frac{1}{2}$. Each plot also includes a best-fit curve of the form

$$
\frac{1}{2} - Cn^{-a},
$$

where $a$ and $C$ are parameters. All of these best-fit curves have $R^2$ over $0.999$, indicating an excellent fit.

When examining these plots, a few very interesting trends can be observed. Firstly, both the constant $C$ and the absolute value of the exponent $a$ trend monotonically upwards, starting around $C = 2.166$ and $a = 0.779$, and reaching close to $C = 2.969$ and $a = 0.832$ by the final plot. This gives reason to believe that the dominant term of the decay is indeed $n^{-1}$, though for values up to $n=750$, there is significant impact from lower degree terms, preventing a quick $a \to 1$. plot_750_to_1000.png shows the tail from $n=750$ to $n=1000$ with the highest values of $C$ and $a$, showing a large jump in their magnitude, which is consistent with strong pre-asymptotic behavior.

It is also apparent that $R^2$ is monotonically increasing as the minimum value of $n$ plotted increases, starting at $0.999615$ in `plot_100_to_500.png` ($[100, 500]$) and ending up at $0.999961$ in `plot_400_to_500.png` ($[400, 500]$). While the $R^2$ value technically decreases slightly in the final two plots, I believe this is caused by the fact that the points are far less dense, and the fact that the $R^2$ is still greater than $0.999$ should be seen as strong evidence that this model is asymptotically very strong.

This data strongly supports the conclusion that

$$
\frac{|A_n + A_n|}{|A_n|^2} \to \frac{1}{2},
$$

and this convergence is monotonic for all computed values. Its convergence is also well-approximated by a power law $Cn^{-a}$, with affective exponent $a$ close to $1$. However, the instability of fitted parameters across windows implies that there is more going on. There may just be lower-order terms preventing the observation of asymptotic conditions at low $n$. On the other hand, the true rate of convergence may be something like

$$
\frac{C\log(n)}{n}
$$

instead of pure $n^{-1}$. A more thorough investigation is required to determine the full rate of convergence.
