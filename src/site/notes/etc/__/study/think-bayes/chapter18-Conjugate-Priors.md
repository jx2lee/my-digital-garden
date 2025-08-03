---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter18-Conjugate-Priors/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

- 앞서 bayes update 에 사용한 그리드 접근 방법의 한계는 매개변수가 증가할 수록 연산량이 증가한다.
- 대안
	- conjugate prior 이용
	- (이후 장) MCMC
	- ABC

# The World Cup Problem Revisited
```python
from scipy.stats import gamma

alpha = 1.4
dist = gamma(alpha)

# grid approximation
import numpy as np
from utils import pmf_from_dist

lams = np.linspace(0, 10, 101)
prior = pmf_from_dist(dist, lams)

# lams 별 4점 득점할 likelihood 계산
from scipy.stats import poisson

k = 4
likelihood = poisson(lams).pmf(k)

# update
posterior = prior * likelihood
posterior.normalize()
0.05015532557804499
```

conjugate prior 로 위 문제를 풀어보자.

# The Conjugate Prior
- 월드컵 문제에서 prior dist 에 gamma 분포를 사용했다.
	- **이는 포아송 분포의 켤레사전분포가 gamma 분포이기 때문이다.**
	- 켤레가 의미하는대로 두 분포가 연결되어 있거나 한 쌍으로 볼 수 있다.
```python
# alpha & beta 두 매개변수를 사용하는 gamma dist 객체를 만드는 함수
def make_gamma_dist(alpha, beta):
    """Makes a gamma object."""
    dist = gamma(alpha, scale=1/beta)
    dist.alpha = alpha
    dist.beta = beta
    return dist

# alpha: 1.4 & beta: 1 인 gamma dist
alpha = 1.4
beta = 1

prior_gamma = make_gamma_dist(alpha, beta)
prior_gamma.mean()
1.4

# update function
def update_gamma(prior, data):
    """Update a gamma prior."""
    k, t = data
    alpha = prior.alpha + k
    beta = prior.beta + t
    return make_gamma_dist(alpha, beta)

# update! (1 경기(t) 4 득점(k) 갱신)
data = 4, 1
posterior_gamma = update_gamma(prior_gamma, data)
posterior_conjugate = pmf_from_dist(posterior_gamma, lams)
```

