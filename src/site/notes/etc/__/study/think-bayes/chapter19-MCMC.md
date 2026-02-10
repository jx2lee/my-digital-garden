---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter19-MCMC/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

MCMC (Markov Chain Monte Carlo, 이하 MCMC) 는 많은 매개변수를 사용하는 경우 사용할 수 있는 도구이다.
- monte carlo 는 distribution 에서 임의의 sample 을 만드는 방법이다.
- grid algorithm 과 달리 posterior distribution 을 구하지 않는다.

# The World Cup Problem
MCMC 방법을 사용하기전 이전에 살펴보았던 grid approximation 로 월드컵 문제를 풀어본다.

# Grid Approximation
```python
from scipy.stats import gamma
from scipy.stats import poisson
from utils import pmf_from_dist
import numpy as np

alpha = 1.4
prior_dist = gamma(alpha)

# 람다의 가능한 값(0~10)을 만들고 (lams) 람다의 prior 를 구한다.
lams = np.linspace(0, 10, 101)
prior_pmf = pmf_from_dist(prior_dist, lams)

# poisson 으로 likelihood 를 구한다. (4골을 넣은 경우의 likelihood)
data = 4
likelihood = poisson.pmf(data, lams)

posterior = prior_pmf * likelihood
posterior.normalize()
0.05015532557804499
```

# Prior Predictive Distribution (사전 예측분포)
```python
from scipy.stats import poisson
from empiricaldist import Pmf

# 사후 예측분포와 같이 사전 예측분포 추정을 위해 sample 를 추출한다.
# ret: 가능한 골 득점률 lamda 배열
sample_prior = prior_dist.rvs(1000)
sample_prior_pred = poisson.rvs(sample_prior)

# sample to pmf
pmf_prior_pred = Pmf.from_seq(sample_prior_pred)
```

