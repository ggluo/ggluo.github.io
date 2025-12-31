---
layout: post
title: Pseudoinverse Derivation
date: 2025-11-06 14:53 +0100
permalink: /blog/pseudoinverse
tag:
    - linear algebra
    - coding
use_math: true
---

The concept of a pseudoinverse arises because we often encounter linear systems $$Ax = b$$ that don't have a unique, exact solution. The least-squares framework provides a way to find the "best" possible solution.
This derivation is typically split into two main cases, both of which fall under the umbrella of "least-squares."

---

#### 1. The overdetermined case (tall matrix, $$m > n$$)

This is the classic least-squares problem. We have more equations ($$m$$) than unknowns ($$n$$), and the system is likely inconsistent (meaning no $$x$$ perfectly satisfies $$Ax = b$$).

**The Goal:** Find the vector $$x$$ that minimizes the squared error, or the L2-norm of the residual vector $$r = Ax - b$$.
\begin{equation}
x^* = \arg \min_{x} \|Ax - b\|^2
\end{equation}

Let's define the squared error $$E(x)$$ as a function of $$x$$. Using the dot product, the squared norm is $$r^T r$$:
\begin{equation}
E(x) = (Ax - b)^T (Ax - b)
\end{equation}
We expand the terms:

$$
\begin{aligned}
E(x)&= (x^T A^T - b^T) (Ax - b) \\
E(x)&= x^T A^T A x - x^T A^T b - b^T A x + b^T b
\end{aligned}
$$

Notice that $$x^T A^T b$$ and $$b^T A x$$ are both scalars. In fact, they are transposes of each other, and since the transpose of a scalar is just the scalar itself, they are equal: $$(b^T A x)^T = x^T A^T b$$.
So, we can combine them:
\begin{align}
E(x) = x^T A^T A x - 2 x^T A^T b + b^T b
\end{align}

To find the $$x$$ that minimizes this error, we take the gradient of $$E(x)$$ with respect to $$x$$ and set it to zero.
Using the matrix calculus rules:
* $$\nabla_x (x^T B x) = (B + B^T) x$$ (which is $$2Bx$$ if $$B$$ is symmetric)
* $$\nabla_x (c^T x) = c$$ (or $$\nabla_x (x^T c) = c$$)

The matrix $$A^T A$$ is symmetric. Applying these rules, we get:

\begin{equation}
\nabla_x E(x) = 2(A^T A) x - 2(A^T b) + 0
\end{equation}

Set the gradient to zero to find the minimum:
$$
\begin{align}
    2(A^T A) x - 2(A^T b) =& 0 \\
(A^T A) x =& A^T b
\end{align}
$$

This final equation, $$A^T A x = A^T b$$, is known as the **normal equation**.
To solve for $$x$$, we can multiply by the inverse of $$(A^T A)$$ and get
$$x = (A^T A)^{-1} A^T b$$.
This step requires a critical assumption: **$$(A^T A)$$ must be invertible**. This is true if and only if $$A$$ has **linearly independent columns** (i.e., $$A$$ has full column rank).

We are looking for a matrix $$A^+$$ (the pseudoinverse) such that $$x = A^+ b$$. By direct comparison, we find the **left pseudoinverse**:

\begin{equation}
A^+ = (A^T A)^{-1} A^T
\end{equation}

It's called the "left" inverse because if you multiply $$A$$ on the left by $$A^+$$, you get the identity matrix:
$$A^+ A = \left[ (A^T A)^{-1} A^T \right] A = (A^T A)^{-1} (A^T A) = I$$.

---

#### 2. The underdetermined case (wide matrix, $$n > m$$)

This is a different kind of "least-squares" problem. We have fewer equations ($$m$$) than unknowns ($$n$$), so the system $Ax = b$ has infinitely many solutions.

**The Goal:** Of all the possible solutions, find the *unique* solution $x$ that has the **minimum norm** ($$\min \|x\|^2$$). This is the "least-squares" solution in the sense that it's the "smallest" solution vector.
We want to solve the constrained optimization problem:

\begin{equation}
\min \|x\|^2 \quad \text{subject to} \quad Ax = b
\end{equation}

We can solve this using the method of Lagrange multipliers. The Lagrangian function $$L$$ is:

\begin{equation}
L(x, \lambda) = x^T x + \lambda^T (Ax - b)
\end{equation}

Here, $$x^T x$$ is $$\|x\|^2$$ and $$\lambda$$ is a vector of Lagrange multipliers.
We find the minimum by setting the gradients with respect to both $$x$$ and $$\lambda$$ to zero.

**Gradient w.r.t. $$x$$:**

$$
\begin{aligned}
\nabla_x L &= 2x + A^T \lambda = 0\\
x &= -\frac{1}{2} A^T \lambda
\end{aligned}
$$


