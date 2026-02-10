---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter14-Survival-Analysis/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

특정 사건이 지속되는 시간을 구하는 통계적 방법인 survival analysis 에 대해 설명한다.

> bayesian methods 는 불완전한 데이터를 사용해야 하는 경우 특히 좋은 선택이라고 한다. (저자왈)

# The Weibull Distribution (베이불 분포, 웨이불 분포)
- 생존 분석에 사용하는 분포
- 분포의 central tendency 에 영향을 미치는 람다와 모양에 shape 에 영향을 주는 k 를 이용

```python
from scipy.stats import weibull_min

def weibull_dist(lam, k):
    return weibull_min(k, scale=lam)

lam = 3
k = 0.8
actual_dist = weibull_dist(lam, k)
```

![](https://i.imgur.com/lpCxqRk.png)

```python
data = actual_dist.rvs(10) # rvs: argument 만큼 random sampling 하는 함수
data
array([ 0.31758457, 17.35490262,  7.9886383 ,  0.89713818,  0.29910551,
        3.50907964,  1.04786533,  0.07217011,  0.90204999,  3.49996289])
```

- 분포 parameter(매개변수) 를 이용해 sample(표본)을 만들어봤다.
- 반대로 sample 을 이용해 parameter 를 추정해본다.

```python
from utils import make_uniform

# 람다의 prior
lams = np.linspace(0.1, 10.1, num=101)
prior_lam = make_uniform(lams, name='lambda')

# k-prior
ks = np.linspace(0.1, 5.1, num=101)
prior_k = make_uniform(ks, name='k')

# 두 param 의 joint dist
# col: lamda, row: k
from utils import make_joint

prior = make_joint(prior_lam, prior_k)

# axis=0: lamda, axis=1: k, axis=2: data with meshgrid
lam_mesh, k_mesh, data_mesh = np.meshgrid(
    prior.columns, prior.index, data)

# weibull dist
densities = weibull_dist(lam_mesh, k_mesh).pdf(data_mesh)
densities.shape
(101, 101, 10)

# get likelihood (sum of axis=2)
likelihood = densities.prod(axis=2)
likelihood.sum()
7.360050998462799e-09

# posterior
from utils import normalize

posterior = prior * likelihood
normalize(posterior)
```

```python
# 하나씩 살펴봤던 과정을 하나의 함수로 묶어본다.
def update_weibull(prior, data):
    """Update the prior based on data."""
    lam_mesh, k_mesh, data_mesh = np.meshgrid(
        prior.columns, prior.index, data)
    
    densities = weibull_dist(lam_mesh, k_mesh).pdf(data_mesh)
    likelihood = densities.prod(axis=2)

    posterior = prior * likelihood
    normalize(posterior)

    return posterior

posterior = update_weibull(prior, data)
```

![](https://i.imgur.com/8wgOK4k.png)

- sample 을 통해 lambda 와 k 를 추정했다.
	- 대략 1~4 사이 분포하고(처음 3이 여기 포함) k 도 (0.5, 1.5) 에 0.8 존재

# Incomplete Data (불완전 데이터)
- 유기견 보호소에 유기견이 드러온 후 입양되는 데까지 걸리는 시간을 추정한다.
- 8주 동안 10마리 개가 입양된 것을 확인했다고 하자. 개가 들어오는 시간은 균등분포를 따른다고 가정한다.

```python
start = np.random.uniform(0, 8, size=10)
start
array([4.0653269 , 6.78916117, 1.16379555, 0.32436407, 2.97318449,
       2.44585731, 6.82059322, 6.82584851, 5.34669996, 4.04786079])

# 임보 시간은 와이불 분포를 따른다고 가정한다.
duration = actual_dist.rvs(10)
duration
array([3.01820543, 0.4958247 , 2.77579963, 4.24047529, 3.97448903,
       3.84258084, 0.29658979, 1.55392259, 2.94100143, 4.96571093])

# create df
import pandas as pd

d = dict(start=start, end=start+duration)
obs = pd.DataFrame(d)
obs = obs.sort_values(by='start', ignore_index=True)
obs

# 불완전한 데이터를 나타내기 위해 status 값을 추가한다.
censored = obs['end'] > 8
obs.loc[censored, 'end'] = 8
obs.loc[censored, 'status'] = 0
```

![](https://i.imgur.com/8686Kuq.png)

- 생존시간 그래프
- 생존 기간 값을 칼럼에 추가한다.

```python
obs['T'] = obs['end'] - obs['start']
```

# Using Incomplete Data
complete & incomplete 한 데이터를 모두 이용해 임보 시간의 분포에 대한 param 을 추정한다.

```python
# data1/data2
data1 = obs.loc[~censored, 'T']
data2 = obs.loc[censored, 'T']
```

![](https://i.imgur.com/f81YVdc.png)

```python
posterior1 = update_weibull(prior, data1)

# update weibull with incomplete data
def update_weibull_incomplete(prior, data):
    """Update the prior using incomplete data."""
    lam_mesh, k_mesh, data_mesh = np.meshgrid(
        prior.columns, prior.index, data)
    
    # evaluate the survival function
    probs = weibull_dist(lam_mesh, k_mesh).sf(data_mesh)
    likelihood = probs.prod(axis=2)

    posterior = prior * likelihood
    normalize(posterior)

    return posterior

posterior2 = update_weibull_incomplete(posterior1, data2)
```

![](https://i.imgur.com/a2t1yzo.png)

- 람다 값의 범위가 넓어졌다.
- marginal dist 를 구해보고 확인한다.

```python
posterior_lam2 = marginal(posterior2, 0)
posterior_k2 = marginal(posterior2, 1)
```

![](https://i.imgur.com/7faLCDt.png)

- posterior dist 의 density 는 0이 되지 않음
- 분포를 정확히 만들고 싶다면 prior 분포를 최대한 넓게 선정하고 update 하면 된다고 한다.
	- 직접 해보진 않았다. 혹시 해보신 분 분포를 그려보면 0 으로 되는지 확인해주세요.

![](https://i.imgur.com/UoY9u6J.png)

- incomplete 데이터를 사용한 margin dist 가 왼쪽으로 이동
- 일반적으로 불완전한 데이터를 사용한 경우 정보가 부족해서 posterior 폭이 넓음

# Light Bulbs (전구)
> 정격 40W, 220V(AC)의 필립스(인도) 램프 50개를 수평 방향으로 가져와 설치한 후 11m x 7m의 실험실 공간에 균일하게 배치했습니다.  
> 이 어셈블리를 12시간 간격으로 정기적으로 모니터링하여 고장을 확인했습니다. 고장이 발생한 순간을 기록하고 마지막 전구까지 고장이 발생하도록 총 32개의 데이터 포인트를 확보했습니다.

```python
# load data
df = pd.read_csv('lamps.csv', index_col=0)
df.head()
```

![](https://i.imgur.com/3qOB6W2.png)

- h: 전구가 방전될 때 까지의 시간
- f: 매 시간 방전된 전구의 갯수
- k: 매 시간 방전되지 않은 전구의 갯수

```python
from empiricaldist import Pmf

pmf_bulb = Pmf(df['f'].to_numpy(), df['h'])
pmf_bulb.normalize()
50

# set prior for lamda, k
lams = np.linspace(1000, 2000, num=51)
prior_lam = make_uniform(lams, name='lambda')

ks = np.linspace(1, 10, num=51) # 100 to 51, 연산은 빨라지지만 정확도는 조금 떨어질 수 있다.
prior_k = make_uniform(ks, name='k')

# joint prior dist
prior_bulb = make_joint(prior_lam, prior_k)

# 서로 다른 수명주기를 갖는 전구가 더 적으므로 50 번 갱신을 위해 값을 변형한다.

data_bulb = np.repeat(df['h'], df['f'])
len(data_bulb)
50

# update weibull
posterior_bulb = update_weibull(prior_bulb, data_bulb)
```

![](https://i.imgur.com/1JyEOeg.png)

summary 값을 이용해 평균 수명을 구할 수 있다.

# Posterior Means
```python
lam_mesh, k_mesh = np.meshgrid(
    prior_bulb.columns, prior_bulb.index)

means = weibull_dist(lam_mesh, k_mesh).mean()
means.shape
(51, 51)

# means * posterior
prod.to_numpy().sum()
1412.7242774305005 # 전구 평균 수명은 약 1,413h

# summarize above lines
def joint_weibull_mean(joint):
    """Compute the mean of a joint distribution of Weibulls."""
    lam_mesh, k_mesh = np.meshgrid(
        joint.columns, joint.index)
    means = weibull_dist(lam_mesh, k_mesh).mean()
    prod = means * joint
    return prod.to_numpy().sum()
```

# Posterior Predictive Dist (사후 예측 분포)

Q. 동일한 전구 100개 설치, 1,000시간 이후 방전된 전구수의 분포는?

```python
# lambda:1550, k:4.25 인 경우 전구가 방전될 확률
lam = 1550
k = 4.25
t = 1000

prob_dead = weibull_dist(lam, k).cdf(t)
prob_dead
0.14381685899960547

# 100개 전구, 방전된 전구 수는 binomial 을 따른다고 가정한다.
from utils import make_binomial

n = 100
p = prob_dead
dist_num_dead = make_binomial(n, p)
```

- lambda & k 를 안다는 전제 하에서는 가능하지만 우리는 요 값을 모름
- 대신 lambda & k 가 가질 수 있는 값과 각각의 확률에 대한 posterior 를 알고 있음
- **따라서 ppd 는 단일 이항분포가 아님**
	- mixture (이항분포 with posterior weight)

```python
posterior_series = posterior_bulb.stack()
posterior_series.head()
k    lambda
1.0  1000.0    8.146763e-25
     1020.0    1.210486e-24
     1040.0    1.738327e-24
     1060.0    2.418201e-24
     1080.0    3.265549e-24
dtype: float64

pmf_seq = []
for (k, lam) in posterior_series.index:
    prob_dead = weibull_dist(lam, k).cdf(t)
    pmf = make_binomial(n, prob_dead)
    pmf_seq.append(pmf)

from utils import make_mixture

post_pred = make_mixture(posterior_series, pmf_seq)
```

![](https://i.imgur.com/aTQmgGr.png)

# summary
==이번 장에서 살펴본 내용==
- 생존분석이란
- lifetime 을 모델링하는 weibull dist
- use joint dist about weibull param 
	- lifetime 을
		- 정확히 아는 경우
		- 최소 범위를 아는 경우
		- 주어진 구간 내 있는 경우
- complete/incomplete 한 데이터로 bayesian 적용