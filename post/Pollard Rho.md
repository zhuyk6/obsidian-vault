# Factoring integer
Given $n$ , $n$ is not a prime, suppose $n = pq$, where $1 < p < n$. The problem is to find $p$.
Using a polynomial $g(x)=x^2+c \mod n$, we can generate a sequence
$$x_0, x_1 = g(x_0), x_2 = g(x_1) = g(g(x_0)), \ldots$$
Consider this sequence mod $p$,
$$
x_0 \mod p, x_1 \mod p, \ldots
$$
Using birthday paradox, after about $O(\sqrt{p})$ steps, we can find $x_i \neq x_j$ such that $x_i \equiv x_j \mod p$ , that is $|x_i - x_j| = kp$ for some integer $k$, then $\gcd(x_i- x_j, n) \neq 1$.

Here we don't need memory to store $x_i$, we can use Floyd cycle detection algorithm. 

```python
from math import gcd
from random import randint, random

def factor(n: int, c: int) -> int:
	# 4 always fails
	if n == 4:
		return 2
	# prime always fails
	elif isprime(n):
		return 1
	
	x, y = 2, 2
	d = 1
	
	def g(x):
		return (x * x % n + c) % n
	
	while d == 1:
		x = g(x)
		y = g(g(y))
		d = gcd(abs(x - y), n)
	if d == n:
		return factor(n, randint(1, n-1))
	else:
		return d
```

# Discrete Logarithm