**Gradient w.r.t. $$\lambda$$:**

$$
\begin{aligned}
\nabla_\lambda L &= Ax - b = 0\\
Ax &= b
\end{aligned}
$$

Now we combine these two results. Substitute the expression for $$x$$ into the constraint equation:

$$
\begin{aligned}
A \left( -\frac{1}{2} A^T \lambda \right) &= b\\
-\frac{1}{2} (A A^T) \lambda &= b
\end{aligned}
$$

Now, solve for the multiplier vector:
$$\lambda = -2 (A A^T)^{-1} b$$

This step requires the assumption that **$$(A A^T)$$ is invertible**, which is true if and only if $A$ has **linearly independent rows** (i.e., $$A$$ has full row rank).

---

The full **Moore-Penrose pseudoinverse** (often derived using Singular Value Decomposition, or SVD) generalizes both of these cases. It also works for matrices that are *rank-deficient* (i.e., neither full column nor full row rank), which is when $$(A^T A)$$ or $$(A A^T)$$ would be singular (non-invertible).

However, the "least-squares" perspective naturally gives rise to these two fundamental forms:

| Case | Matrix Shape | Problem | Solution $$x = A^+ b$$ | Pseudoinverse $$A^+$$ |
| :--- | :--- | :--- | :--- | :--- |
| **Overdetermined** | Tall ($$m > n$$), full column rank | $$\min \|Ax - b\|^2$$ | Least-Squares Solution | $$A^+ = (A^T A)^{-1} A^T$$ |
| **Underdetermined** | Wide ($$n > m$$), full row rank | $$\min \|x\|^2$$ s.t. $$Ax=b$$ | Minimum Norm Solution | $$A^+ = A^T (A A^T)^{-1}$$ |

---

#### 3. Pseudoinverse using singular value decomposition

Here is the derivation of the pseudoinverse using the Singular Value Decomposition (SVD). This is the most general and powerful approach, as it works for **all** matrices, including those that are rank-deficient (where the previous least-squares derivations would fail).

First, any $$m \times n$$ matrix $$A$$ can be decomposed using SVD as:

\begin{equation}
A = U \Sigma V^T
\end{equation}

Where:
* **$$U$$**: An $$m \times m$$ orthogonal matrix ($$U^T U = I_m$$). Its columns are the *left singular vectors*.
* **$$V$$**: An $$n \times n$$ orthogonal matrix ($$V^T V = I_n$$). Its columns are the *right singular vectors*.
* **$$\Sigma$$**: An $$m \times n$$ diagonal matrix containing the singular values $$\sigma_1, \sigma_2, ...$$ in decreasing order. These values are non-negative.

If $$A$$ has rank $$r$$, then $$r$$ of the singular values are positive, and the rest are zero.


The key to the SVD approach is defining the pseudoinverse of the simple diagonal matrix $$\Sigma$$.

Let's say $$\Sigma$$ (an $$m \times n$$ matrix) looks like this:

$$\Sigma = \begin{bmatrix} \sigma_1 & 0 & \dots & 0 \\ 0 & \sigma_2 & \dots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \dots & \sigma_n \\ \vdots & \vdots & \dots & \vdots \\ 0 & 0 & \dots & 0 \end{bmatrix}$$

To find its pseudoinverse $$\Sigma^+$$, we do two things:
1.  **Transpose** its dimensions to get an $$n \times m$$ matrix.
2.  **Reciprocate** all the **non-zero** singular values, leaving the zeros as zeros.

If $$\Sigma_{ii} = \sigma_i > 0$$, then $$(\Sigma^+)_{ii} = 1/\sigma_i$$.
If $$\Sigma_{ii} = 0$$, then $$(\Sigma^+)_{ii} = 0$$.

**Example:**

If

$$\Sigma = \begin{bmatrix} \sigma_1 & 0 & 0 \\ 0 & \sigma_2 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0 \end{bmatrix}$$

then

$$\Sigma^+ = \begin{bmatrix} 1/\sigma_1 & 0 & 0 & 0 \\ 0 & 1/\sigma_2 & 0 & 0 \\ 0 & 0 & 0 & 0 \end{bmatrix}$$


With $$\Sigma^+$$ defined, the pseudoinverse of $$A$$ (called the **Moore-Penrose Pseudoinverse**) is simply:

\begin{equation}
A^+ = V \Sigma^+ U^T
\end{equation}

This definition elegantly handles all cases: overdetermined, underdetermined, and rank-deficient.
Now, let's show that this SVD definition is consistent with the two cases we derived earlier.

##### Case 1: overdetermined (tall matrix, $$m > n$$, full column rank $$r=n$$)

* Our previous formula was $$A^+ = (A^T A)^{-1} A^T$$.
* Let's plug the SVD into this formula and see if we get $$V \Sigma^+ U^T$$.

