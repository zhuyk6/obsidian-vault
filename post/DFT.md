# Definition
Given $(x_0, x_1, \ldots, x_{N-1})$ , the **discrete Fourier transformation** is:
$$y_j = {1\over \sqrt{N}}\sum_{k=0}^{N-1} x_k e^{-2\pi i jk / N}$$
The inverse DFT is:
$$x_k = {1\over \sqrt{N}} \sum_{j=0}^{N-1} y_j e^{2\pi ijk / N}$$
# Computation
[[FFT]]
