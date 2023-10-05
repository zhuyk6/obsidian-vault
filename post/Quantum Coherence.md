

# Usage

## The maximally coherent state

Any $d \times d$ state $\hat{\rho}$ can be prepared from $\left| \Psi _d \right\rangle$ by using only incoherent operations.

Let 
$$
K_{n} := \sum_{i=1}^{d} c_i \left| i  \right\rangle \left\langle m_{i + n - 1} \right|
$$
where $\sum_{i=1}^{d} \left\vert c_i  \right\vert^2 = 1$ and $m_{x} = \mod(x-1, d) + 1 = x - \left\lfloor \frac{x-1}{d} \right\rfloor d$.
That is 
$$
\begin{align*}
  K_1 & = \left| 1 \right\rangle \left\langle 1 \right| + \left| 2 \right\rangle \left\langle 2 \right| + \cdots + \left| d \right\rangle \left\langle d \right| \\
  K_2 & = \left| 1 \right\rangle \left\langle 2 \right| + \left| 2 \right\rangle \left\langle 3 \right| + \cdots + \left| d-1  \right\rangle \left\langle d  \right| + \left| d \right\rangle \left\langle 1 \right| \\
  \vdots
\end{align*}
$$

It can by verified that 
$$
K_n^\dagger K_n = \sum_{i=1}^{d} \left\vert c_i \right\vert^2 \left| m_{i+n-1} \right\rangle \left\langle m_{i+n-1 } \right|
$$
then
$$
\begin{align}
\sum_{n=1 }^{d} K_n^\dagger K_n & = \sum_{n=1 }^{d} \sum_{i=1}^{d} \left\vert c_i \right\vert^2 \left| m_{i+n-1} \right\rangle \left\langle m_{i+n-1 } \right| \\
 & =\sum_{i=1}^{d} \left\vert c_i \right\vert^2 \sum_{n=1 }^{d}  \left| m_{i+n-1} \right\rangle \left\langle m_{i+n-1 } \right| \\
 & =\sum_{i=1}^{d} \left\vert c_i \right\vert^2 \sum_{n=1 }^{d}  \left| n\right\rangle \left\langle n \right| \\
 & =\sum_{i=1}^{d} \left\vert c_i \right\vert^2 I = I
\end{align}
$$

Next we need to verify it is incoherent operation. For any $\hat{\sigma } = \sum_{i=1}^{d} \sigma _i  \left| i \right\rangle \left\langle i \right| \in \mathcal{I}$, 
$$
\begin{align}
  K_n \hat{\sigma } K_n^\dagger 
& = \sum_{i=1}^{d} c_i \left| i  \right\rangle \left\langle m_{i+n-1 } \right| \sum_{k=1 }^{d} \sigma _k \left| k \right\rangle \left\langle k \right| \sum_{j=1 }^{d} c_j^\dagger \left| m_{j+n-1 } \right\rangle \left\langle j \right| \\
& = \sum_{i,j,k = 1 }^{d} c_i c_j^\dagger  \sigma _k \delta _{k, m_{i+n-1 }} \delta _{k, m_{j+n-1 }} \left|i   \right\rangle \left\langle j \right| \\
& = \sum_{i=1}^{d} \left\vert c_i  \right\vert^2 \sigma _{m_{i+n-1 }} \left|i   \right\rangle \left\langle i \right| \quad \in \mathcal{I}
\end{align}
$$

Now let's see what will happen when this operation applies to $\Psi _m$ :
$$
K_n \left| \Psi _m  \right\rangle = \frac{1}{\sqrt{d}} \sum_{i=1}^{d} c_i \left|i   \right\rangle
$$
then 
$$
K_n \left| \Psi _m \right\rangle \left\langle \Psi _m \right| K_n^\dagger = \frac{\left| \phi  \right\rangle \left\langle \phi  \right|}{d}
$$
where $\left| \phi  \right\rangle = \sum_{i=1}^{d} c_i \left|i  \right\rangle$. That means $\Psi _m \rightarrow \phi$ with probability $1 / d$ .

If we want to transform $\Psi _m$ to $\rho$, where $\rho = \sum_{l } q_l \left| \phi _l \right\rangle \left\langle \phi _l \right|, \sum_l q_l = 1, \left| \phi _l \right\rangle = \sum_i c_i^{(l )} \left| i \right\rangle$, then let
$$
K_n^{(l )} := \sqrt{q_l } \sum_{i=1}^{d} c_i^{(l )} \left| i  \right\rangle \left\langle m_{i+n-1 } \right| 
$$
It can be easily verified that it sum to identity and it's incoherent. And 
$$
\sum_{n, l=1}^d K_n^{(l )} \Psi _m {K_n^{(l )}}^\dagger  = \sum_{l=1 }^{d} q_l \left| \phi _l \right\rangle \left\langle \phi _l \right| = \rho
$$

