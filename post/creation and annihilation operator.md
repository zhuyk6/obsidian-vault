# Definition
The Hamiltonian for a particle in a one-dimensional parabolic potential is
$$H = \frac{p^2}{2m} + {1\over 2} m\omega^2 x^2$$
where $p$ is momentum operator, $m$ is mass, $x$ is position operator, $\omega$ is related to the potential depth.

- creation: $a^\dagger$   
- annihilation: $a$
where
$$\begin{align}
a & = \frac{1}{\sqrt{2m \hbar \omega}} \left( m\omega x + ip \right) \\
a^\dagger & = \frac{1}{\sqrt{2m \hbar \omega}} \left( m\omega x - ip \right) 
\end{align}$$
Then the Hamiltonian can be rewritten as 
$$H = \hbar \omega \left( a^\dagger a + {1\over 2} \right)$$

# Properties
## ceate or delete state
The eigenstates $|n\rangle$ of $H$, where $n = 0, 1, \ldots$ 
$$\begin{align}
a^\dagger a |n\rangle & = n |n\rangle \\
a^\dagger |n\rangle & = \sqrt{n+1} | n+1\rangle \\
a |n\rangle & = \sqrt{n} |n-1\rangle
\end{align}$$

## Anticommutation relations
anticommutator: $\lbrace A, B\rbrace\equiv AB + BA$
$$\begin{align}
\lbrace a_i, a_j^\dagger \rbrace & = \delta_{i,j} \\
\lbrace a_i, a_j \rbrace & = \lbrace a_i^\dagger , a_j^\dagger \rbrace = 0
\end{align}$$

