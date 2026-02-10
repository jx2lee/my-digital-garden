---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter17-Regression/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

==overview==
두 변수 간 관계를 모델링하는 단순한 선형 회귀(regression)을 몇 가지 예제를 통해 살펴본다.

# More Snow?

> 이 지역에는 예전만큼 눈이 많이 내리지 않는 것 같습니다. "여기 주변"이란 제가 태어나고 자랐으며 현재 살고 있는 매사추세츠주 노퍽 카운티를 의미합니다. 여기서 '예전'이란 1978년에 27인치의 눈이 내려 몇 주 동안 학교에 가지 않아도 되었던 어릴 때와 비교하는 것입니다.
  다행히도 제 추측을 데이터로 테스트할 수 있습니다. 노퍽 카운티는 북미에서 가장 오래된 연속 기상 기록을 보유하고 있는 블루힐 기상 관측소가 위치한 곳이기도 합니다.
  이 기상 관측소를 비롯한 여러 기상 관측소의 데이터는 미국 국립해양대기청(NOAA)에서 확인할 수 있습니다. 저는 1967년 5월 11일부터 2020년 5월 11일까지 블루힐 관측소에서 데이터를 수집했습니다.

```python
import pandas as pd

df = pd.read_csv('2239075.csv', parse_dates=[2])
df.head(3)
df.head(3)
       STATION                   NAME       DATE  PRCP  ...  WT09  WT11  WT16  WT18
0  USC00190736  BLUE HILL COOP, MA US 1967-05-11  0.43  ...   NaN   NaN   NaN   NaN
1  USC00190736  BLUE HILL COOP, MA US 1967-05-12  0.00  ...   NaN   NaN   NaN   NaN
2  USC00190736  BLUE HILL COOP, MA US 1967-05-13  0.00  ...   NaN   NaN   NaN   NaN

# 우리가 사용할 칼럼은 DATE 와 SNOW 다.
# 연도별 내린 눈의 총 양을 구한다.
df['YEAR'] = df['DATE'].dt.year
snow = df.groupby('YEAR')['SNOW'].sum()
```

