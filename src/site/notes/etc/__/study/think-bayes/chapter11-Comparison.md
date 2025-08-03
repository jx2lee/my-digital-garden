---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter11-Comparison/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

11장에서 살펴볼 내용
- joint distributions

# Outer Operations
[outer product](https://en.wikipedia.org/wiki/Outer_product)
```python
import numpy as npimport pandas as pd

x = [1, 3, 5]
y = [2, 4]

X, Y = np.meshgrid(x, y)
X * Y
array([[ 2,  6, 10],
       [ 4, 12, 20]])

X + Y
array([[3, 5, 7],
       [5, 7, 9]])

X > Y
array([[False,  True,  True],
       [False, False,  True]])
```

```python
df = pd.DataFrame(X * Y, columns=x, index=y)
df
   1   3   5
2  2   6  10
4  4  12  20
```

# How Tall Is A?
- problem
	- 미국인 성인 남성 중 두명을 A, B 라고 하자
	- 만약 A 가 B 보다 큰 것 같을 때, A 의 키는 얼마인가?
- 다음과 같이 접근해보자
	- 키에 대한 prior 분포를 만든다.
	- A 와 B 의 prior joint 분포를 만든다.
	- A 가 더 크다는 정보를 이용해 update 한다.
	- 갱신한 joint posterior 분포에서 A 의 posterior 를 떼온다.

```python
from scipy.stats import norm
from empiricaldist import Pmf


mean = 178 # 미국의 평균 남성 키
qs = np.arange(mean-24, mean+24, 0.5)

std = 7.7 # 미국의 남성 키의 표준편차
ps = norm(mean, std).pdf(qs) # A/B prior 분포를 정규분포를 사용해보자.

prior = Pmf(ps, qs)
prior.normalize()
```

# Joint Distribution
$$P(A_x~\mathrm{and}~B_y)$$
- $A_X$ : A의 키가 x
- $B_y$ : B의 키가 y
- 두 사건은 독립이다. (A 가 큰게 B 에 영향을 주지 않음)

$$P(A_x~\mathrm{and}~B_y) = P(A_x)~P(B_y)$$

```python
def make_joint(pmf1, pmf2):
    """Compute the outer product of two Pmfs."""
    X, Y = np.meshgrid(pmf1, pmf2)
    return pd.DataFrame(X * Y, columns=pmf1.qs, index=pmf2.qs)

joint = make_joint(prior, prior)
joint
              154.0         154.5  ...         201.0         201.5
154.0  4.066492e-08  4.968249e-08  ...  6.044433e-08  4.968249e-08
154.5  4.968249e-08  6.069973e-08  ...  7.384804e-08  6.069973e-08
155.0  6.044433e-08  7.384804e-08  ...  8.984443e-08  7.384804e-08
155.5  7.322789e-08  8.946639e-08  ...  1.088459e-07  8.946639e-08
156.0  8.834180e-08  1.079319e-07  ...  1.313112e-07  1.079319e-07
             ...           ...  ...           ...           ...
199.5  1.061267e-07  1.296606e-07  ...  1.577467e-07  1.296606e-07
200.0  8.834180e-08  1.079319e-07  ...  1.313112e-07  1.079319e-07
200.5  7.322789e-08  8.946639e-08  ...  1.088459e-07  8.946639e-08
201.0  6.044433e-08  7.384804e-08  ...  8.984443e-08  7.384804e-08
201.5  4.968249e-08  6.069973e-08  ...  7.384804e-08  6.069973e-08
[96 rows x 96 columns]

joint.to_numpy().sum()
1.0
```

# Visualizing the Joint Distribution
```python
import matplotlib.pyplot as plt

def plot_joint(joint, cmap='Blues'):
    """Plot a joint distribution with a color mesh."""
    vmax = joint.to_numpy().max() * 1.1
    plt.pcolormesh(joint.columns, joint.index, joint, 
                   cmap=cmap,
                   vmax=vmax,
                   shading='nearest')
    plt.colorbar()
    
    decorate(xlabel='A height in cm',
             ylabel='B height in cm')
```

![](https://i.imgur.com/ZJIxz5G.png)

```python
def plot_contour(joint):
    """Plot a joint distribution with a contour(등고선)."""
    plt.contour(joint.columns, joint.index, joint,
                linewidths=2)
    decorate(xlabel='A height in cm',
             ylabel='B height in cm')
```

![](https://i.imgur.com/Auun9gp.png)

# Likelihood
joint prior distribution 을 update 하여 joint posterior distribution 을 만들어보자.

```python
x = joint.columns
y = joint.index
X, Y = np.meshgrid(x, y)

A_taller = (X > Y) # A 가 B 보다 크다는 것을 안다.
A_taller.dtype
dtype('bool')

a = np.where(A_taller, 1, 0)
likelihood = pd.DataFrame(a, index=x, columns=y)
```

![](https://i.imgur.com/V13IyEl.png)

# The Update
```python
posterior = joint * likelihood
posterior
       154.0         154.5  ...         201.0         201.5
154.0    0.0  4.968249e-08  ...  6.044433e-08  4.968249e-08
154.5    0.0  0.000000e+00  ...  7.384804e-08  6.069973e-08
155.0    0.0  0.000000e+00  ...  8.984443e-08  7.384804e-08
155.5    0.0  0.000000e+00  ...  1.088459e-07  8.946639e-08
156.0    0.0  0.000000e+00  ...  1.313112e-07  1.079319e-07
      ...           ...  ...           ...           ...
199.5    0.0  0.000000e+00  ...  1.577467e-07  1.296606e-07
200.0    0.0  0.000000e+00  ...  1.313112e-07  1.079319e-07
200.5    0.0  0.000000e+00  ...  1.088459e-07  8.946639e-08
201.0    0.0  0.000000e+00  ...  0.000000e+00  7.384804e-08
201.5    0.0  0.000000e+00  ...  0.000000e+00  0.000000e+00
[96 rows x 96 columns]

def normalize(joint):
    """Normalize a joint distribution."""
    prob_data = joint.to_numpy().sum()
    joint /= prob_data
    return prob_data

normalize(posterior)
0.49080747821526977
```

![](https://i.imgur.com/6hoNy7b.png)

joint prior 분포와 비슷하지만 B 가 A 보다 큰 경우는 제외되어 반쪽짜리 원모양이 나온다.

# Marginal Distributions
joint posterior 분포에서 A/B 각각의 posterior 분포를 구해보자.

```python
column = posterior[180] # A 가 180 일 확률을 구해보면 다음과 같다.
column.head()
154.0    0.000010
154.5    0.000013
155.0    0.000015
155.5    0.000019
156.0    0.000022
Name: 180.0, dtype: float64

column.sum()
0.014808747635171374
```

```python
column_sums = posterior.sum(axis=0) #A 의 posterior 분포는 각 열의 합으로 구할 수 있다.
column_sums.head()
154.0    0.000000e+00
154.5    4.968249e-08
155.0    1.342924e-07
155.5    2.715402e-07
156.0    4.866675e-07
dtype: float64
```

```python
marginal_A = Pmf(column_sums)
154.0    0.000000e+00
154.5    4.968249e-08
155.0    1.342924e-07
155.5    2.715402e-07
156.0    4.866675e-07
             ...     
199.5    5.252914e-04
200.0    4.374926e-04
200.5    3.628035e-04
201.0    2.995769e-04
201.5    2.463125e-04
```

joint 분포로 부터 가져온 단일변수 분포를 marginal 분포라고 한다.

![](https://i.imgur.com/i8xdG1m.png)

```python
row_sums = posterior.sum(axis=1)
marginal_B = Pmf(row_sums)
```

![](https://i.imgur.com/HXt8skK.png)

```python
def marginal(joint, axis):
    """Compute a marginal distribution."""
    return Pmf(joint.sum(axis=axis))

marginal_A = marginal(posterior, axis=0)
marginal_B = marginal(posterior, axis=1)
```

![](https://i.imgur.com/pLXrLje.png)

```python
prior.mean()
177.99516026921506

print(marginal_A.mean(), marginal_B.mean())
89.51704156110516 85.20558192870402
```

표준편차를 이용해 posterior 가 prior 보다 좀 더 가운데로 모인 구조임을 확인해본다.
```python
prior.std()
7.624924796641578

print(marginal_A.std(), marginal_B.std())
6.270461177645469 6.280513548175111
```

# Conditional Posteriors
A 키를 재보니 170 이었다고 해보자. 그럼 B 의 키는 몇으로 추측할 수 있을까?

```python
column_170 = posterior[170]

cond_B = Pmf(column_170)
cond_B.normalize()
0.004358061205454471
```

![](https://i.imgur.com/FCHhkMs.png)

# Dependence and Independence
앞서 A B 는 독립적이라고 언급했다. 따라서 $P(A_x | B_y) = P(A_x)$ 와 동일하다.

하지만 posterior 의 경우 A B 가 독립적이지 않다. A 가 B 보다 크다는 정보가 곧 B 의 키에 대한 정보기 때문에 의존하게 된다.