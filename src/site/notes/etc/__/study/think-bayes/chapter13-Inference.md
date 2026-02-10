---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter13-Inference/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

# Inference (추론)

> p-value 는 어쩌고?

- 일반적 추론 방식과 베이지안 추론 방식을 비교할 때마다 가장 자주 나오는 질문 중 하나
- 고전적 통계 추론
	- student t-검정 -> p-value
	- 영가설 유의성 검정 (null hypothesis significance testing)
- bayesian inference
	- 두 집단 간 차이에 대한 posterior 분포 계산
	- 위 분포를 기준으로
		- 가장 가능성이 높은 차이의 크기
		- 실제 차이를 포함할 가능성이 높은 신뢰할 수 있는 간격
		- 우월 확률 (probability of superiority) 또는 차이가 임계값을 초과할 확률

# Improving Reading Ability

> 한 교육자는 교실에서 새로운 지시형 읽기 활동이 초등학생의 읽기 능력 향상에 도움이 되는지 테스트하기 위해 실험을 실시했습니다. 그녀는 21명의 학생으로 구성된 3학년 학급에 8주 동안 이러한 활동을 따르도록 했습니다. 23명의 3학년 학생으로 구성된 대조군 학급은 활동 없이 동일한 커리큘럼을 따랐습니다. 8주가 끝날 무렵, 모든 학생들은 치료가 개선하도록 설계된 읽기 능력의 측면을 측정하는 읽기 능력 정도(DRP) 시험을 치렀습니다.

```python
import pandas as pd

df = pd.read_csv('drp_scores.csv', skiprows=21, delimiter='\t')
df.head(3)
  Treatment  Response
0   Treated        24
1   Treated        43
2   Treated        58
```

```python
# Treated 와 Control 그룹으로 나누어보자
grouped = df.groupby('Treatment')
responses = {}

for name, group in grouped:
    responses[name] = group['Response']

# 두 집단의 점수의 CDF 분포를 그리면 다음과 같다.
from empiricaldist import Cdf
from utils import decorate

for name, response in responses.items():
    cdf = Cdf.from_seq(response)
    cdf.plot(label=name)
    
decorate(xlabel='Score', 
         ylabel='CDF',
         title='Distributions of test scores')
```