![](https://i.imgur.com/kNCzqEj.png)

# Regression Model
베이지안이든 아니든 regression 의 기초는 아래 값들의 합이라고 가정한다.
- A linear function of time
- A series of random values drawn from a distribution that is not changing over time.

$$y = a x + b + \epsilon$$
- y: 우리가 보고자 하는 값(강수량)
- x: year
- $\epsilon$: random value
- a, b: slope & intercept
	- 데이터를 이용해 이 값을 추정한다.

강수량을 예측하기 전 **앱실론의 분포는 평균 0 & 표준편차 $\sigma$ 인 정규분포를 따른다고 가정한다**.
```python
# 가정이 설득력 있는지 직접 그래프를 그려 확인해본다.
from empiricaldist import Pmf

pmf_snowfall = Pmf.from_seq(snow)
mean, std = pmf_snowfall.mean(), pmf_snowfall.std()
mean, std
(64.19038461538462, 26.28802198439569)

from scipy.stats import norm

# 데이터 분포와 비교하기 위해 normal dist 를 생성한다.
dist = norm(mean, std)
qs = pmf_snowfall.qs
ps = dist.cdf(qs)
```

![](https://i.imgur.com/nQ39P13.png)

# Least Squares Regression (최소제곱회귀)
$$y = a x + b + \epsilon$$
- regression model's param
	- a
	- b
	- $\epsilon$ 분포의 표준편차 ($\sigma$)
- 최소제곱회귀를 사용하기 위해 statsmodel 모듈을 사용한다.

```python
# df to series
data = snow.reset_index()
data.head(3)
data
    YEAR   SNOW
0   1967   28.6
1   1968   44.7
2   1969   99.2

# 앞장에서 한것과 동일하게 데이터를 평균만큼 뺀다.
offset = round(data['YEAR'].mean())
data['x'] = data['YEAR'] - offset
data['y'] = data['SNOW']
offset
1994

# slope & intercept 를 추정해보자.
import statsmodels.formula.api as smf

formula = 'y ~ x'
results = smf.ols(formula, data=data).fit()
results.params
Intercept    64.446325
x             0.511880
dtype: float64

# sigma
results.resid.std()
25.385680731210623
```

- 1994 년 초 예산 강수량은 약 64 인치다.
- 연간 0.5인치 비율로 증가한다.
- residual std == $\sigma$

위에서 구한 3개의 추정값을 이용해 모델에 사용한 변수의 사전분포를 만들어본다.

# Priors
```python
# 분포 range 가 다른 이유
## 분포 선택 오류를 확인할 수 있다.
## 정교한 slope 를 만들고 덜 중요한 sigma 에 사용하는 연산량을 줄일 수 있다.

import numpy as np
from utils import make_uniform

qs = np.linspace(-0.5, 1.5, 51)
prior_slope = make_uniform(qs, 'Slope')

qs = np.linspace(54, 75, 41)
prior_inter = make_uniform(qs, 'Intercept')

qs = np.linspace(20, 35, 31)
prior_sigma = make_uniform(qs, 'Sigma')


from utils import make_joint

def make_joint3(pmf1, pmf2, pmf3):
    """Make a joint distribution with three parameters."""
    joint2 = make_joint(pmf2, pmf1).stack()
    joint3 = make_joint(pmf3, joint2).stack()
    return Pmf(joint3)

prior = make_joint3(prior_slope, prior_inter, prior_sigma)
prior.head(3)
lope  Intercept  Sigma
-0.5   54.0       20.0     0.000015
                  20.5     0.000015
                  21.0     0.000015
Name: , dtype: float64

prior.shape
(64821,) # 51, 41, 31
```

# Likelihood
```python
inter = 64
slope = 0.51
sigma = 25

xs = data['x']
ys = data['y']

expected = slope * xs + inter
resid = ys - expected

densities = norm(0, sigma).pdf(resid)

likelihood = densities.prod()
likelihood
1.3551948769060997e-105
```

# The Update
```python
likelihood = prior.copy()

for slope, inter, sigma in prior.index:
    expected = slope * xs + inter
    resid = ys - expected
    densities = norm.pdf(resid, 0, sigma)
    likelihood[slope, inter, sigma] = densities.prod()

posterior = prior * likelihood
posterior.normalize()
6.769832641515688e-107

posterior_slope = posterior.marginal(0) # slope
posterior_inter = posterior.marginal(1) # inter
posterior_sigma = posterior.marginal(2) # sigma
```

![](https://i.imgur.com/tTroUc2.png)

- density 가 가장 높은 sigma 는 약 26 이다.
	- `posterior_sigma.idxmax()`
- nuisance parameter

![](https://i.imgur.com/Xr5yhBC.png)

- posterior mean 은 64 이다.
	- `posterior_inter.mean()`

![](https://i.imgur.com/zmZldQa.png)

- posterior mean 은 약 0.51 이다.

# Marathon World Record
> 많은 달리기 종목에서 시간 경과에 따른 세계 기록 속도를 그래프로 그려보면 그 결과는 놀라울 정도로 직선입니다. 저를 포함한 많은 사람들이 이 현상의 원인을 추측해 왔습니다.  
> 또한 언제쯤 마라톤 세계 기록이 2시간 미만으로 단축될 것인지에 대한 추측도 있었습니다. (참고: 2019년에 엘리우드 킵초게가 2시간 이내에 마라톤을 완주했는데, 이는 제가 충분히 인정할 만한 놀라운 성과이지만 여러 가지 이유로 세계 기록으로 인정받지 못했습니다.)  
> 따라서 베이지안 회귀의 두 번째 예로 마라톤(남성 선수)의 세계 기록 진행 상황을 고려하고, 선형 모델의 매개 변수를 추정하고, 이 모델을 사용하여 2시간 벽을 언제 깰지 예측해 보겠습니다.

purpose: **마라톤 선수가 2시간대 벽을 언제 깰 것인가?**

![](https://i.imgur.com/puTsvXq.png)

- date: 세계 신기록이 깨진 날의 pandas timestamp
- speed: 세계 신기록 페이스($mph^6$)
```python
# regression 을 위해 1995년 값을 전체 데이터셋에서 빼자 (1995년 -> 중간지점)
offset = pd.to_datetime('1995')
timedelta = table['date'] - offset

data['x'] = timedelta.dt.total_seconds() / 3600 / 24 / 365.24

import statsmodels.formula.api as smf

formula = 'y ~ x'
results = smf.ols(formula, data=data).fit()
results.params
Intercept 12.464040
x 0.015931
 dtype: float64
# intercept -> 1995년 세계 신기록 (얼추 비슷)

# sigma
results.resid.std()
0.04419653543387603
```

# The Priors
```python
qs = np.linspace(0.012, 0.018, 51)
prior_slope = make_uniform(qs, 'Slope')

qs = np.linspace(12.4, 12.5, 41)
prior_inter = make_uniform(qs, 'Intercept')

qs = np.linspace(0.01, 0.21, 31)
prior_sigma = make_uniform(qs, 'Sigma')

# joint prior dist
prior = make_joint3(prior_slope, prior_inter, prior_sigma)

prior
Slope  Intercept  Sigma   
0.012  12.4       0.010000    0.000015
                  0.016667    0.000015
                  0.023333    0.000015
                  0.030000    0.000015
                  0.036667    0.000015
                                ...   
0.018  12.5       0.183333    0.000015
                  0.190000    0.000015
                  0.196667    0.000015
                  0.203333    0.000015
                  0.210000    0.000015
Name: , Length: 64821, dtype: float64

# likelihood
xs = data['x']
ys = data['y']
likelihood = prior.copy()

for slope, inter, sigma in prior.index:
    expected = slope * xs + inter
    resid = ys - expected
    densities = norm.pdf(resid, 0, sigma)
    likelihood[slope, inter, sigma] = densities.prod()

# update
posterior = prior * likelihood
posterior.normalize()

# marginal
posterior_slope = posterior.marginal(0)
posterior_inter = posterior.marginal(1)
posterior_sigma = posterior.marginal(2)
```

![](https://i.imgur.com/DQPfcug.png)

- posterior_inter.mean() = 12.5, 1995년 예측한 마라톤 세계 신기록

![](https://i.imgur.com/0eguYEs.png)

- posterior_slope.mean() = 0.015

# Prediction
posterior 의 임의의 값을 가져와 위 생성한 regression 식에 대입하여 prediction 값을 만든다.
```python
sample = posterior.choice(101)

xs = np.arange(-25, 50, 2)
pred = np.empty((len(sample), len(xs)))

for i, (slope, inter, sigma) in enumerate(sample):
    epsilon = norm(0, sigma).rvs(len(xs))
    pred[i] = inter + slope * xs + epsilon

# 5,50,95 percentiles
low, median, high = np.percentile(pred, [5, 50, 95], axis=0)

# graph
times = pd.to_timedelta(xs*365.24, unit='days') + offset
plt.fill_between(times, low, high, 
                 color='C2', alpha=0.1)
plt.plot(times, median, color='C2')

plot_speeds(data)
```

![](https://i.imgur.com/HpECp7J.png)
- 점선: 2시간 마라톤 페이스(13.1)
- 2030-2040 사이에 기록을 깰 수 있을것으로 예측했다.
	- 보다 정확한 값을 구하려면 점선과 그래프가 만나는 지점을 구한다.
```python
# 정확한 연도 예측해보기!
from scipy.interpolate import interp1d

future = np.array([interp1d(high, xs)(13.1),
                   interp1d(median, xs)(13.1),
                   interp1d(low, xs)(13.1)])

dts = pd.to_timedelta(future*365.24, unit='day') + offset
pd.DataFrame(dict(datetime=dts),
             index=['early', 'median', 'late'])

early    2028-03-24 16:47:21.722121600
median   2035-03-10 14:59:51.082915200
late     2040-12-29 22:53:36.679804800
```
