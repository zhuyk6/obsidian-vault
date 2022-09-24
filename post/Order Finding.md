# Order
In number theory, if $x$ is co-prime to $N$, then $x \in Z_N^*$ , the order of $x$ is defined as the smallest positive integer such that
$$
x^r \equiv 1 \mod N 
$$
> $Z_N^* \equiv \lbrace x : \gcd(x, N)=1\rbrace$ which forms a group.

According to Euler's theorem, 
$$
x^{\varphi(N)} \equiv 1 \mod N
$$
therefore $r | \varphi(N)$, then $r \leq \varphi(N) < N$ .

# Unitary Operator
Let $L = \lceil \log_2 N\rceil$, that means we will use $L$ qubits. Here we define the unitary operator as
$$
U |y\rangle \equiv \begin{cases}
|x y \mod N\rangle & , 0 \leq x < N \\
|y\rangle & , y \geq N
\end{cases}
$$
Obviously $U$ only act on $[0, N)$, its effect is just times $x$.

The key point is what are eigenvalues and eigenvectors of $U$. From the definition of $U$, according to number theory, we know $(1, x, x^2, x^3, \ldots, x^{r-1})$ forms a ring. Therefore we must have
$$
\begin{aligned}
& \ U \frac{1}{\sqrt{r}} \left( |1\rangle + |x\rangle + |x^2 \rangle + \cdots |x^{r-1}\rangle \right) \\
= & \ \frac{1}{\sqrt{r}} \left( |x\rangle + |x^2\rangle + |x^3\rangle + \cdots |x^r\rangle \right) \\
= & \ |\psi_0\rangle
\end{aligned}
$$
We have found an eigenpair $(1, |\psi_0\rangle)$. Of course this is a trivial eigenpair and useless. But under this eigenpair, we can construct other pairs.

We just add some amplitudes $\lambda$ :
$$
\begin{aligned}
U |\psi_\lambda\rangle & = U \frac{1}{\sqrt{r}} \left( |1\rangle + \lambda |x\rangle + \lambda^2 |x^2\rangle + \cdots \lambda^{r-1}|x^{r-1}\rangle \right) \\
& = \frac{1}{\sqrt{r}} \left(\lambda^{r-1}|1\rangle + |x\rangle +\lambda |x^2\rangle  + \cdots \lambda^{r-2} |x^{r-1}\rangle\right) \\
& = \lambda^{-1} \frac{1}{\sqrt{r}} (\lambda^n |1\rangle + \lambda |x\rangle + \lambda^2 |x^2\rangle + \cdots \lambda^{r-1} |x^{r-1}\rangle)
\end{aligned}
$$
If we set $\lambda^r = 1$, then we find an eigenpair $(\lambda^{-1}, |\psi_\lambda\rangle)$ .

The function $\lambda^{r}=1$ has $r$ solutions $e^{2\pi i k\over r}$ , here we set $w_k = e^{- 2\pi i k / r}$.

Up to now we have found $r$ eigenpairs $(w_k, |\psi_k\rangle)$ for $k = 0, 1, \ldots r-1$. We push the order $r$ into the amplitude of $w_k$, so we can use phase estimation to calculate it.

But there is a question : how to prepare $|\psi_k\rangle$ ? The construction of $|\psi_k\rangle$ is based on the knowledge of $r$. So we can't prepare $|\psi_k\rangle$, but we can prepare the combination of them.
$$
\begin{aligned}
\frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} |\psi_k\rangle
& = \frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} e^{-2\pi i s k / r} | x^s\rangle \\
& = \frac{1}{r} \sum_{s=0}^{r=1} |x^s\rangle \sum_{k=0}^{r-1} e^{-2\pi i sk / r} \\
& = \sum_{s=0}^{r-1} \delta_{0,s} |x^s\rangle = |1\rangle
\end{aligned}
$$
Therefore we just need to prepare $|1\rangle$, which is the combination of $|\psi_k\rangle$ . Apply the phase estimation to this state, we will get the estimation of $s/r$ with probability $1/r$ for $s = 0, 1, \ldots , r-1$.

# Get the Order
After the phase estimation procedure, we get a real number $\phi \in \mathbb{Q}$ , how can we find the order $r$ from this $\phi$ ?

The key point is the [[Continued Fractions Algorithm]].

Using $t = 2L + 1 + \lceil \log_2 (2 + \frac{1}{2\epsilon}) \rceil$ qubits in the phase estimation, we will obtain $\phi \approx s/r$ accurate to $2L+1$ bits, with probability at least $(1-\epsilon) / r$ .

Using theorem in the [[Continued Fractions Algorithm]], $|s / r - \phi| \leq 1 / (2r^2)$ , then in the procedure of the continued factions algorithm of $\phi$, we will get the $s/r$ in some step.

But there is still a problem: the iterations of continued fractions algorithm are $p_n / q_n$ where $\gcd(p_n, q_n) = 1$, if $\gcd(s, r) \neq 1$, how to find $r$ ?
There are some methods to solve this problem. But in practical we always just check $x^r \equiv 1 \mod N$. If failed, just try again.