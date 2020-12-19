# Basic Loss

## Focal Loss
$$
Loss = \begin{cases}
\alpha(1-p)^\gamma (-log(p)) &if\ gt==1\\
(1-\alpha)p^\gamma (-log(1-p)) &if\ gt==0\\
best:\alpha=0.25, \gamma=2
\end{cases}
$$

