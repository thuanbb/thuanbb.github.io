---
title: Matrix Decomposition 
updated: 2023-03-07 22:00
---



* TOC
{:toc}

## Eigen Decomposition

$$ \mathbf{A} \in \mathbb{R}^{n \times n}$$, $$\mathbf{A}$$ is diagonalizable, then 

$$ \mathbf{A}= \mathbf{Q}\mathbf{\Lambda}\mathbf{Q}^{-1}$$, $$ \mathbf{Q} = \begin{bmatrix}
| & | & ...  & |\\
\mathbf{v}_{1} & \mathbf{v}_{2}  & ... & \mathbf{v}_{n}  \\ 
| & | & ... & |
\end{bmatrix} $$, $$ \mathbf{\Lambda} = \begin{bmatrix}
\lambda_{1} & 0 & 0  & 0\\
0 & \lambda_{2}  & ... & 0\\ 
0| 0 | 0 ... & \lambda_{n}  |
\end{bmatrix} $$ 