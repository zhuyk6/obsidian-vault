# Pure state
For a pure two-qubits state $|\psi\rangle$ , the entanglement is defined as
$$
E\left( |\psi\rangle \right) \equiv -tr(\rho_A \log_2 \rho_A) = -tr(\rho_B \log_2 \rho_B)
$$
Here is a formation:
$$
E(|\psi\rangle) = H\left(\frac{1 + \sqrt{1 - C^2}}{2}\right)
$$
where
$$
\begin{align}
H(x) & \equiv -x\log_2 x - (1-x)\log_2 (1-x) \\
C & \equiv \left|\sum_i \alpha_i^2\right| \\
\end{align}
$$
where $\alpha_i$ are amplitudes of **magic basis**. That is $|\psi\rangle = \sum_i \alpha_i |e_i\rangle$ . The magic basis is defined as
$$
\begin{align}
|e_1\rangle & \equiv \frac{|00\rangle + |11\rangle}{\sqrt{2}} \\
|e_2\rangle & \equiv i\frac{|00\rangle - |11\rangle}{\sqrt{2}} \\
|e_3\rangle & \equiv i\frac{|01\rangle + |10\rangle}{\sqrt{2}} \\
|e_4\rangle & \equiv \frac{|01\rangle - |10\rangle}{\sqrt{2}} \\
\end{align}
$$
## Proof
Consider RHS
$$
\begin{align}
RHS & = H\left(\frac{1 + \sqrt{1-C^2}}{2}\right) \\
& = -\frac{1+\sqrt{1-C^2}}{2} \log_2 \frac{1+\sqrt{1-C^2}}{2} - \frac{1-\sqrt{1-C^2}}{2} \log_2 \frac{1-\sqrt{1-C^2}}{2}
\end{align}
$$

$|\psi\rangle$ is pure and according to Schmidt decomposition
$$
|\psi\rangle = \sum_i \lambda_i |i^A\rangle |i^B\rangle \quad \text{where } \lambda_i \geq 0
$$
Then we can compute
$$
\begin{align}
\rho & = |\psi\rangle \langle \psi| \\
& = \sum_{i,j} \lambda_i \lambda_j |i^A\rangle |i^B\rangle \langle j^A|\langle j^B| \\
\rho_A & = \sum_i \lambda_i^2 |i^A\rangle \langle i^A| \\
E(|\psi\rangle) & = -tr(\rho_A\log_2 \rho_A) \\
& = -tr\left( \sum_i \lambda_i^2 \log_2 \lambda_i^2 |i^A\rangle 
\langle i^A| \right) \\
& = -\sum_i \lambda_i^2 \log_2 \lambda_i^2
\end{align}
$$
LHS equal to RHS iff 
$$
\begin{cases}
\lambda_1^2 = \frac{1 + \sqrt{1 - C^2}}{2} \\
\lambda_2^2 = \frac{1 - \sqrt{1 - C^2}}{2}
\end{cases}
$$
Therefore $\frac{1\pm\sqrt{1-C^2}}{2}$ are eigenvalues of $\rho_A$ and $\rho_B$.
Next we will prove it.