![](https://i.imgur.com/y9Zin3u.png)

# What the Actual?
$$\lambda^{\alpha-1} e^{-\lambda \beta}$$
- pdf of gamma distribution
	- $\alpha$ and $\beta$ 에 대해 $\lambda$ 각 값에 대한 확률밀도
	- 정규화 인수(Normalizing factor)는 생략했다.
- `가정`
	- 한 팀이 t 게임동안 k 만큼 득점했다.
- 가정에 의해 poisson dist 의 pmf 는 다음과 같다.
	- $$\lambda^k e^{-\lambda t}$$
	- gamma 와 동일하게 정규화 인수는 생략했다.
- 두 pmf 의 곱은 다음과 같다.
	- $$\lambda^{\alpha-1+k} e^{-\lambda(\beta + t)} = \lambda^{(\alpha+k)-1} e^{-\lambda(\beta + t)}$$
	- **이는 $\alpha + k$ and $\beta + t$ 매개변수를 갖는 정규화되지 않은 gamma 분포임을 확인할 수 있다.**
	- 수식으로 gamma 분포를 유도하는 것을 자세히 보고 싶다면 [링크](https://towardsdatascience.com/gamma-distribution-intuition-derivation-and-examples-55f407423840)를 참고하자.

# Binomial Likelihood (이항가능도)

```python
# 유로동전 문제를 다시 살펴본다.
# 그리드 알고리즘을 사용할 때 uniform 분포를 사용했다.
from utils import make_uniform

xs = np.linspace(0, 1, 101)
uniform = make_uniform(xs, 'uniform')

# 250회 중 140회 앞면이 나오는 경우에 대한 likelihood
from scipy.stats import binom

k, n = 140, 250
xs = uniform.qs
likelihood = binom.pmf(k, n, xs)

# update!
posterior = uniform * likelihood
posterior.normalize()
```

이항분포의 켤레사전분포인 베타 분포를 이용해 문제를 풀어보자.
```python
import scipy.stats

def make_beta(alpha, beta):
    """Makes a beta object."""
    dist = scipy.stats.beta(alpha, beta)
    dist.alpha = alpha
    dist.beta = beta
    return dist

# uniform -> alpha:1 & beta:1 인 beta dist
alpha = 1
beta = 1

prior_beta = make_beta(alpha, beta)
```

$$x^{\alpha-1} (1-x)^{\beta-1}$$
$$x^{k} (1-x)^{n-k}$$
- 두 식을 gamma 분포 유도하는 방법과 동일하게 곱해보면.. (*again, omitted normalizing factor!*)
	- $$x^{\alpha-1+k} (1-x)^{\beta-1+n-k} = x^{(\alpha+k)-1} (1-x)^{(\beta+n-k)-1}$$
	- $\alpha+k$ and $\beta+n-k$ 매개변수를 갖는 beta dist

```python
# conjugate dist 를 이용하면 매개변수의 뜻을 더 잘 이해할 수 있다(?)
# alpha: 관측한 성공 횟수
# beta: 실패 횟수
def update_beta(prior, data):
    """Update a beta distribution."""
    k, n = data
    alpha = prior.alpha + k
    beta = prior.beta + n - k
    return make_beta(alpha, beta)

data = 140, 250
posterior_beta = update_beta(prior_beta, data)

# 그리드 알고리즘으로 구한 posterior 와 비교하기 위한 작업
posterior_conjugate = pmf_from_dist(posterior_beta, xs)
```

![](https://i.imgur.com/8War4H3.png)

# Lions and Tigers and Bears

> 사자와 호랑이, 곰만 있는 야생 동물 보호구역을 방문했지만 각각 몇 마리가 있는지는 모른다고 가정해 보겠습니다. 투어 중에 사자 3마리, 호랑이 2마리, 곰 1마리를 보게 됩니다. 모든 동물이 표본에 나타날 확률이 같다고 가정할 때, 다음에 볼 동물이 곰일 확률은 얼마인가요?

```python
# 가정: 사자/호랑이/곰 비율 -> 0.4/0.3/0.3
from scipy.stats import multinomial

data = 3, 2, 1
n = np.sum(data)
ps = 0.4, 0.3, 0.3

multinomial.pmf(data, n, ps)
0.10368

# 동물 비율에 대한 prior dist 를 선택하고 갱신을 통해 데이터의 확률을 구할 수 있지만
# multinomial 분포의 켤레사전분포인 디클레레 분포를 활용해보자
```

# The Dirichlet Distribution
```python
# 디클레레의 경우 분포값은 확률 x 의 벡터이고 매개변수는 벡터 alpha
from scipy.stats import dirichlet

alpha = 1, 2, 3
dist = dirichlet(alpha)

dist.rvs()
array([[0.2742988 , 0.28900196, 0.43669924]])
# 임의의 값의 합은 모두 1이며 서로 exclusive 하다.

# 1000개 임의의 값을 디클레레 분포에서 가져와보자
sample = dist.rvs(1000)

# 분포의 생김새를 살펴보기 위해 cdf 를 구하고 그려본다.
from empiricaldist import Cdf

cdfs = [Cdf.from_seq(col) 
        for col in sample.transpose()]
```

![](https://i.imgur.com/yB8ELaC.png)

- column 0: 가장 값이 작은 매개변수 & 가장 낮은 확률값을 갖는다.
- column 2: 가장 값이 큰 매개변수 & 가장 높은 확률값을 갖는다.
- 위 그래프를 보고 주변분포가 베타 분포임을 알 수 있다는데, 직접 marginal 분포를 구해보자.
	- 베타분포의 CDF 는 다음과 같다. (위키)
	- ![](https://i.imgur.com/tRy24Tn.png)
```python
def marginal_beta(alpha, i):
    """디클레레 분포의 i 번째 주변분포를 구한다"""
    total = np.sum(alpha)
    return make_beta(alpha[i], total-alpha[i])

marginals = [marginal_beta(alpha, i)
             for i in range(len(alpha))]
```

![](https://i.imgur.com/Z2f1TwE.png)

# Summary
- 슬프게도 켤레사전분포를 이용해 풀수 있는 문제는 그리 많지 않다.
	- ..?!
- 따라서 이후에 설명한 ABC & MCMC 가 필요하다.
	- 라는 걸 보여주기 위해 이 장을 끼워넣었나 싶다.
