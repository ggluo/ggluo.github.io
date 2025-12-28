---
layout: post
title: data-consistency
permalink: /blog/data-consistency
date: 2025-11-06 15:40 +0100
published: false
---
You've hit on a key difference between two common approaches to data consistency. The paper uses the pseudoinverse ($A^+$), while tools like BART often use the adjoint ($A^*$), and they are used for fundamentally different reasons.

* **$A^*$ (Adjoint) is for Gradient Descent:** Using the adjoint $A^*$ (as in $x_t \leftarrow \tilde{x}_t - \alpha A^*(A\tilde{x}_t - \tilde{y}_t)$) is a **gradient descent step**. It takes a *small step* in the direction that *reduces* the k-space error. This is common in iterative optimization algorithms that slowly converge to a solution.

* **$A^+$ (Pseudoinverse) is for Projection:** The paper's **Equation (24)** is not a gradient step; it's a **one-shot orthogonal projection**. This operation takes the current image estimate $\tilde{x}_t$ and finds the *closest possible* image $x_t$ that is in **perfect agreement** with the measurements $\tilde{y}_t$.

### Why This Paper Uses $A^+$

The method described here, which the authors relate to "range-null decomposition", relies on this direct projection. The goal of this specific step is not just to "reduce" the error, but to **enforce** perfect data consistency on the noiseless estimate.

As the paper itself notes, this hard projection is only suitable for **clean (noiseless) measurements**. If the data $y_0$ were noisy, this step would perfectly inject all that noise into the final image. That is why they use a different "Plug-and-Play (PnP)" method (Algorithm 2) for the noisy data scenario.

This formula is the direct solution to a constrained optimization problem. Here is the derivation.

### 1. 🎯 The Optimization Problem

The goal is to **enforce data consistency**. We have a current image estimate, $\tilde{x}_t$, which is good but doesn't perfectly match our measurements, $\tilde{y}_t$.

We want to find a new image, $x_t$, that solves the following problem:

> "Find the image $x_t$ that is **as close as possible** to $\tilde{x}_t$ while **perfectly satisfying** the measurement constraint $A(x_t) = \tilde{y}_t$."

Mathematically, this is written as:

$$
\text{minimize} \quad \frac{1}{2} ||x - \tilde{x}_t||_2^2
$$
$$
\text{subject to} \quad A(x) = \tilde{y}_t
$$

(We use $\frac{1}{2}||\cdot||_2^2$ because it has the same minimum as $||\cdot||_2$ and its derivative is simpler.)

---

### 2. 📝 The Method: Lagrange Multipliers

We can solve this constrained problem by introducing a Lagrange multiplier, $\lambda$, and forming the Lagrangian function, $\mathcal{L}$:

$$
\mathcal{L}(x, \lambda) = \underbrace{\frac{1}{2} ||x - \tilde{x}_t||_2^2}_{\text{Proximity to } \tilde{x}_t} + \underbrace{\lambda^T (A(x) - \tilde{y}_t)}_{\text{Constraint}}
$$

To find the minimum, we must find the point where the gradients of $\mathcal{L}$ with respect to both $x$ and $\lambda$ are zero.

---

### 3. ⚙️ Solving the System

#### Step 1: Find the gradient with respect to $x$

We take the partial derivative of $\mathcal{L}$ with respect to $x$ and set it to zero.

> (Note: The derivative of $\frac{1}{2} ||x - c||_2^2$ with respect to $x$ is $(x - c)$. The derivative of $\lambda^T A(x)$ with respect to $x$ is $A^T\lambda$.)

$$
\frac{\partial \mathcal{L}}{\partial x} = (x - \tilde{x}_t) + A^T\lambda = 0
$$

Solving for $x$, we get:
$$
x = \tilde{x}_t - A^T\lambda
$$
This tells us the *form* of our solution $x_t$, but we still need to find $\lambda$.

#### Step 2: Use the constraint to find $\lambda$

We know our solution $x_t$ *must* satisfy the constraint $A(x_t) = \tilde{y}_t$. So, we plug our formula for $x$ from Step 1 into the constraint:

$$
A(\tilde{x}_t - A^T\lambda) = \tilde{y}_t
$$

Now we distribute $A$:
$$
A(\tilde{x}_t) - A(A^T\lambda) = \tilde{y}_t
$$

And rearrange to solve for $\lambda$:
$$
A(\tilde{x}_t) - \tilde{y}_t = A A^T \lambda
$$

$$
\lambda = (A A^T)^+ (A(\tilde{x}_t) - \tilde{y}_t)
$$
(We use the pseudoinverse $(A A^T)^+$ in case $A A^T$ is not invertible).

#### Step 3: Put it all together

Now we substitute this expression for $\lambda$ back into our solution for $x$ from Step 1:

$$
x_t = \tilde{x}_t - A^T \left[ (A A^T)^+ (A(\tilde{x}_t) - \tilde{y}_t) \right]
$$

This looks complicated, but we can simplify it using a standard identity of the Moore-Penrose pseudoinverse:
> **Pseudoinverse Identity:** $A^T(A A^T)^+ = A^+$

Applying this identity, the $A^T (A A^T)^+$ term simplifies to just $A^+$:

$$
x_t = \tilde{x}_t - A^+ (A(\tilde{x}_t) - \tilde{y}_t)
$$

This is **Equation (24)**. It is the exact, analytical solution to the problem of finding the closest image to $\tilde{x}_t$ that perfectly matches the measurements $\tilde{y}_t$.