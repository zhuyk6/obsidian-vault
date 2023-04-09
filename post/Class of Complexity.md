
# Decision problem
$L$ is said decidable iff there exists a Turing machine that can judge whether $w \in \Sigma^\star$ and always gives an answer. (**never loop, always halt**).

For a decision problem $L \subset \lbrace 0 , 1\rbrace^\star$ , for all input $w$, $L$ either accepts $w$ or reject $w$ .

# Promise problem
Promise problem is a generalization of a decision problem. For a promise problem, there are two languages $L_\text{Yes}$ and $L_\text{No}$ , which $L_\text{Yes} \cap L_\text{No} = \emptyset$ . The set $L_\text{Yes} \cup L_\text{No}$ is called the promise. 

If the input $w$ belongs to the promise, the Turing machine should give the right answer (accept or reject). If $w$ doesn't belong to the promise, the machine can give any answer or loop.

# Completeness

# Hardness




# P

# NP


# BPP

# BQP
[BQP(bounded-error quantum polynomial time)](https://en.wikipedia.org/wiki/BQP) is the class of decision problems solvable by a quantum computer in polynomial time, with an error probability of at most $1/3$ for all instances.

> success probability should be $\frac{1}{2} + \epsilon$ where $\epsilon > 0$ .
> 
> **(The Chernoff bound)**  Suppose $X_1,\ldots,X_n$ are independent and identically distributed random variables, each taking the value $1$ with probability $1/2 + \epsilon$	, and the value $0$ with probability $1/2 −\epsilon$	.Then
> $$p\left( \sum_{i=1}^n X_i \leq n/2 \right) \leq e^{-2\epsilon^2 n}$$
> Using this theorem, we can always repeat several times and get a algorithm with higher success probability.

# PSPACE

