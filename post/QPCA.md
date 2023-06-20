
# Simulation of $e^{i \rho \Delta t }$

The first important equation is as the following:
$$
\text{tr}_{1} \left[ e^{- i S \Delta t } \rho \otimes \sigma e^{i S \Delta t } \right] = \sigma - i \Delta t [\rho, \sigma ] + O(\Delta t^2 )
$$
where $S$ is the swap operator, which acts as $S \left| i   \right\rangle \left| j  \right\rangle = \left| j  \right\rangle \left| i  \right\rangle$.

**Proof:** 

Because of $S^{2} = I$, we can expand $e^{i S \Delta t}$ as following:
$$
\begin{align}
  e^{i S \Delta t} & = \sum_{k=0}^{\infty} \frac{\left( i S \Delta t \right)^{k}}{k!} \\
& = \left[ 1 - \frac{1}{2!} \left( \Delta t  \right)^{2} + \frac{1}{4!} \left( \Delta t  \right)^{4} \cdots \right] I 
 + i \left[ \Delta t - \frac{1}{3!} \left( \Delta t  \right)^{3} \cdots  \right] S \\
 & = \cos \Delta t I + i \sin \Delta t S 
\end{align}
$$

Expanding $e^{i S \Delta t }$ we can get 
$$
\begin{align}
  & e^{- i S \Delta t } \rho \otimes \sigma e^{i S \Delta t } \\
& = \left[ \cos \Delta t I - i \sin \Delta t S  \right] \rho  \otimes \sigma \left[ \cos \Delta t I + i \sin \Delta t S  \right] \\
& = \cos ^{2} \Delta t \rho \otimes \sigma + i \sin \Delta t \cos \Delta t \rho \otimes \sigma S \\
& - i \sin \Delta t \cos \Delta t S \rho \otimes \sigma + \sin ^{2} \Delta t S \rho \otimes \sigma S 
\end{align}
$$

$S$ is the swap operator, so $S \rho \otimes \sigma S = \sigma \otimes \rho$. But $S \rho \otimes \sigma$ is complex.

Here is a interesting lemma: $\text{tr}_{1} \left[ S \rho \otimes \sigma  \right] = \rho \sigma$ and $\text{tr}_{1} \left[ \rho \otimes \sigma S \right] = \sigma \rho$.

**Proof:** 
$$
\begin{align}
  \text{tr} _{1} \left[ S \rho \otimes \sigma  \right] 
& = \text{tr} _{1} \left[ S \left( \sum_{i }^{} \lambda _{i } \left| p_{i} \right\rangle \left\langle p_{i} \right| \right) \otimes \left( \sum_{j}^{ } \mu _{j} \left| q_{j} \right\rangle \left\langle q_{j} \right| \right) \right] \\
& = \text{tr} _{1} \left[ S \sum_{ij}^{ } \lambda _{i } \mu _{j} \left| p_{i} q_{j} \right\rangle \left\langle p_{i} q_{j} \right| \right] \\
& = \text{tr} _{1} \left[ \sum_{ij}^{ } \lambda _{i } \mu _{j} \left|  q_{j} p_{i} \right\rangle \left\langle p_{i} q_{j} \right| \right] \\
& = \sum_{ij }^{ } \lambda _{i } \mu _{j} \left\langle p_{i} | q_{j} \right\rangle \left| p_{i} \right\rangle \left\langle q_{j} \right| \\
& = \left( \sum_{i }^{ } \lambda _{i} \left| p_{i} \right\rangle \left\langle p_{i} \right| \right) \left( \sum_{j}^{ } \mu _{j} \left| q_{j} \right\rangle \left\langle q_{j} \right| \right) \\
& = \rho \sigma 
\end{align}
$$

Using this lemma, we can substitute into the equation:
$$
\begin{align}
    \text{tr} _{1} e^{-i S \Delta  t } \rho \otimes \sigma e^{i S \Delta  t }
& = \cos ^{2} \Delta t \sigma + \sin ^{2} \Delta t \rho - i \sin \Delta t \cos \Delta t [\rho , \sigma ] \\
& = \left( 1 - \sin ^{2} \Delta t  \right) \sigma + \sin ^{2} \Delta t \rho - \frac{i }{2} \sin 2 \Delta t [\rho , \sigma ] \\
& = \sigma - i \Delta t [\rho , \sigma ] + O(\Delta t^{2})
\end{align}
$$


Another important observation is that $\sigma - i \Delta t[\rho ,\sigma ] = e^{-i \rho \Delta t } \sigma e^{i \rho \Delta t } + O(\Delta t^{2})$. This is because 
$$
\begin{align}
  e^{i \rho  \Delta t } & = I + i \rho \Delta t + O(\Delta t^{2}) \\
  e^{-i \rho \Delta t } & = I - i \rho \Delta t + O(\Delta t^{2}) \\
  e^{-i \rho \Delta t } \sigma e^{i \theta \Delta t } & = \left[ I - i \rho \Delta t + O(\Delta t^{2}) \right] \sigma \left[ I + i \rho \Delta t + O(\Delta t^{2}) \right] \\
    & = \sigma + i \Delta t \sigma \rho - i \Delta t \rho \sigma + O(\Delta t^{2}) \\
    & = \sigma  - i \Delta t [\rho ,\sigma ] + O(\Delta t^{2})
\end{align}
$$


