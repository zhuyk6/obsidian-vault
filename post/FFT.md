# Fast Fourier Transform
Let $w_N = e^{2\pi i / N}$. 
$w_N$ has a property :
$$w_{kN}^{kl} = e^{2\pi i kl / kN} = e^{2\pi i l / N} = w_N^l$$
We can divide $x_k$ into even and odd parts
$$
\begin{align}
y_j 
& = {1\over \sqrt{N}} \sum_{k = 0}^{N-1} x_k w_N^{-jk} \\
& = {1\over \sqrt{N}} \sum_{m = 0}^{N/2-1} x_{2m} w_N^{-2jm} + {1\over \sqrt{N}} \sum_{m = 0}^{N/2-1} x_{2m+1} w_N^{-2jm-j} \\
& = {1\over \sqrt{2}}\left({1\over \sqrt{N/2}} \sum_{m = 0}^{N/2-1} x_{2m} w_{N/2}^{-jm}\right) + {1\over \sqrt{2}}w_N^{-j} \left({1\over \sqrt{N/2}}\sum_{m = 0}^{N/2-1} x_{2m+1} w_{N/2}^{-jm}\right) \\
\end{align}
$$
Let
$$
\begin{align}
E_j & = {1\over \sqrt{N/2}} \sum_{m = 0}^{N/2-1} x_{2m} w_{N/2}^{-jm} & = FFT(x_0, x_2, \ldots)(j)\\
O_j & = {1\over \sqrt{N/2}} \sum_{m = 0}^{N/2-1} x_{2m+1} w_{N/2}^{-jm} & = FFT(x_1, x_3, \ldots)(j)
\end{align}
$$
When $0 \leq j < N/2$ 
$$
\begin{align}
y_j = {1\over \sqrt{2}} \left( E_j + w_N^{-j} O_j \right) \\
y_{j+N/2} = {1\over \sqrt{2}} \left( E_j - w_N^{-j} O_j \right)
\end{align}
$$
# Inverse Fast Fourier Transform
$$
\begin{align}
x_j 
& = {1\over \sqrt{N}} \sum_{k = 0}^{N-1} y_k w_N^{jk} \\
& = {1\over \sqrt{N}} \sum_{m = 0}^{N/2-1} y_{2m} w_N^{2jm} + {1\over \sqrt{N}} \sum_{m = 0}^{N/2-1} y_{2m+1} w_N^{2jm+j} \\
& = {1\over \sqrt{2}}\left({1\over \sqrt{N/2}} \sum_{m = 0}^{N/2-1} y_{2m} w_{N/2}^{jm}\right) + {1\over \sqrt{2}}w_N^{j} \left({1\over \sqrt{N/2}}\sum_{m = 0}^{N/2-1} y_{2m+1} w_{N/2}^{jm}\right) \\
\end{align}
$$
Therefore
$$
\begin{align}
x_j = {1\over \sqrt{2}} \left( E_j + w_N^{j} O_j \right) \\
x_{j+N/2} = {1\over \sqrt{2}} \left( E_j - w_N^{j} O_j \right)
\end{align}
$$
> The difference between FFT and IFFT is just setting $w_N$ to $w_N^*$ .

# Code
```python TI:"FFT and DFT"
from typing import *
import numpy as np

def fft(x: List[complex], inverse):
    n = len(x)
    if n == 1:
        # y[0] = x[0]
        return x
    e = fft([x[i * 2] for i in range(n // 2)], inverse)
    o = fft([x[i * 2 + 1] for i in range(n // 2)], inverse)
    # w = complex(np.cos(2 * np.pi / n), np.sin(2 * np.pi / n))
    w = np.exp(-2 * np.pi * 1j / n)
    if inverse:
        w = w.conj()
    y = [e[i] + (w ** i) * o[i] for i in range(n // 2)]
    y += [e[i] - (w ** i) * o[i] for i in range(n // 2)]
    return list(map(lambda y: y / np.sqrt(2), y))

def dft(x: List[complex], inverse):
    n = len(x)
    y = [0 for _ in range(n)]
    w = np.exp(-2 * np.pi * 1j / n)
    if inverse:
        w = w.conj()
    for j in range(n):
        for k in range(n):
            y[j] += x[k] * (w ** (j * k))
        y[j] /= np.sqrt(n)
    return y
```