1.  **$$A^T A$$**:
    $$A^T A = (U \Sigma V^T)^T (U \Sigma V^T) = (V \Sigma^T U^T) (U \Sigma V^T)$$
    Since $$U^T U = I$$, this simplifies to:
    $$A^T A = V (\Sigma^T \Sigma) V^T$$

2.  **$$(A^T A)^{-1}$$**:
    $$(A^T A)^{-1} = (V (\Sigma^T \Sigma) V^T)^{-1}$$
    Using the rule $$(XYZ)^{-1} = Z^{-1} Y^{-1} X^{-1}$$ and knowing $$V^{-1} = V^T$$:
    $$(A^T A)^{-1} = (V^T)^{-1} (\Sigma^T \Sigma)^{-1} V^{-1} = V (\Sigma^T \Sigma)^{-1} V^T$$

3.  **$$A^+ = (A^T A)^{-1} A^T$$**:
    $$A^+ = \left[ V (\Sigma^T \Sigma)^{-1} V^T \right] (U \Sigma V^T)^T$$
    $$A^+ = \left[ V (\Sigma^T \Sigma)^{-1} V^T \right] (V \Sigma^T U^T)$$
    Since $$V^T V = I$$, this simplifies to:
    $$A^+ = V (\Sigma^T \Sigma)^{-1} \Sigma^T U^T$$

4.  analyze the $$\Sigma$$ part:
    * $$\Sigma$$ is $$m \times n$$. Let's say $$\Sigma = \begin{bmatrix} D_n \\ 0 \end{bmatrix}$$, where $$D_n$$ is an $$n \times n$$ diagonal matrix of $$\sigma_i$$.
    * $$\Sigma^T = \begin{bmatrix} D_n & 0 \end{bmatrix}$$.
    * $$\Sigma^T \Sigma = \begin{bmatrix} D_n & 0 \end{bmatrix} \begin{bmatrix} D_n \\ 0 \end{bmatrix} = D_n^2$$. (This is an $$n \times n$$ invertible matrix).
    * $$(\Sigma^T \Sigma)^{-1} = (D_n^2)^{-1} = D_n^{-2}$$.
    * Now, $$(\Sigma^T \Sigma)^{-1} \Sigma^T = D_n^{-2} \begin{bmatrix} D_n & 0 \end{bmatrix} = \begin{bmatrix} D_n^{-1} & 0 \end{bmatrix}$$.
    * This resulting matrix, $$\begin{bmatrix} D_n^{-1} & 0 \end{bmatrix}$$, is **exactly the SVD definition of $$\Sigma^+$$**.

5.  Conclusion:
    $$A^+ = V (\Sigma^+) U^T$$. The formulas match.

##### Case 2: underdetermined (wide matrix, $$n > m$$, full row rank $$r=m$$)

* Our previous formula was $$A^+ = A^T (A A^T)^{-1}$$.
* A similar substitution (which I'll skip the full algebra for) shows:
    $$A^+ = (V \Sigma^T U^T) \left[ (U \Sigma V^T)(V \Sigma^T U^T) \right]^{-1}$$
    $$A^+ = (V \Sigma^T U^T) \left[ U (\Sigma \Sigma^T) U^T \right]^{-1}$$
    $$A^+ = (V \Sigma^T U^T) \left[ U (\Sigma \Sigma^T)^{-1} U^T \right]$$
    $$A^+ = V \Sigma^T (\Sigma \Sigma^T)^{-1} U^T$$

* Again, if you analyze the $$\Sigma$$ part ($$\Sigma^T (\Sigma \Sigma^T)^{-1}$$), you'll find it simplifies to $$\Sigma^+$$.
* **Conclusion**: $$A^+ = V \Sigma^+ U^T$$. The formulas match again.

##### Why the SVD Method is Better

The SVD derivation $$A^+ = V \Sigma^+ U^T$$ is superior because it **never fails**.

* In the rank-deficient case, $$r < \min(m, n)$$.
* This means both $$(A^T A)$$ and $$(A A^T)$$ will be singular (non-invertible) because their $$\Sigma$$ components ($$D_r^2$$) will contain zeros. Our first derivation using $$(...)^{-1}$$ would fail.
* However, the SVD definition of $$\Sigma^+$$ **gracefully handles this**. It only reciprocates the $$\sigma_i$$ that are *non-zero* and leaves the zeros as zeros.

This $$A^+ = V \Sigma^+ U^T$$ always gives the unique **minimum-norm least-squares solution**:
1.  **Least-Squares**: It finds the $$x$$ (or $$x$$'s) that minimizes the error $$\|Ax - b\|^2$$.
2.  **Minimum-Norm**: If there are multiple solutions (as in the underdetermined or rank-deficient case), it picks the *one and only one* solution $$x$$ that also has the smallest length $$\|x\|^2$$.