![](https://i.imgur.com/CIohIxk.png)

- **겹치는 부분은 있으나 Treated (실험군) 가 점수가 높다.**
- 두 집단 모두 완벽한 정규분포는 이루고 있지 않지만 정규분포를 사용해도 무방
	- *CDF 모양이 S 자를 따르고 있다. 이는 정규분포의 CDF 와 유사하므로 저자는 정규분포임을 보여주지 않고 빠르게 넘어간 듯 하다.*
- 학생 모집단에서 점수의 분포는 정규분포를 이루고 아직 평균과 표준편차는 모른다고 가정
	- mu, sigma
	- **이 값을 추정(estimate)하기 위해 bayesian update**

# Estimating Parameters (매개변수 추정)

- parameter 의 prior 분포가 필요
	- **두 개 param -> joint distribution**
- 절차
	- marginal distribution 계산
	- marginal distribution 간 외적을 이용해 joint distribution 계산

mu & sigma uniform 이라 가정한다.

```python
from empiricaldist import Pmf

def make_uniform(qs, name=None, **options):
    """Make a Pmf that represents a uniform distribution."""
    pmf = Pmf(1.0, qs, **options)
    pmf.normalize()
    if name:
        pmf.index.name = name
    return pmf

import numpy as np

qs = np.linspace(20, 80, num=101) #상한과 하한을 설정(20, 80)
prior_mu = make_uniform(qs, name='mean')

qs = np.linspace(5, 30, num=101) #마찬가지로 상/하한 설정(5, 30)
prior_sigma = make_uniform(qs, name='std')

from utils import make_joint # 이전에 살펴본 joint 분포 생성 함수 (retrun dataframe)

prior = make_joint(prior_mu, prior_sigma)

data = responses['Control'] # 대조군 데이터를 미리 생성한다.
data.shape
(23, )
```

# Likelihood (가능도)
```python
# mu & sigma 값의 각 쌍에 대한 점수별 확률을 살펴본다.
mu_mesh, sigma_mesh, data_mesh = np.meshgrid(
    prior.columns, prior.index, data) # x: mu, y: sigma, z: data

mu_mesh.shape
(101, 101, 23)

from scipy.stats import norm

# probability density
densities = norm(mu_mesh, sigma_mesh).pdf(data_mesh)
densities.shape
(101, 101, 23)

from utils import normalize

# axis=2 값 -> likelihood
likelihood = densities.prod(axis=2)
likelihood.shape
(101, 101)

# bayesian update
posterior = prior * likelihood
normalize(posterior)
posterior.shape
(101, 101)
```

```python
def update_norm(prior, data):
    """Update the prior based on data."""
    mu_mesh, sigma_mesh, data_mesh = np.meshgrid(
        prior.columns, prior.index, data)
    
    densities = norm(mu_mesh, sigma_mesh).pdf(data_mesh)
    likelihood = densities.prod(axis=2)
    
    posterior = prior * likelihood
    normalize(posterior)

    return posterior

# Control
data = responses['Control']
posterior_control = update_norm(prior, data)

# Treated
data = responses['Treated']
posterior_treated = update_norm(prior, data)
```

![](https://i.imgur.com/Ctyx6Ow.png)

- 실험이 차이를 만들어냈다면 -> 실험으로 인해 평균 점수가 올라가고 분포 정도(spread) 를 낮춰줬다 라고 볼 수 있다.
- mu & sigma 의 marginal 분포를 살펴보면 보다 명확히 알 수 있다(고 한다)

# Posterior Marginal Distributions

```python
from utils import marginal

pmf_mean_control = marginal(posterior_control, 0)
pmf_mean_treated = marginal(posterior_treated, 0)
```

![](https://i.imgur.com/ZrJQt4K.png)

- posterior 양 tail 쪽이 모두 0 -> prior 상/하한값 범위가 충분함을 의미

```python
Pmf.prob_gt(pmf_mean_treated, pmf_mean_control)
0.980479025187326
```

# Distribution of Differences (차이의 분포)

집단 간 차이를 수량화하고자 sub_dist() 를 이용해 차이 분포를 계산한다.
```python
pmf_diff = Pmf.sub_dist(pmf_mean_treated, pmf_mean_control)

len(pmf_mean_treated), len(pmf_mean_control), len(pmf_diff)
(101, 101, 879)
```

![](https://i.imgur.com/iBTeRlD.png)

제약조건(*외적 시 데이터의 크기가 많이 커진다 & pmf 를 도식화 할 때 지저분하다*)을 해소할 두 가지 방법 중 하나는 CDF 를 그리는 것
![](https://i.imgur.com/EftqbMb.png)

다른 방법은 KDE 를 이용해 PDF 의 smooth approximation 을 구하는 것
```python
from scipy.stats import gaussian_kde

def kde_from_pmf(pmf, n=101):
    """Make a kernel density estimate for a PMF."""
    kde = gaussian_kde(pmf.qs, weights=pmf.ps)
    qs = np.linspace(pmf.qs.min(), pmf.qs.max(), n)
    ps = kde.evaluate(qs)
    pmf = Pmf(ps, qs)
    pmf.normalize()
    return pmf

kde_diff = kde_from_pmf(pmf_diff)
# kde_diff plot
```

![](https://i.imgur.com/6rXWxDN.png)

```python
pmf_diff.mean()
9.954413088940848

pmf_diff.credible_interval(0.9)
array([ 2.4, 17.4])
```

- 시험 점수 평균이 45점 일 때 분포의 평균은 약 10점이므로 상당한 효과를 거두었다 볼 수 있음
	- **45점은 어디서 나온 숫자인가요? 주어진 데이터의 시험점수 평균이 대략 46 이기 때문에 around 45 라고 이야기한건가요?**
- credible interval 에 따르면 2 -> 17.4 를 올리는 효과가 있었다고 볼 수 있음

# Using Summary Statistics
- 만약 우리가 더 큰 데이터를 다룬다면..
	- 3차원 배열을 계산하면 굉장히 오래걸릴 수 있음
	- likelihood 가 굉장히 작아짐
- 이런 경우 summary 값을 구하고 이의 likelihood 를 계산하는 방법
	- **한 집단의 평균과 표준편차를 구하면 likelihood 값을 계산할 수 있음**

```python
# 모평균 mu 와 sigma 가 다음과 같다고 해보자
mu = 42
sigma = 17

# 임의로 n = 20 인 표본을 추출했다고 하자. 표본의 평균은 m, 표본의 표준편차: s
n = 20
m = 41
s = 18
```

mathematical statistics (수리통계) 을 이용해 likelhood 를 계산할 수 있음
1. 𝜇 및 𝜎 가 주어지면 m 의 분포는 파라미터 $\mu$ and $\sigma/\sqrt{n}$ 와 함께 정규 분포이다.
2. 𝑠의 분포는 더 복잡하지만 𝑡=𝑛𝑠2/𝜎2 변환을 계산하면 𝑡의 분포는 매개 변수 𝑛-1 을 사용한 카이제곱분포가 된다.
3. Basu 정리에 따르면 m 와 s 는 독립적이다.

```python
dist_m = norm(mu, sigma/np.sqrt(n)) # 평균의 표본 분포 (sampling distribution of mean)
like1 = dist_m.pdf(m)
like1
0.10137915138497372

t = n * s**2 / sigma**2
t
22.422145328719722

from scipy.stats import chi2

dist_s = chi2(n-1) # 카이제곱분포

like2 = dist_s.pdf(t)
like2
0.04736427909437004

like = like1 * like2 # basu 정리에 따름
like
0.004801750420548287
```

# Update with Summary Statictics
```python
summary = {}

for name, response in responses.items():
    summary[name] = len(response), response.mean(), response.std()
    
summary
{'Control': (23, 41.52173913043478, 17.148733229699484), 'Treated': (21, 51.476190476190474, 11.00735684721381)}
```

Control 의 summary 를 구한다.
```python
n, m, s = summary['Control']
```

mu & sigma 가설값을 외적을 이용해 meshgrid 를 생성한다.
```python
mus, sigmas = np.meshgrid(prior.columns, prior.index)
mus.shape
(101, 101)
```

mu & sigma 의 likelihood 를 계산하고 update 해본다.
```python
like1 = norm(mus, sigmas/np.sqrt(n)).pdf(m)
like1.shape

ts = n * s**2 / sigmas**2
like2 = chi2(n-1).pdf(ts)
like2.shape

posterior_control2 = prior * like1 * like2
normalize(posterior_control2)

# 위 과정을 하나로 묶은 함수
def update_norm_summary(prior, data):
    """Update a normal distribution using summary statistics."""
    n, m, s = data
    mu_mesh, sigma_mesh = np.meshgrid(prior.columns, prior.index)
    
    like1 = norm(mu_mesh, sigma_mesh/np.sqrt(n)).pdf(m)
    like2 = chi2(n-1).pdf(n * s**2 / sigma_mesh**2)
    
    posterior = prior * like1 * like2
    normalize(posterior)
    
    return posterior

# 위 함수를 이용해 Treated 그룹도 update 한다.
data = summary['Treated']
posterior_treated2 = update_norm_summary(prior, data)
```

![](https://i.imgur.com/0J42IUU.png)

이는 앞서 계산했던 것과 비슷해보이지만 marginal distribution 을 구해보면 완전히 같지는 않은 것을 확인할 수 있다.

# Posterior Marginal Distributions

```python
from utils import marginal

pmf_mean_control2 = marginal(posterior_control2, 0)
pmf_mean_treated2 = marginal(posterior_treated2, 0)
```

![](https://i.imgur.com/oAdLXNe.png)


```python
Pmf.prob_gt(pmf_mean_treated, pmf_mean_control)
0.980479025187326
```

- summary 기반 posterior 가 짧고 더 넓은 형태
	- **이는 데이터 분포가 정규분포를 따른다고 가정하고 summary 값을 사용해 갱신했기 때문**
	- 이 가정은 불안정하기 때문에 summary 를 사용하면 일부 정보를 손실할 수 있음
- 정보가 적어지면 parameter 에 대한 확신(certain)이 줄어들 수 밖에 없음