Let 
$$
\begin{align}
|e_{00}\rangle & \equiv \frac{|00\rangle + |11\rangle}{\sqrt{2}} \\
|e_{01}\rangle & \equiv i\frac{|00\rangle - |11\rangle}{\sqrt{2}} \\
|e_{10}\rangle & \equiv i\frac{|01\rangle + |10\rangle}{\sqrt{2}} \\
|e_{11}\rangle & \equiv \frac{|01\rangle - |10\rangle}{\sqrt{2}} \\
\end{align}
$$
we can summarize them as
$$
|e_{xy}\rangle \equiv \frac{i^{x\oplus y}}{\sqrt{2}} \left( |0,x\rangle + (-1)^y |1,\bar{x}\rangle \right)
$$
Then calculate
$$
\begin{align}
\rho & = \left( \sum_{x,y} \alpha_{xy} |e_{xy}\rangle \right) \left( \sum_{a,b} \alpha_{ab}^* \langle e_{xy}|  \right) \\
& = \sum_{x,y,a,b} \frac{i^{x\oplus y}}{\sqrt{2}} \alpha_{xy} \frac{(-i)^{a\oplus b}}{\sqrt{2}} \alpha_{ab}^* \Big[ |0,x\rangle + (-1)^y |1, \bar{x}\rangle \Big] \Big[ \langle 0, a| + (-1)^b \langle 1, \bar{a} \Big] \\
& = \sum_{x,y,a,b} \frac{i^{x\oplus y}}{\sqrt{2}} \alpha_{xy} \frac{(-i)^{a\oplus b}}{\sqrt{2}} \alpha_{ab}^* \Big[ 
  |0,x\rangle\langle 0,a| 
+ (-1)^b |0,x\rangle \langle 1,\bar{a}|
+ (-1)^y |1,\bar{x}\rangle \langle 0,a|
+ (-1)^{y+b} |1,\bar{x}\rangle\langle 1,\bar{a}|
\Big] 
\end{align}
$$

and then
$$
\begin{align}
\rho_B
& = \sum_{x,y,a,b} \frac{i^{x\oplus y}}{\sqrt{2}} \alpha_{xy} \frac{(-i)^{a\oplus b}}{\sqrt{2}} \alpha_{ab}^* 
\Big[ 
  |x\rangle\langle a| 
+ (-1)^{y+b} |\bar{x}\rangle\langle\bar{a}|
\Big] \\
& = \sum_{x,y,a,b} \frac{i^{x\oplus y}}{\sqrt{2}} \alpha_{xy} \frac{(-i)^{a\oplus b}}{\sqrt{2}} \alpha_{ab}^* |x\rangle\langle a|
+ \sum_{x,y,a,b} (-1)^{y+b}\frac{i^{\bar{x}\oplus y}}{\sqrt{2}} \alpha_{\bar{x}y} \frac{(-i)^{\bar{a}\oplus b}}{\sqrt{2}} \alpha_{\bar{a}b}^* |x\rangle\langle a|\\
& = P_1 + P_2
\end{align}
$$
next we will calculate $P_1$ and $P_2$.
$$
\begin{align}
P_1 
& = \sum_{x,a}\sum_y |x\rangle\langle a| \frac{i^{x\oplus y}}{\sqrt{2}} \alpha_{xy} \frac{\alpha_{aa}^* - i \alpha_{a\bar{a}}^*}{\sqrt{2}} \\
& = \sum_{x,a} |x\rangle \langle a| \frac{\alpha_{xx} + i \alpha_{x\bar{x}}} {\sqrt{2}} \frac{\alpha_{aa}^* - i \alpha_{a\bar{a}}^*}{\sqrt{2}} \\
& = \sum_{x,a} |x\rangle \langle a| S_x S_a^*
\end{align}
$$
where
$$
S_x \equiv \frac{\alpha_{xx} + i \alpha_{x\bar{x}}}{\sqrt{2}}
$$
and
$$
\begin{align}
P_2
& = \sum_{x,a} \sum_{y,b}|x\rangle \langle a| (-1)^{y+b} \frac{i^{\bar{x}\oplus y}}{\sqrt{2}} \alpha_{\bar{x}y} \frac{(-i)^{\bar{a}\oplus b}}{\sqrt{2}} \alpha_{\bar{a}b}^* \\
& = \sum_{x,a} \sum_y |x\rangle\langle a| (-1)^y \frac{i^{\bar{x}\oplus y}} {\sqrt{2}} \alpha_{\bar{x}y} \frac{(-1)^a (-i) \alpha_{\bar{a}a}^* + (-1)^{\bar{a}} \alpha_{\bar{a}\bar{a}}^*}{\sqrt{2}} \\
& = \sum_{x,a} \sum_y |x\rangle\langle a| (-1)^y \frac{i^{\bar{x}\oplus y}} {\sqrt{2}} \alpha_{\bar{x}y} (-1)^a \frac{-i \alpha_{\bar{a}a}^* - \alpha_{\bar{a}\bar{a}}^*}{\sqrt{2}} \\
& = \sum_{x,a} |x\rangle\langle a| (-1)^{x+a} \frac{\alpha_{\bar{x}\bar{x}}-i \alpha_{\bar{x}x}}{\sqrt{2}} \left(\frac{\alpha_{\bar{a}\bar{a}}-i\alpha_{\bar{a}a}}{\sqrt{2}}\right)^* \\
& = \sum_{x,a} |x\rangle\langle a| (-1)^{x+a} T_{\bar{x}}T_{\bar{a}}^*
\end{align}
$$
where
$$
T_x \equiv \frac{\alpha_{xx} - i\alpha_{x\bar{x}}}{\sqrt{2}}
$$
Then
$$
\rho_B = \sum_{x,a} |x\rangle\langle a| \Big[ S_xS_a^* + (-1)^{x+a} T_\bar{x} T_\bar{a}^*\Big]
$$
Now we get the matrix form of $\rho_B$ in the computational basis.
$$
\rho_B = \begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
$$
where
$$
\begin{align}
a & = S_0 S_0^* + T_1 T_1^* \\
d & = S_1 S_1 ^* + T_0 T_0^* \\
b & = S_0 S_1^* - T_1 T_0^* \\
c & = S_1 S_0^* - T_0 T_1^*
\end{align}
$$
The determinant of $\rho_B$ is
$$
\begin{align}
\det \rho_B 
& = ad - bc \\
& = |S_0|^2 |T_0|^0 + |S_1|^2 |T_1|^2 + S_0 T_0 S_1^* T_1^* + S_0^* T_0^* S_1 T_1 \\
& = |S_0 T_0|^2 + |S_1 T_1|^2 + 2\Re(S_0T_0 S_1^* T_1^*) \\
& = (S_0T_0 + S_1T_1)(S_0^*T_0^* + S_1^*T_1^*) \\
& = |S_0T_0 + S_1 T_1|^2
\end{align}
$$
It's easy to check
$$
S_xT_x = \frac{\alpha_{xx}+i\alpha_{x\bar{x}}}{\sqrt{2}} \frac{\alpha_{xx} - i\alpha_{x\bar{x}}}{\sqrt{2}} = \frac{\alpha_{xx}^2 + \alpha_{x\bar{x}}^2}{2}
$$
Then
$$
\det \rho_B = \left|\frac{\alpha_{00}^2 + \alpha_{01}^2 + \alpha_{11}^2 + \alpha_{10}^2}{2}\right|^2 = \frac{1}{4} C^2
$$

