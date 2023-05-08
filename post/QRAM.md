References:
- Giovannetti, V., Lloyd, S. and Maccone, L. (2008a) ‘Architectures for a quantum random access memory’, Physical Review A, 78(5), p. 052310. Available at: https://doi.org/10.1103/PhysRevA.78.052310.
- Giovannetti, V., Lloyd, S. and Maccone, L. (2008b) ‘Quantum Random Access Memory’, Physical Review Letters, 100(16), p. 160501. Available at: https://doi.org/10.1103/PhysRevLett.100.160501.

Aim:
$$\sum_j \psi_j \ket{j}_a \xrightarrow{\text{QRAM}} \sum_j \psi_j \ket{j}_a \ket{D_j}_d$$
- Address register $a$: superposition of addresses.
- Data register $d$: superposition of memory cells.

![[QRAM.png]]
$$\begin{cases}
	U \ket{0} \ket{\text{wait}} & = \ket{f}  \ket{\text{left}} \\
	U \ket{1} \ket{\text{wait}} & = 
	\ket{f} \ket{\text{right}}
\end{cases}$$

Each time send a bit. According to that bit, transform a trit to left or right. In each layer, one and only one trit has been changed.

For machine learning task, $\lbrace \vec{x}_i\rbrace$ where $\vec{x}_i \in \mathbb{C}^N$ and $i = 1 \ldots M$ .
Using $O(MN)$ hardware resources, but only $O(\log MN)$ operations to access them.