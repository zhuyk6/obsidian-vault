![[Jensen's inequality.png]]

If $f$ is convex on $[L, R]$, then $\forall x_1, x_2\in [L, R]$ and $\forall t\in [0,1]$ the following inequality holds. 
$$
f(tx_1 + (1-t)x_2) \leq tf(x_1) + (1-t)f(x_2)
$$

# Finite form
If $f$ is convex, $x_1, \ldots, x_n$ in its domain, positive weights $a_1, \ldots, a_n$ , then
$$
f\left(\frac{\sum_i a_i x_i}{\sum_i a_i}\right) \leq \frac{\sum_i a_i f(x_i)}{\sum_i a_i}
$$
Equality holds if and only if $x_1=x_2=\cdots =x_n$ or $f$ is linear.

The inequality is reverse if $f$ is concave:
$$
f\left(\sum_i a_i x_i \over \sum_i a_i\right) \geq \frac{\sum_i a_i f(x_i)}{\sum_i a_i}
$$
# Probabilistic form
$\varphi$ is convex, then
$$
\varphi\left(E(X)\right) \leq E(\varphi (X))
$$
# Proof: finite form
Finite form can be written as: $\varphi$ is convex, $\lambda_1, \ldots, \lambda_n \geq 0$ and $\sum_i \lambda_i = 1$, then for any $x_1, x_2, \ldots, x_n$ 
$$
\varphi(\lambda_1x_1+\lambda_2 x_2 + \cdots \lambda_nx_n) \leq \lambda_1 \varphi(x_1) + \cdots \lambda_n \varphi(x_n)
$$

This can be proved by induction.
For $n=2$, this statement is true because of convexity hypothesis.
Suppose the statement is true for some $n$, so
$$
\varphi(\lambda_1x_1+\lambda_2 x_2 + \cdots \lambda_nx_n) \leq \lambda_1 \varphi(x_1) + \cdots \lambda_n \varphi(x_n)
$$
for any $\lambda_1, \ldots, \lambda_n$ such that $\lambda_1 + \cdots + \lambda_n = 1$ .
Now $\lambda_1 + \cdots + \lambda_{n+1} = 1$, there must be one $\lambda < 0$ , let $\lambda_{n+1} < 0$, then
$$
\begin{align}
&\ \varphi\left( \sum_{i=1}^{n+1}\lambda_i x_i \right) \\
= &\ \varphi\left( \sum_{i=1}^n \lambda_i x_i + \lambda_{n+1} x_{n+1} \right) \\
= &\ \varphi\left( (1 - \lambda_{n+1})\sum_{i=1}^n \frac{\lambda_i}{1 - \lambda_{n+1}} x_i + \lambda_{n+1} x_{n+1} \right) \\
\leq &\ (1-\lambda_{n+1}) \varphi\left(\sum_{i=1}^n \frac{\lambda_i}{1-\lambda_{n+1}} x_i\right) + \lambda_{n+1} \varphi(x_{n+1}) \\
\leq &\ (1-\lambda_{n+1}) \left(\sum_{i=1}^n \frac{\lambda_i}{1-\lambda_{n+1}} \varphi(x_i)\right) + \lambda_{n+1} \varphi(x_{n+1}) \\
= &\ \sum_{i=1}^{n+1} \lambda_i \varphi(x_i)
\end{align}
$$
