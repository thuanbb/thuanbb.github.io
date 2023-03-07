---
title: Matrix Decomposition 
updated: 2023-03-07 22:00
---



* TOC
{:toc}

## Eigen Decomposition

$$ \mathbf{A} \in \mathbb{R}^{n \times n}$$, $$\mathbf{A}$$ is diagonalizable, then 

$$ \mathbf{A}= \mathbf{Q}\mathbf{\Lambda}\mathbf{Q}^{-1}$$, 

$$ \mathbf{Q} = \begin{bmatrix}
| & | & ...  & |\\
\mathbf{v}_{1} & \mathbf{v}_{2}  & ... & \mathbf{v}_{n}  \\ 
| & | & ... & |
\end{bmatrix} $$, $$ \mathbf{\Lambda} = \begin{bmatrix}
\lambda_{1} & 0 & 0  & 0\\
0 & \lambda_{2}  & ... & 0\\ 
0 & 0 & ... &  \lambda_{n}
\end{bmatrix} $$ 

$$ \{\lambda_{i}\}^{n}_{i=0} $$ are eigenvalues with corresponding eigenvectors $$\{\mathbf{v}_{i}\}^{n}_{i}$$

This means $$ \mathbf{A}\mathbf{v}_{i} = \lambda_{i}\mathbf{v}_{i} $$ (eigen values equation) 


If $$\mathbf{A}$$ is symmetric ($$\mathbf{A}=\mathbf{A}^{T}$$), then we can always write $$ \mathbf{A}= \mathbf{Q}\mathbf{\Lambda}\mathbf{Q}^{-1}$$, 

### Application
Eigen Decomposition is very useful to simplify and reduce matrix product computation with $$\mathbf{A}$$. 

**Problem 1.**  Let $$\mathbf{A} \in \mathbb{R}^{n \times n}$$ be a diagonalizable matrix with eigenvalues $$ \{\lambda_{i}\}^{n}_{i=0} $$ 
	+ Compute $$\mathbf{A}^{k}$$

