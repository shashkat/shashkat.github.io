---
title: 'Two ways to think about matrix-vector multiplication'
date: 2025-07-10
permalink: posts/matrix-multiplication/
tags:
  - Visualization
  - Linear Algebra
---

Useful post about how one can think about matrix-vector multiplication in two different ways, which can be useful in different contexts.

Lets take an example of a matrix-vector multiplication:
$$\textbf{A}\vec{b} = \vec{c}$$. \\
Assume the dimensions of $$\textbf{A}$$ to be $$(3,2)$$, $$\vec{b}$$ to be $$(2,1)$$, $$\vec{c}$$ to be $$(3,1)$$. Hence, we have:  

$$
\begin{pmatrix}
  p & q \\
  r & s \\
  t & u
\end{pmatrix}
\begin{pmatrix}
  v \\
  w
\end{pmatrix}
=
\begin{pmatrix}
  x \\
  y \\
  z
\end{pmatrix}
$$

The more common way to think of the resultant vector is as a collection of the dot products of each row of $$\textbf{A}$$ with the only column in $$\vec{b}$$:

$$
\begin{pmatrix}
  x \\
  y \\
  z
\end{pmatrix}
=
\begin{pmatrix}
  [p,q] \cdot [v,w] \\
  [r,s] \cdot [v,w] \\
  [t,u] \cdot [v,w]
\end{pmatrix}
=
\begin{pmatrix}
  p \cdot v + q \cdot w \\
  r \cdot v + s \cdot w \\
  t \cdot v + u \cdot w 
\end{pmatrix}
$$

The other way is to think of the resultant vector as a sum of two vectors, which are a weighted sum of the columns of $$\textbf{A}$$, according to the values in $$\vec{b}$$.

$$
\begin{pmatrix}
  x \\
  y \\
  z
\end{pmatrix}
=
v \cdot
\begin{pmatrix}
  p \\
  r \\
  t
\end{pmatrix}
+
w \cdot
\begin{pmatrix}
  q \\
  s \\
  u
\end{pmatrix}
=
\begin{pmatrix}
  p \cdot v + q \cdot w \\
  r \cdot v + s \cdot w \\
  t \cdot v + u \cdot w 
\end{pmatrix}
$$

In the form of an animation, the methods would look like following:


<!-- 
$$\textbf{A}$$
$$\vec{b}$$
$$\vec{c}$$
For the following matrix vector multiplication, we have:

Suppose we want to multiply a matrix $$\textbf{A}$$ with vector $$\vec{b}$$ and get the result $$\vec{c}$$: $$\textbf{A}\vec{b} = \vec{c}$$. 

$$
\begin{pmatrix}
  a & b & c \\
  d & e & f \\
  g & h & i
\end{pmatrix}
$$ -->

## References
