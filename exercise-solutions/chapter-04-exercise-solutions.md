1. How does Hadamard Product (element-wise product) differ from Matrix Multiplication?

Hadarmard product performs multiplication between corresponding elements of the two tensors of identical shape (or via broadcasting over singleton dimensions). The output of the Hadamard product is the same as the input tensor. In contrast, matrix multiplication always multiply and therefore combine over the last dimension of the former matrix and the leading dimension of the latter matrix. The result of matrix multiplication is not the same as that of the input.

2. Implement matrix multiplication between a matrix of size `(5, 4)` and a matrix of size `(4, 3)` using for-loop. What is the time complexity of this implementation? Compare the performance of this implementation with NumPy's `np.matmul` function.

```python
import numpy as np

def matmul_for_loop(x, y):
    # assuming x and y are 2 dimensional
    output = []
    for x_row in x: # x row
        output.append([])
        for y_col in np.transpose(y):
            res = 0
            for ii in range(len(x_row)):
                res += x_row[ii] * y_col[ii]
            output[-1].append(res)

    return output
```

Let us denote the shape of `x` as `(M, K)` and shape of `y` as `(K, N)`. Since we are looping over the rows of `x` and columns of `y`, while performing multiplication and summation over the two inner dimensions, the overall time complexity is $O(MNK)$. Performance using `%timeit`:

```python
import numpy as np
x = np.random.rand(5, 4)
y = np.random.rand(4, 3)

%timeit x @ y # 652 ns ± 28.4 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
%timeit matmul_for_loop(x, y) # 15.7 µs ± 292 ns per loop (mean ± std. dev. of 7 runs, 100,000 loops each)
```

3. Suppose we have the following matrices

```python
import numpy as np
import torch
A = np.arange(20).reshape(5, 4)
B = torch.arange(12).reshape(4, 3)
```

What would happen in the operation `A @ B`?

`A @ B` performs the matrix multiplication between the matrix `A` and `B` among the two inner dimensions, resulting a matrix of shape `(5, 3)`.

4. Use `einsum` to implement a tensor multiplication between a tensor of shape `(5, 4, 3)` and a tensor of shape `(3, 4, 2)`, so that the resulting shape of the output is `(5, 4, 4, 2)`. Here, we treat the first dimension of the first tensor as the batch dimension, and the last dimension of the second tensor as the batch dimension as well. That is, we would like to do batch-wise matrix multiplication between pairs of matrices of shape `(4, 3)` and `(3, 4)`.

```python
import numpy as np
x = np.random.rand(5, 4, 3)
y = np.random.rand(3, 4, 2)

z = np.einsum("ijk,klm->ijlm", x, y)
```


5. Implement the same operations as above using `tensordot` instead.

```python
import numpy as np
x = np.random.rand(5, 4, 3)
y = np.random.rand(3, 4, 2)

z = np.tensordot(x, y, axes=[(2), (0)])
```


6. Compare the performance of `faster_cross_corr` with `np.corrcoef` and `pd.DataFrame.corr` to compute pairwise cross-correlation between features. Suppose the features are stored in a matrix of size `num_samples x feature_size` matrix. Plot the time taken for each method as a function of the `feature_size`.

```python
import time
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from tqdm import tqdm
import faster_cross_corr

num_trials = 30
num_samples = 100
time_taken1 = np.zeros((num_trials, num_samples))
time_taken2 = np.zeros((num_trials, num_samples))
time_taken3 = np.zeros((num_trials, num_samples))
sizes = np.logspace(1, 3, num=num_samples, base=10, dtype=int)

for trial in tqdm(range(num_trials)):
    for ii, size in tqdm(enumerate(sizes)):
        x = np.random.randn(1000, size)
        tnow = time.time()
        faster_cross_corr(x, x)
        time_taken1[trial, ii] = time.time() - tnow
        # np.corrcoef
        tnow = time.time()
        np.corrcoef(x, x, rowvar=False)[:size, size:]
        time_taken2[trial, ii] = time.time() - tnow
        # pd.DataFrame.corr
        df = pd.DataFrame(x)
        tnow = time.time()
        df.corr()
        time_taken3[trial, ii] = time.time() - tnow
   
time_taken1 = time_taken1.mean(axis=0)
time_taken2 = time_taken2.mean(axis=0)
time_taken3 = time_taken3.mean(axis=0)


fig, ax = plt.subplots(1, 1)
ax.plot(sizes, time_taken1, label="faster_cross_corr")
ax.plot(sizes, time_taken2, label="np.corrcoef")
ax.plot(sizes, time_taken3, label="df.corr")

ax.legend()
ax.set_xlabel("Size")
ax.set_ylabel("Time (sec)")
ax.set_xscale("log")
ax.set_yscale("log")
```

![Correlation.](./assets/chapter-04-Cross-Corr.svg)


7. What is the difference between inverse and Penrose-Moore pseudo-inverse? Illustrate the results given a random square matrix to be inverted.

The inverse of the matrix can only be computed on square matrices. By contrast, psudo-inverse can be computed on matrices of arbitrary sizes.

