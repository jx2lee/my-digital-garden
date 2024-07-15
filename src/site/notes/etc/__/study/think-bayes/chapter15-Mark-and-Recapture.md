---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter15-Mark-and-Recapture/","noteIcon":""}
---

#think-bayes #probability #study 

---

==overview==
- 모집단(population)에서 표본을 추출해 특정 표식(mark)을 하고 다시 표본을 추출하는 방식(=mark and recapture)
	- 두번째 추출에서 mark 된 표본을 통해 모집단 크기를 추정할 수 있음
- 3개의 param 을 사용하기 때문에 3차원 joint dist 이용

# The Grizzly Bear Problem
> 1996년과 1997년에 연구자들은 캐나다 브리티시 컬럼비아와 앨버타의 여러 지역에 곰 덫을 설치하여 회색곰의 개체수를 추정했습니다. 이 기사에서는 이 실험에 대해 설명합니다.
> 
> 이 '덫'은 미끼와 여러 가닥의 철조망으로 구성되어 있으며, 미끼에 접근한 곰의 털 샘플을 채취하기 위한 것입니다. 연구진은 머리카락 샘플을 사용하여 DNA 분석을 통해 개별 곰을 식별합니다.
> 
> 첫 번째 세션 동안 연구진은 76개 지점에 덫을 설치했습니다. 10일 후 다시 돌아와 1043개의 모발 샘플을 확보하고 23마리의 곰을 식별했습니다. 10일간의 두 번째 세션에서 연구진은 19마리의 곰으로부터 1191개의 샘플을 얻었으며, 19마리 중 4마리는 첫 번째 세션에서 확인된 곰의 것이었습니다.

- 실험 간 개별 곰들이 관측될 확률을 모델링 해야함
- 알려지지 않은 확률로 각 곰들이 실험에 포착한다고 가정해보고 문제를 접근

**실제 곰의 모집단 개체수가 100이라고 가정해보자**
- N: 실제 개체수, 100
- K: 첫 실험에서 확인된 곰의 수, 23
- n: 두 번째 실험에서 관측된 곰의 수, 19
- k: 두 번째 실험에서 확인된 곰의 수, 4
- $\binom{K}{k}$ 는 K 의 모집단에서 크기 k 의 부분집합 수

==hypergeometric dist, 초기하분포==
$$\binom{K}{k} \binom{N-K}{n-k}/ \binom{N}{n}$$

```python
import numpy as np
from scipy.stats import hypergeom

N = 100
K = 23
n = 19

ks = np.arange(12)
ps = hypergeom(N, K, n).pmf(ks)
```