![](https://i.imgur.com/HjVln6i.png)

- 모델이 쓸만한 지 확인하기 위해 prior predictive dist 를 구한다.
- MCMC 방법의 첫 단계이기도 하다.

# Introducing PyMC3
- prior 에서 득점률을 구한다.
- poisson 분포에서 득점수를 구한다.

```python
import pymc3 as pm

with pm.Model() as model:
    lam = pm.Gamma('lam', alpha=1.4, beta=1.0)
    goals = pm.Poisson('goals', lam)
```

# Sampling the Prior
```python
with model:
    trace = pm.sample_prior_predictive(1000)

# trace 는 lam & goals 를 표본과 연결하는 객체이다. lam의 sample 을 구해보자.
sample_prior_pymc = trace['lam']
sample_prior_pymc.shape
(1000,)
```

![](https://i.imgur.com/4VC2AAr.png)
sample_prior_pymc vs sample_prior (scipy gamma 분포에서 추추출한 sample)

```python
# 이번에는 prior predictive 분포에서 표본을 가져와 비교해본다.
sample_prior_pred_pymc = trace['goals']
sample_prior_pred_pymc.shape
(1000,)
```

![](https://i.imgur.com/UO3aYMD.png)
sample_prior_prd_pymc vs sample_prior_pred (scipy.stats 의 possion 으로 구한 prior predictive dist)

# When Do We Get to Inference?
```python
with pm.Model() as model2:
    lam = pm.Gamma('lam', alpha=1.4, beta=1.0)
    goals = pm.Poisson('goals', lam, observed=4)

options = dict(return_inferencedata=False)

with model2:
    trace2 = pm.sample(500, **options) # lam posterior 에서 sample 을 가져오는 함수 이용

sample_post_pymc = trace2['lam']
```

![](https://i.imgur.com/9KIPFmP.png)
sample_post_pymc vs posterior

# Posterior Predictive Distribution
```python
with model2:
    post_pred = pm.sample_posterior_predictive(trace2)

sample_post_pred_pymc = post_pred['goals']

sample_post = posterior.sample(1000)
sample_post_pred = poisson(sample_post).rvs()
```

![](https://i.imgur.com/VKsgUzp.png)
sample_post_pred_pymc vs sample_post_pred

**pymc 는 복잡한 모델에서 진가를 발휘한다.**

# Happiness
> 최근 에스테반 오티즈-오스피나와 맥스 로저가 쓴 '행복과 삶의 만족도'를 읽었는데, 이 책에서는 국가 간, 국가 내, 시간 경과에 따른 소득과 행복의 관계에 대해 여러 가지를 논의합니다.
> 
> 여기에는 행복과 6가지 잠재적 예측 요인 간의 관계를 탐구하는 다중 회귀 분석 결과가 포함된 '세계 행복 보고서'가 인용되어 있습니다:
> - 1인당 GDP로 표시되는 소득
> - 사회적 지원
> - 출생 시 건강 기대 수명
> - 삶의 선택의 자유
> - 관대함
> - 부패에 대한 인식
> 
> 종속 변수는 갤럽 월드 여론조사에서 사용한 '캔트릴 사다리 질문'에 대한 응답의 전국 평균입니다:
> 
> 맨 아래 0부터 맨 위 10까지 계단 번호가 매겨진 사다리를 상상해 보십시오. 사다리의 맨 위는 귀하에게 가능한 최고의 삶을, 맨 아래는 최악의 삶을 나타냅니다. 개인적으로 현재 자신이 사다리의 어느 단계에 서 있다고 생각하십니까?

bayesian regression 으로 문제를 풀어보도록 한다.

# Simple Regression
```python
log_gdp = df['Logged GDP per capita']
```

![](https://i.imgur.com/ozThWPT.png)

$$y = a x + b + \epsilon$$

```python
from scipy.stats import linregress

result = linregress(log_gdp, score)
Slope      0.717738
Intercept  -1.198646

# with pymc
x_data = log_gdp
y_data = score

with pm.Model() as model3:
    a = pm.Uniform('a', 0, 4)
    b = pm.Uniform('b', -4, 4)
    sigma = pm.Uniform('sigma', 0, 2)

    y_est = a * x_data + b
    y = pm.Normal('y', 
                  mu=y_est, sd=sigma, 
                  observed=y_data)
```

- a,b,sigma 의 prior: 충분히 넓은 uniform dist
- y_est: regression equation 의 종속 변수 추정값
- y: y_est/sigma 의 normal dist

```python
# posterior dist 로 부터 500개 sample 을 추출한다.
with model3:
    trace3 = pm.sample(500, **options)
# arviz 라이브러리의 plot_posterior 로 매개변수의 posterior 분포를 그래프로 그릴 수 있다.

trace3
<MultiTrace: 2 chains, 500 iterations, 6 variables>

import arviz as az

with model3:
    az.plot_posterior(trace3, var_names=['a', 'b']);
```

![](https://i.imgur.com/SvKovdM.png)

- HDI: 최고밀도구간(highest-density interval)

행복 문제는 총 8개 매개변수를 사용한다(6개 지표 + intercept/sigma). 이를 grid approximation 으로 접근하면 계산량이 어마어마해진다.
이때 pymc 로 mcmc 로 풀어본다.

# Multiple Regression
```python
columns = ['Ladder score',
           'Logged GDP per capita',
           'Social support',
           'Healthy life expectancy',
           'Freedom to make life choices',
           'Generosity',
           'Perceptions of corruption']

subset = df[columns]

# standardize data
standardized = (subset - subset.mean()) / subset.std()

y_data = standardized['Ladder score']

# 변수별 standarize
x1 = standardized[columns[1]]
x2 = standardized[columns[2]]
x3 = standardized[columns[3]]
x4 = standardized[columns[4]]
x5 = standardized[columns[5]]
x6 = standardized[columns[6]]

with pm.Model() as model4:
    b0 = pm.Uniform('b0', -4, 4)
    b1 = pm.Uniform('b1', -4, 4)
    b2 = pm.Uniform('b2', -4, 4)
    b3 = pm.Uniform('b3', -4, 4)
    b4 = pm.Uniform('b4', -4, 4)
    b5 = pm.Uniform('b5', -4, 4)
    b6 = pm.Uniform('b6', -4, 4)
    sigma = pm.Uniform('sigma', 0, 2)

    y_est = b0 + b1*x1 + b2*x2 + b3*x3 + b4*x4 + b5*x5 + b6*x6
    y = pm.Normal('y', 
                  mu=y_est, sd=sigma, 
                  observed=y_data)

# sample of posterior distribution
with model4:
    trace4 = pm.sample(500, **options)

# 각 매개변수 별 sample 을 뽑고 이의 평균을 구한다.
param_names = ['b1', 'b2', 'b3', 'b4', 'b5', 'b6']

means = [trace4[name].mean() 
         for name in param_names]
index = columns[1:]
table = pd.DataFrame(index=index)
table['Posterior mean'] = np.round(means, 3)
table['94% CI'] = cis
table
```

![](https://i.imgur.com/Ac9eGvj.png)

- 행복은 GDP 와 가장 연관성이 있어 보이고, 그다음으로 사회복지/기대수명/자유 순을 갖는다.
- 나머지 요인은 상대적으로 연관성이 약해보이며, 관용(Generosity)의 credible interval 은 0을 포함하고 있다. 이는 실제로 행복과 상관이 없다고 볼 수 있다.
