
The following provides a one-to-one correspondence between linear maps $\mathcal(E) \in \mathcal{B}(M_d, M_d′)$ and operators $\mathcal{J} \in \mathcal{B}(C_d′ ⊗ C_d)$:
$$
\begin{align}
\mathcal{J}_\mathcal{E} & := \left( T \otimes \text{id}_d \right) \left( | \Omega\rangle \langle \Omega|\right) = \sum_{i,j} |i\rangle \langle j| \otimes \mathcal{E}(|i \rangle \langle j|) \\
\mathcal{E}(\rho) & := \text{Tr}_A \left[ \mathcal{J}_\mathcal{E} \left( \rho^\text{T} \otimes \mathbb{I}_B \right) \right]
\end{align}
$$
where $| \Omega \rangle = \frac{1}{\sqrt{d}} \sum_i |ii\rangle$ is the maximally entangled state.

And if $\mathcal{J}_\mathcal{E}$ is positive if and only if $\mathcal{E}$ is completely positive.

Here gives how to transform $\mathcal{J}_\mathcal{E}$ to $\mathcal{E}$ :
$$
\begin{align}
\mathcal{J}_\mathcal{E} \left( \rho^\text{T} \otimes \mathbb{I}_B \right)
& = \left[\sum_{i, j} |i\rangle \langle j| \otimes \mathcal{E}(|i \rangle \langle j|) \right] \left[\sum_{p, q} \rho_{p, q} |q \rangle \langle p|  \otimes \mathbb{I}_B \right] \\
& = \sum_{i, j, p} \rho_{p, j} |i \rangle \langle p| \otimes \mathcal{E}(|i \rangle \langle j|)
\end{align}
$$
After tracing the part A:
$$
\text{Tr}_A \left[\mathcal{J} \left( \rho^\text{T} \otimes \mathbb{I}_B \right)\right]
= \sum_{i, j} \rho_{i, j} \mathcal{E} (|i\rangle \langle j|) = \mathcal{E} (\rho)
$$
