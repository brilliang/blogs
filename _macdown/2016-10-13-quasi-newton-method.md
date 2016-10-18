# <del>quasi-Newton method</del>

This article records my reading notes of <统计学习方法>, but I found many mistakes after finished reading something else. So please ignore this one and read my next blog.


---

I am trying to understand all the detail in spark LogisticRegression, which applys different kinds of [quasi-Newton](https://en.wikipedia.org/wiki/Quasi-Newton_method) methods for optimisaztion. In this article I will record what I have learnt about it.

## [Newton method](https://en.wikipedia.org/wiki/Newton%27s_method_in_optimization)
All stories come from the [Taylor Expansion ](https://en.wikipedia.org/wiki/Taylor%27s_theorem). Let's assume $f: R^n \rightarrow R $, is convex and 二阶可导

$ f(x) = f(x_k) + \nabla f(x_k)^T (x - x_k) + \frac{1}{2}(x - x_k)^T H(x_k) (x - x_k) $ 
<div align="right">(1)</div>

H is the [hessian matrix](https://en.wikipedia.org/wiki/Hessian_matrix) of function $ f $:

$ H(x) = [\frac{\partial ^2f}{\partial x_i \partial y_j}]_{m*n} $

**SYMBOL** $ f_k = f(x_k); g_k = \nabla f(x_k); H_k = H(x_k);$

for given $ x_k $, all the 3 values are constant.

---

only if $ \nabla f(x)  = 0$, the f(x) is the mininum value.

taking the derivative of (1), we can get:

$ \nabla f(x) = g_k + H_k (x-x_k) = 0$
<div align="right">(2)</div>
after expanding it, we can get 

$ x_{k+1} =  x_k - H_k^{-1}* g_k$
<div align="right">(3)</div>

> In my understanding, it is just applying [Newton Method](https://en.wikipedia.org/wiki/Newton%27s_method) to solve the equation g(x) = 0

## gradient descent
For comparison, I list the method of gradient descent here:

$ x_{k+1} = x_k - \lambda g_k$

the theory under it is that the negative gradient shows the direction where the value of f descents fastest.

## if hessian matrix is [positive definite](https://en.wikipedia.org/wiki/Positive-definite_matrix)
Compare Newton method to gradient descent method, it is easy to see that we are still using the the $ g_x $ as the optimisation direction. The point is, if the hessian matrix is positive definite, then this direction is always points to descent.

if we substitue x with (3), into (1), and ignore second order item, we can get:

$ f(x) = f_k - g_k^T * H_k^{-1} * g_k$

then we get the result: if the hessian matrix is positive definite, which means $  g_k^T * H_k^{-1} * g_k > 0 $, then f(x) will go the descent direction.

## Secant Equation
From (2), it is easy to get $ g_{k+1} - g_k = H_k * (x_{k+1} - x_k)$

**SYMBOL**: $ y_k=g_{k+1} - g_k;  \delta_k= x_{k+1} - x_k $

so the quasi-Newton condition is defined as:

$ y_k = \delta_k H_k$
<div align="right">(4)</div>
or,  $ H_k^{-1}  y_k = \delta_k$
<div align="right">(5)</div>
___
# quasi-Newton method
$ H^{-1}$ is not easy to get. So we are trying to find a matrix to subsitute B and is easier to calculate. In every iteration of calculation, it is subject to the quasi-Newton condition.

## DFP
$ G_{k+1} = G_k + P_k + Q_k$

We are going to choose P and Q to make both $ G_{k+1} $ and $ G_k $ subject to the quasi-Newton condition. Tricky to set $ P_k y_k = \delta _k;  Q_k y_k = -G_k y_k $, we can get 

$ G_{k+1} y_k = G_k y_k + P_k y_k + Q_k y_k = \delta _k$

so we can get one sulotion:

$ P_k = \frac{\delta _k \delta_k ^T}{\delta _k ^T y_k}; Q_k = - \frac{G_k y_k y_k^T G_k}{y_k^T G_k y_k}$

if the initial G is positive definite, then all G in iteration is positive definite.

## BFGS
if we consider the quasi-Newton condition in the persipective of hessian matrix, rather than its reverse:$ H_k \delta_k = y_k $

We can try to find a matrix B to approximate H. Using the same method as DFP, $ B_{k+1} = B_k + P_k + Q_k $

Tricky to set $ P_k \delta _k = y_k ;  Q_k \delta _k = -B_k \delta _k $, we can get

$ B_{k+1} \delta_k = B_k \delta_k+ P_k \delta_k+ Q_k \delta_k = y_k$

solution of P and Q is 
$ P_k = \frac{y_k y_k^T}{y_k^T \delta_k}, Q_k=-\frac{B_k \delta_k \delta_k^T B_k}{\delta _k^T  B_k \delta _k} $

if the initial G is positive definite, then all G in iteration is positive definite.

### Algorithm:
1. initial $x_0$, choose a positive definite matrix $B_0, k=0$
2. calculate $g_k = \nabla f(x_k)$, if $||g_k|| \leq \epsilon$, return $x = x_k$
3. calculate $p_k$ from  $B_k p_k = g_k$
4. linear search for step length $\lambda$:  $ argmin _ \lambda f(x_k + \lambda p_k) $
5. update $x_{k+1} = x_k + \lambda p_k, k = k + 1$, goto step 2.

## Broyden Algorithm
by applying Sherman-Morrison formula twice, we can get another approximation of $H^{-1}$.
we can linear combination the tow approximations of $H^{-1}$:

$G_{k+1} = \alpha G^{DFP} + (1 - \alpha) G^{BFGS}$
 