```python
import numpy as np
x = np.random.randn(5, 5)
x_inverse = np.linalg.inv(x)
x_pseudo_inverse = np.linalg.pinv(x)
```

In the case of square matrix, the results are identical.


8. Use Singular Value Decomposition (SVD) to perform Principal Component Analysis (PCA) on the following data points. Plot the main axes of the components.

```python
import numpy as np
x = np.random.multivariate_normal(
    mean=[-1, 1], 
    cov=[[0.3, 0.4], [0.4, 0.3]], 
    size=5000
)
```

```python
import numpy as np
import numpy as np
import matplotlib.pyplot as plt
x = np.random.multivariate_normal(
    mean=[-1, 1], 
    cov=[[0.3, 0.4], [0.4, 0.3]], 
    size=5000
)
x = x - x.mean(axis=0, keepdims=True)

U, S, Vh = np.linalg.svd(x)
variance = np.sqrt(S**2 / 5000)
components = variance[:, None] * Vh

fig, ax = plt.subplots(1, 1)
ax.scatter(x[:, 0], x[:, 1])
ax.arrow(0, 0, components[0, 0], components[0, 1], color="red", head_width=0.1)
ax.arrow(0, 0, components[1, 0], components[1, 1], color="red", head_width=0.1)
ax.set_aspect("equal")
ax.set_xlabel("component 1")
ax.set_ylabel("component 2")
```

![PCA](./assets/chapter-04-pca.svg)

9. Implement double-exponential curve fitting as described by the book "Regressions et Equations Integrale".

```python
def fit_double_exp(x, y):
    """
    Fitting y = b * exp(p * x) + c * exp(q * x)
    Implemented based on:
        Regressions et Equations Integrales by Jean Jacquelin
    Assuming x and y are sorted ascendingly.
    """
    # Start algorithm
    n = len(x)
    S = np.zeros_like(x)
    S[1:] = 0.5 * (y[:-1] + y[1:]) * np.diff(x)
    S = np.cumsum(S)
    SS = np.zeros_like(x)
    SS[1:] = 0.5 * (S[:-1] + S[1:]) * np.diff(x)
    SS = np.cumsum(SS)

    # Getting the parameters
    M = np.empty((4, 4))
    N = np.empty((4, 1))

    M[:, 0] = np.array([np.sum(SS**2), np.sum(SS * S), np.sum(SS * x), np.sum(SS)])

    M[0, 1] = M[1, 0]
    M[1:,1] = np.array([np.sum(S**2),  np.sum(S * x), np.sum(S)])

    M[:2,2] = M[2, :2]
    M[2, 2] = np.sum(x**2)
    M[3, 2] = np.sum(x)

    M[:3,3] = M[3,:3]
    M[2, 3] = M[3, 2]
    M[3, 3] = n

    N[:, 0] = np.array([np.sum(SS * y), np.sum(S * y), np.sum(x * y), np.sum(y)])

    # Regression for p and q
    ABCD = np.matmul(np.linalg.inv(M), N)
    #set_trace()
    A, B, C, D = ABCD.flatten()
    p = 0.5 * (B + np.sqrt(B**2 + 4 * A))
    q = 0.5 * (B - np.sqrt(B**2 + 4 * A))

     # Regression for b, c
    I = np.empty((2, 2))
    J = np.empty((2, 1))

    beta = np.exp(p * x)
    eta  = np.exp(q * x)
    I[0, 0] = np.sum(beta**2)
    I[1, 0] = np.sum(beta * eta)
    I[0, 1] = I[1, 0]
    I[1, 1] = np.sum(eta**2)


    J[:, 0] = [np.sum(y * beta), np.sum(y * eta)]

    bc = np.matmul(np.linalg.inv(I), J)
    b, c = bc.flatten()

    return b, c, p, q
```

10. Try to fit the following data points using the double-exponential curve fitting method.

```python
import numpy as np
x = np.random.rand(500) * 10
x = np.sort(x)
Y = 0.2 * np.exp(-3 * x) + 0.9 * np.exp(-0.5 * x) + 0.05 * np.random.randn(500)
```


```python
import numpy as np
import matplotlib.pyplot as plt
# TODO: import fit_double_exp

x = np.random.rand(500) * 10
x = np.sort(x)
y = 0.2 * np.exp(-3 * x) + 0.9 * np.exp(-0.5 * x) + 0.05 * np.random.randn(500)

# Fit the model
b, c, p, q = fit_double_exp(x, y)

# Predict the fitted curve
x_fit = np.linspace(0, 10, 100)
y_fit = b * np.exp(p * x_fit) + c * np.exp(q * x_fit)

# Plot
fig, ax = plt.subplots(1, 1)
ax.scatter(x, y)
ax.plot(x_fit, y_fit, color="red", label="fitted")
ax.set_xlabel("x")
ax.set_ylabel("y")
ax.legend()
```

![Fit double exponential curve.](./assets/chapter-04-fit-double-exp.svg)
