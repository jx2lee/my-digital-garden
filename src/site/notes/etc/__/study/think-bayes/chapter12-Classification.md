---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter12-Classification/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

# Penguin Data
분류는 1990년대에 1세대 스팸 필터의 기초로 유명해진 베이지안 방법의 가장 잘 알려진 응용 분야일 수 있다. 
  
이 장에서는 남극 팔머 장기 생태 연구소의 크리스틴 고먼 박사가 수집하여 제공한 데이터를 사용하여 bayesian classification 를 보여준다(고먼, 윌리엄스, 프레이저, "남극 펭귄(피고셀리스 속) 커뮤니티 내의 생태학적 성적 이형성과 환경적 변동성", 2014년 3월 참조). 이 데이터를 사용하여 펭귄을 종별로 분류할 것이다.

![](https://i.imgur.com/Bc9lOT1.png)

```python
import pandas as pd

df = pd.read_csv('penguins_raw.csv')
df.shape
(344, 17)
```

- 체질량(g) 단위
- 플리퍼 길이(mm)
- 부리 길이(mm)
- 부리 깊이 (mm)

![](https://i.imgur.com/MyHF48k.png)

> 동일한 종의 경우 분산이 작아 분류에 사용하기 매우 유용하다.
> -> 분산이란 확률변수가 기댓값으부터 얼마나 떨어져 있는 곳에 분포하는지를 가늠하는 숫자이다. 같은 종의 펭귄들의 특정 값에 대한 분산이 작다는 것은, 종을 특징한 값으로 표현할 수 있기 때문에 분류에 용이하다고 저자가 이야기한 것 같다.

```python
def make_cdf_map(df, colname, by='Species2'):
    """Make a CDF for each species."""
    cdf_map = {}
    grouped = df.groupby(by)[colname]
    for species, group in grouped:
        cdf_map[species] = Cdf.from_seq(group, name=species)
    return cdf_map
```

![](https://i.imgur.com/nRwSjwD.png)
![](https://i.imgur.com/oaqqejB.png)

> 정규분포 특징과 시그모이드
> - features
> 	- 평균, 중앙값(median) 및 mode 는 동일하다.
> 	- 분포는 평균에 대해 대칭을 이루며, 절반은 평균보다 낮고 절반은 평균보다 높다.
> 	- 분포는 평균(mean)과 표준 편차(standard deviation)라는 두 가지 값으로 설명할 수 있다.
> - sigmoid
> 	- **시그모이드 함수**는 S자형 곡선 또는 **시그모이드 곡선을** 갖는 [수학 함수](https://ko.wikipedia.org/wiki/%ED%95%A8%EC%88%98 "함수")이다. 시그모이드 함수의 예시로는 [로지스틱 함수](https://ko.wikipedia.org/wiki/%EB%A1%9C%EC%A7%80%EC%8A%A4%ED%8B%B1_%ED%95%A8%EC%88%98 "로지스틱 함수")가 있으며 다음 수식으로 정의된다.
> 		- ${\displaystyle S(x)={\frac {1}{1+e^{-x}}}={\frac {e^{x}}{e^{x}+1}}}$
> 		- logistic function
> 			- ![](https://i.imgur.com/CBuSSij.png)
> 	- definition
> 		- 시그모이드 함수는 실함수로써 유계이고 미분가능하며, 모든 점에서 음이 아닌 미분값을 가지고 단 하나의 변곡점을 가진다.

# Normal Models
1. 세 종에 대한 prior distribution 을 정의한다.
2. 각 종에 대한 가설의 likelihood 을 구한다.
3. 각 가설에 대한 posterior distribution 을 구한다.

```python
from scipy.stats import norm

def make_norm_map(df, colname, by='Species2'):
    """Make a map from species to norm object."""
    norm_map = {}
    grouped = df.groupby(by)[colname]
    for species, group in grouped:
        mean = group.mean()
        std = group.std()
        norm_map[species] = norm(mean, std)
    return norm_map

flipper_map = make_norm_map(df, 'Flipper Length (mm)')
flipper_map.keys()
dict_keys(['Adelie', 'Chinstrap', 'Gentoo'])
```

```python
# 발길이가 193 일 때 각 종의 likelihood ?
data = 193
flipper_map['Adelie'].pdf(data)

hypos = flipper_map.keys()
likelihood = [flipper_map[hypo].pdf(data) for hypo in hypos]
likelihood
[0.054732511875530694, 0.05172135615888162, 5.8660453661990634e-05]
```

확률밀도와 확률간 관계
> - 직접 설명하는 것 보단 링크를 참고하면 도움이 될 것 같다. 이전시간에 교영님이 설명해준 내용도 함께 참고하면 더 좋을 것 같다.
> - https://actuary.skku.edu/actuarial/roadmap2.do?mode=download&articleNo=111941&attachNo=81385

# The Update
```python
from empiricaldist import Pmf

prior = Pmf(1/3, hypos)
prior
Adelie Penguin (Pygoscelis adeliae)          0.333333
Chinstrap penguin (Pygoscelis antarctica)    0.333333
Gentoo penguin (Pygoscelis papua)            0.333333
Name: , dtype: float64

posterior = prior * likelihood
posterior.normalize()
posterior
Adelie Penguin (Pygoscelis adeliae)          0.513860
Chinstrap penguin (Pygoscelis antarctica)    0.485589
Gentoo penguin (Pygoscelis papua)            0.000551
Name: , dtype: float64
```

발 길이가 193mm 인 펭귄이 젠투일 순 가능성은 거의 없지만, 아델리 펭귄이나 턱끈 펭귄일 가능성은 비슷하다.

```python
def update_penguin(prior, data, norm_map):
    """Update hypothetical species."""
    hypos = prior.qs
    likelihood = [norm_map[hypo].pdf(data) for hypo in hypos]
    posterior = prior * likelihood
    posterior.normalize()
    return posterior

posterior1 = update_penguin(prior, 193, flipper_map)
posterior1
Adelie Penguin (Pygoscelis adeliae)          0.513860
Chinstrap penguin (Pygoscelis antarctica)    0.485589
Gentoo penguin (Pygoscelis papua)            0.000551
Name: , dtype: float64
```

발 길이로는 분류가 확실히 되지 않기 때문에 부리 길이의 분포를 계산해보자.

```python
culmen_map = make_norm_map(df, 'Culmen Length (mm)')

# 부리 상단 길이가 48mm 인 펭귄을 보았다고 하자.
posterior2 = update_penguin(prior, 48, culmen_map)
posterior2
Adelie Penguin (Pygoscelis adeliae)          0.001557
Chinstrap penguin (Pygoscelis antarctica)    0.474658
Gentoo penguin (Pygoscelis papua)            0.523785
Name: , dtype: float64
```

# Naive Bayesian Classification
```python
def update_naive(prior, data_seq, norm_maps):
    """Naive Bayesian classifier
    
    prior: Pmf
    data_seq: sequence of measurements
    norm_maps: sequence of maps from species to distribution
    
    returns: Pmf representing the posterior distribution
    """
    posterior = prior.copy()
    for data, norm_map in zip(data_seq, norm_maps):
        posterior = update_penguin(posterior, data, norm_map)
    return posterior

colnames = ['Flipper Length (mm)', 'Culmen Length (mm)']
norm_maps = [flipper_map, culmen_map]
```

발 길이가 193mm, 부리 길이가 48mm 인 펭귄을 발견했다고 해보자. update 해본다.

```python
data_seq = 193, 48
posterior = update_naive(prior, data_seq, norm_maps)
posterior
Adelie Penguin (Pygoscelis adeliae)          0.003455
Chinstrap penguin (Pygoscelis antarctica)    0.995299
Gentoo penguin (Pygoscelis papua)            0.001246
Name: , dtype: float64

posterior.max_prob()
'Chinstrap'
```

```python
import numpy as np

df['Classification'] = np.nan

for i, row in df.iterrows():
    data_seq = row[colnames]
    posterior = update_naive(prior, data_seq, norm_maps)
    df.loc[i, 'Classification'] = posterior.max_prob()

def accuracy(df):
    """Compute the accuracy of classification."""
    valid = df['Classification'].notna()
    same = df['Species'] == df['Classification']
    return same.sum() / valid.sum()

accuracy(df)
0.9473684210526315
```

발/부리 길이 등 피처 간 연관성을 고려하지 않아 나이브 라고 불린다고 한다. 추가로 피처의 결합분포를 사용하는 덜 나이브한 분류기도 살펴본다.

# Joint Distributions
```python
import matplotlib.pyplot as plt

def scatterplot(df, var1, var2):
    """Make a scatter plot."""
    grouped = df.groupby('Species')
    for species, group in grouped:
        plt.plot(group[var1], group[var2],
                 label=species, lw=0, alpha=0.3)
    
    decorate(xlabel=var1, ylabel=var2)

var1 = 'Flipper Length (mm)'
var2 = 'Culmen Length (mm)'
scatterplot(df, var1, var2)
```

![](https://i.imgur.com/Y7IUopu.png)

- 결합분포는 대략 타원형 형태를 나타낸다.
- 1/3사분면에 분포했으므로 부리 길이와 발 길이간 상관관계가 있음을 나타낸다.

만약 각 특징이 독립적이라면?
```python
def make_pmf_norm(dist, sigmas=3, n=101):
    """Make a Pmf approximation to a normal distribution."""
    mean, std = dist.mean(), dist.std()
    low = mean - sigmas * std
    high = mean + sigmas * std
    qs = np.linspace(low, high, n)
    ps = dist.pdf(qs)
    pmf = Pmf(ps, qs)
    pmf.normalize()
    return pmf

from utils import make_joint

joint_map = {}
for species in hypos:
    pmf1 = make_pmf_norm(flipper_map[species])
    pmf2 = make_pmf_norm(culmen_map[species])
    joint_map[species] = make_joint(pmf1, pmf2)
```

![](https://i.imgur.com/jhN0dc7.png)

영 시원치 않다. 다변량 multivariate normal 분포를 이용해 모델링해본다.

# Multivariate Normal Distribution
다변량 정규분포는 특정값의 평균, 각 피처의 값이 얼마나 퍼져 있는 지를 나타내는 숫자 분산과, 피처 간 관계를 숫자로 나타낸 공분산이 들어있는 공분산 행렬로 정의한다.

측정 데이터를 이용해 평균과 공분산 행렬을 추정할 수 있다.
```python
features = df[[var1, var2]]

mean = features.mean()
mean
Flipper Length (mm)    200.915205
Culmen Length (mm)      43.921930
dtype: float64

cov = features.cov()
cov
                     Flipper Length (mm)  Culmen Length (mm)
Flipper Length (mm)           197.731792           50.375765
Culmen Length (mm)             50.375765           29.807054
```

```python
from scipy.stats import multivariate_normal

multinorm = multivariate_normal(mean, cov)
multinorm
<scipy.stats._multivariate.multivariate_normal_frozen object at 0x1459794c0>

def make_multinorm_map(df, colnames):
    """Make a map from each species to a multivariate normal."""
    multinorm_map = {}
    grouped = df.groupby('Species')
    for species, group in grouped:
        features = group[colnames]
        mean = features.mean()
        cov = features.cov()
        multinorm_map[species] = multivariate_normal(mean, cov)
    return multinorm_map

multinorm_map = make_multinorm_map(df, [var1, var2])
multinorm_map
{'Adelie Penguin (Pygoscelis adeliae)': <scipy.stats._multivariate.multivariate_normal_frozen object at 0x1458b1f40>, 'Chinstrap penguin (Pygoscelis antarctica)': <scipy.stats._multivariate.multivariate_normal_frozen object at 0x14597b100>, 'Gentoo penguin (Pygoscelis papua)': <scipy.stats._multivariate.multivariate_normal_frozen object at 0x14597b580>}
```

# Visualizing Normal Distribution

![](https://i.imgur.com/bhrVrEj.png)

다변량 정규분포는 특징 간 상관관계를 고려하기 때문에 이 데이터에는 다변량 정규분포가 더 적합하다. 경계션 간 겹치는 데이터가 적기 때문에 이를 사용하면 분류기 성능을 높일 수 있다.

# A Less Naive Classifier
앞서 update_penguin() 에서 norm_map 의 각 값은 norm 객체였지만 multivariate_normal 객체를 넣어도 동일하게 동작한다.

발 길이가 193이고 부리 길이가 48인 펭귄을 분류해보자.
```python
data = 193, 48
posterior_multi = update_penguin(prior, data, multinorm_map)
posterior_multi
Adelie Penguin (Pygoscelis adeliae)          0.002740
Chinstrap penguin (Pygoscelis antarctica)    0.997257
Gentoo penguin (Pygoscelis papua)            0.000003
Name: , dtype: float64

df['Classification'] = np.nan

for i, row in df.iterrows():
    data = row[colnames]
    posterior = update_penguin(prior, data, multinorm_map)
    df.loc[i, 'Classification'] = posterior.idxmax()

accuracy(df)
0.9532163742690059
```