Now compute eigenvalues of $\rho$.
Consider the characteristic function
$$
|\rho_B - \lambda I| = \begin{vmatrix}
a - \lambda & b \\
c & d - \lambda
\end{vmatrix} = 0
$$
We get
$$
\lambda^2 - (a+d)\lambda + ad - bc = 0
$$
Because $\rho_B$ is a density operator, so $tr(\rho_B) = a+d = 1$, then
$$
\lambda^2 - \lambda + \det \rho_B = 0
$$
the solutions are
$$
\lambda_{1,2} = \frac{1 \pm \sqrt{1 - 4\det \rho_B}}{2} = \frac{1\pm \sqrt{1 - C^2}}{2}
$$
Now we have proved the conclusion that 
$$
E(|\psi\rangle) = H\left(\frac{1 + \sqrt{1-C^2}}{2} \right)
$$

## Why magic basis ?
Here is a question: why magic basis ?
It's easy to see that magic basis is relative to Bell basis.
In fact, we can use Bell basis to calculate and we can easily get the formation
$$
\det \rho_B = \frac{1}{4} |\mu_{00}^2 - \mu_{01}^2 - \mu_{10}^2 + \mu_{11}^2|^2
$$
where $\mu_{i}$ are amplitudes of Bell basis.
Therefore the meaning of magic basis is just making the formation be a better form.

# Mixed state

todo

- [] complement concurrence for mixed state.