![](https://i.imgur.com/5lWeqKw.png)

그럼 K, n, k 이 주어졌을 때 N 은 어떻게 추정할 수 있을까?

# The update
곰의 개체수는 50~500 사이이고 범위 내 확률은 모두 동일하다고 가정한다.

```python
import numpy as np
from utils import make_uniform

# calculate prior
qs = np.arange(50, 501)
prior_N = make_uniform(qs, name='N')
prior_N.shape
(451, )

# calculate likelihood
Ns = prior_N.qs
K = 23
n = 19
k = 4

likelihood = hypergeom(Ns, K, n).pmf(k)

# update
posterior_N = prior_N * likelihood
posterior_N.normalize()
0.07755224277106727
```

![](https://i.imgur.com/LNqDjqf.png)

```python
# max
posterior_N.max_prob()
109

# 분포가 오른쪽 꼬리가 긴 형태이므로 mean 은 max 보다 크다
posterior_N.mean()
173.79880627085637

# 신뢰구간도 엄청 길다
posterior_N.credible_interval(0.9)
array([ 77., 363.])
```

# Two-Parameter Model
```python
# 곰의 수 N 과 곰을 발견할 확률 p 2개 param 을 사용하는 모델을 만들어본다.
# assumption
## 두 실험에서의 확률 p 는 동일하다
## 확률은 독립적이다

K = 23
n = 19
k = 4

# K10: 1회차 발견/2회차 미발견 곰 수
# k01: 1회차 미발견/2회차 발견 곰 수
# k11: 1/2회차 발견 곰 수
k10 = 23 - 4
k01 = 19 - 4
k11 = 4

# assumption: 우린 N 과 p 를 알고 있다. (N=100, p=0.2)
# calculate likelihood
N = 100

observed = k01 + k10 + k11
k00 = N - observed
k00
62

# 리스트로 데이터를 저장하고 쓰자
x = [k00, k01, k10, k11]
x
[62, 15, 19, 4]

p = 0.2
q = 1-p
y = [q*q, q*p, p*q, p*p]
y
[0.6400000000000001,
 0.16000000000000003,
 0.16000000000000003,
 0.04000000000000001]
```

데이터 확률은 다항분포로 나타난다.

$$\frac{N!}{\prod x_i!} \prod y_i^{x_i}$$
- N: population
- x: 각 그룹에 대한 수열
- y: 각 그룹의 확률에 대한 수열

```python
from scipy.stats import multinomial

likelihood = multinomial.pmf(x, N, y)
likelihood
0.0016664011988507257
```

# The Prior
```python
# N 의 prior: prior_N, p 의 prior: uniform
## prior_N 도 uniform
qs = np.linspace(0, 0.99, num=100)
prior_p = make_uniform(qs, name='p')

# get joint prior
from utils import make_joint

joint_prior = make_joint(prior_p, prior_N)
joint_prior.shape
(451, 100)

# stack 함수로 df -> 1-dim 시리즈로 변경
from empiricaldist import Pmf

joint_pmf = Pmf(joint_prior.stack())
joint_pmf.head(3)
```

![](https://i.imgur.com/vyLFuf3.png)

# The Update
```python
likelihood = joint_pmf.copy()

# param 별 likelihood 를 계산한다.
observed = k01 + k10 + k11

for N, p in joint_pmf.index:
    k00 = N - observed
    x = [k00, k01, k10, k11]
    q = 1-p
    y = [q*q, q*p, p*q, p*p]
    likelihood[N, p] = multinomial.pmf(x, N, y)

# posterior
posterior_pmf = joint_pmf * likelihood
posterior_pmf.normalize()

# Series to DF
joint_posterior = posterior_pmf.unstack()
```

![](https://i.imgur.com/5egWXxn.png)

- 가장 가능성 높은 N 은 이전과 동일하게 대략 100
	- p: 0.2
- p 가 낮을수록 N 은 커지고, p 가 높을 수록 N 이 낮아지는 걸 보면 두 변수는 상관관계가 있을 것 같은데
	- check marginal

# Joint and Marginal Dist

```python
from utils import marginal

posterior2_p = marginal(joint_posterior, 0)
posterior2_N = marginal(joint_posterior, 1)
```

![](https://i.imgur.com/QN21xZ1.png)

# The Lincoln Index Problem
> "프로그램에서 20개의 버그를 발견한 테스터가 있다고 가정해 보겠습니다. 실제로 프로그램에 얼마나 많은 버그가 있는지 추정하고 싶습니다. 적어도 20개 이상의 버그가 있다는 것을 알고 있고 테스터에 대한 신뢰도가 매우 높다면 20개 정도의 버그가 있다고 생각할 수 있습니다. 하지만 테스터가 별로 좋지 않을 수도 있습니다. 수백 개의 버그가 있을 수도 있습니다. 버그가 얼마나 많은지 어떻게 알 수 있을까요? 테스터 한 명으로는 알 방법이 없습니다. 하지만 테스터가 두 명이라면 테스터의 숙련도를 모르더라도 좋은 아이디어를 얻을 수 있습니다."

```python
# assumption: 첫 검수자가 20개, 두번째 검수자가 15개, 겹치는 건 3개라고 가정하자. 총 버그수를 추정하는 문제를 풀어본다.
k10 = 20 - 3
k01 = 15 - 3
k11 = 3

# p0: 첫 번째 검수자가 버그를 발견할 확률
# p1: 두 번째 검수자가 버그를 발견할 확률
p0, p1 = 0.2, 0.15

def compute_probs(p0, p1):
    """Computes the probability for each of 4 categories."""
    q0 = 1-p0
    q1 = 1-p1
    return [q0*q1, q0*p1, p0*q1, p0*p1]

y = compute_probs(p0, p1)
y
[0.68, 0.12, 0.17, 0.03]

# 위 확률을 알고 있다 가정하고 N posterior 를 구해보자
# lower:32, upper: 350 & uniform 이라 가정
qs = np.arange(32, 350, step=5)
prior_N = make_uniform(qs, name='N')
prior_N.head(3)
```

![](https://i.imgur.com/GkKmd8m.png)

```python
data = np.array([0, k01, k10, k11])

likelihood = prior_N.copy()
observed = data.sum()
x = data.copy()

for N in prior_N.qs:
    x[0] = N - observed
    likelihood[N] = multinomial.pmf(x, N, y)

# update
posterior_N = prior_N * likelihood
posterior_N.normalize()
0.0003425201572557094
```

![](https://i.imgur.com/8E6AVay.png)

- p0 p1 을 알고 있다는 가정하에서 posterior mean 은 102, 90% credible interval 은 (77, 127)
	- 결과는 버그를 발견할 확률을 알고 있다는 가정

# Three-Parameter Model
```python
# N, p0, p1 을 사용하는 모델
# N prior: prior_N
qs = np.linspace(0, 1, num=51)
prior_p0 = make_uniform(qs, name='p0')
prior_p1 = make_uniform(qs, name='p1')

# joint dist of N & p0
joint2 = make_joint(prior_p0, prior_N)
joint2.shape
(64, 51)

# joint dist of N & p0 to stack!
joint2_pmf = Pmf(joint2.stack())
joint2_pmf.head(3)
```

![](https://i.imgur.com/0YoSTgg.png)

```python
# joint dist of (N + p0) + p1
joint3 = make_joint(prior_p1, joint2_pmf)
joint3.shape
(3264, 51)

# joint dist of N & p0 & p1 to stack!
joint3_pmf = Pmf(joint3.stack())
joint3_pmf.shape
(166464,)

joint3_pmf.head(3)
```

![](https://i.imgur.com/O3OIUn8.png)

```python
# calculate likelihood
likelihood = joint3_pmf.copy()
observed = data.sum()
x = data.copy()

for N, p0, p1 in joint3_pmf.index:
    x[0] = N - observed
    y = compute_probs(p0, p1)
    likelihood[N, p0, p1] = multinomial.pmf(x, N, y)

# update
posterior_pmf = joint3_pmf * likelihood
posterior_pmf.normalize()
8.941088283758206e-06
```

- N's marginal posterior
	- ![](https://i.imgur.com/e3diipV.png)
	- posterior mean: 105
- p0 & p1's marginal posterior
	- ![](https://i.imgur.com/jEoeFpu.png)
	- posterior mean: 23% & 18%

# Summary
==이번장에서 살펴본 내용==
- mark and recapture 개념
- 확률 분포
	- 초기하분포: 표본을 재수집하지 않고 추출하는 경우 사용, 이항분포의 변형 형태
	- 다항분포: 두 가지 이상의 가능한 결과가 있는 이항 분포를 일반화한 분포
- 세 param 을 이용한 모델