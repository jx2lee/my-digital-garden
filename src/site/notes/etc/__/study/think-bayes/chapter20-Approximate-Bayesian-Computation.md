---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter20-Approximate-Bayesian-Computation/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

- Approximate Bayesian Computation, 이하 ABC 에 대해 살펴본다.
- 다른 방법에 비해 많은 연산을 필요로 하지만, 그만큼 구현하고 효율적이기도 하다.

# The Kidney Tumor Problem (신장 종양 문제)
> "저는 신장암 4기이며 군에서 전역하기 전에 암이 발생했는지 확인하려고 합니다. 은퇴일과 발견일을 감안할 때 제가 언제 질병에 걸렸을 확률이 50대 50인지 확인할 수 있습니까? 은퇴 날짜에 대한 확률을 결정할 수 있습니까? 내 종양은 발견 당시 15.5cm x 15cm였습니다. 2등급이었습니다."

- 부피 배가 시간(volumetric doubling time): 종양 크기가 두배가 되는데 걸리는 시간
- 상호 배가 시간(Reciprocal doubling time, RDT): 연간 종양 크기 배수

# A Simple Growth Model
- two assumptions
	- 종양 배가 시간은 일정한 상수다.
	- 종양은 구형을 띈다. (동그란 모양이다)
- define two points
	- `t1`: 질문자가 전역한 시간
	- `t2`: 종양이 발견된 시간

```python
# t1: 1cm 였을 때 t2 때의 크기를 추정해본다.
import numpy as np

def calc_volume(diameter):
    """Converts a diameter to a volume."""
    factor = 4 * np.pi / 3
    return factor * (diameter/2.0)**3

d1 = 1
v1 = calc_volume(d1)
v1
0.5235987755982988

# 장 외 공저 논문에서 vdt 의 median: 811 이라고 이야기한다. vdt 로 rdt 의 값을 구해본다.
median_doubling_time = 811
rdt = 365 / median_doubling_time
rdt
0.45006165228113443

interval = 9.0
doublings = interval * rdt
doublings
4.05055487053021

# v1 과 doubling 이 주어졌을 때, t2 때의 부피는 다음과 같다.
v2 = v1 * 2**doublings
v2
8.676351488087187

def calc_diameter(volume):
    """Converts a volume to a diameter."""
    factor = 3 / np.pi / 4
    return 2 * (factor * volume)**(1/3)

d2 = calc_diameter(v2)
d2
2.5494480788327483
```

전역 때 종양 지름이 1cm 였고 median 비율로 성장했다면(논문에서 이야기한 811), 종양을 발견했을 때 약 2.5cm 가 된다.

# A More General Model
진단 시점의 종양 크기가 주어졌을 때, 연령대별 크기가 어떻게 다른지에 대한 문제를 풀어보자.
- 성장률 분포로부터 값을 가져온다.
- 구간 끝 종양 크기를 구한다.
- 종양 크기가 특정 값을 초과할 때까지 위 과정을 반복한다.

```python
rdts = [5.089,  3.572,  3.242,  2.642,  1.982,  1.847,  1.908,  1.798,
        1.798,  1.761,  2.703, -0.416,  0.024,  0.869,  0.746,  0.257,
        0.269,  0.086,  0.086,  1.321,  1.052,  1.076,  0.758,  0.587,
        0.367,  0.416,  0.073,  0.538,  0.281,  0.122, -0.869, -1.431,
        0.012,  0.037, -0.135,  0.122,  0.208,  0.245,  0.404,  0.648,
        0.673,  0.673,  0.563,  0.391,  0.049,  0.538,  0.514,  0.404,
        0.404,  0.33,  -0.061,  0.538,  0.306]

rdt_sample = np.array(rdts)
len(rdt_sample)
53

# 논문에 나온 데이터를 기반으로 RDF 를 추정해보자.
from utils import kde_from_sample

qs = np.linspace(-2, 6, num=201)
pmf_rdt = kde_from_sample(rdt_sample, qs)
```

![](https://i.imgur.com/AFlLMkN.png)

# Simulation
종양 크기가 최대가 될 때까지 시간구간에 대해 시뮬레이션을 진행한다.
- 각 구간의 시작점에, 성장률 분포 하나의 값을 가져와 구간 끝점에서의 종양크기를 구한다.
- 구간 월 데이터의 mean 값이 245일이기 때문에 245일로 구간을 나누었다.
- 지름이 3cm 보다 작으면 수술할 일이 거의 없고 성장으로 인한 혈액소모가 거의 발생하지 않기 때문에 초기 지름은 3cm 로 설정하였다.
	- 최대 지름은 20cm 로 설정하였다.
```python
interval = 245 / 365      # year
min_diameter = 0.3        # cm
max_diameter = 20         # cm

# max value from calc_volume
v0 = calc_volume(min_diameter)
vmax = calc_volume(max_diameter)
v0, vmax

import pandas as pd

def simulate_growth(pmf_rdt):
    """Simulate the growth of a tumor."""
    age = 0
    volume = v0
    res = []
    
    while True:
        res.append((age, volume))
        if volume > vmax: # 부피가 우리가 설정한 지름20cm 일때의 값을 넘으면 loop 를 탈출한다.
            break

        rdt = pmf_rdt.choice()
        age += interval 
        doublings = rdt * interval
        volume *= 2**doublings
        
    columns = ['age', 'volume']
    sim = pd.DataFrame(res, columns=columns)
    # 부피를 calc_diameter 함수를 이용해 지름으로 변환한다.
    sim['diameter'] = calc_diameter(sim['volume'])
    return sim

sim = simulate_growth(pmf_rdt)
sim.head(3)
sim.tail(3)
```

![](https://i.imgur.com/l6P4wEX.png)

![](https://i.imgur.com/6doXgjd.png)

![](https://i.imgur.com/1su0P80.png)

- 시간(year)에 따라 성장하는 종양의 지름(log)
- (순서대로)점선은 4/8/16cm 일 때의 log 값
- 16cm 일 때 최소 10년, 최대 40년이며 보통 15-30 년일 가능성이 높았다.
- 보다 자세한 값을 구하기 위해 interpolate(보간) 하여 확률을 구해보자

```python
from scipy.interpolate import interp1d

def interpolate_ages(sims, diameter):
    """Estimate the age when each tumor reached a given size."""
    ages = []
    for sim in sims:
        interp = interp1d(sim['diameter'], sim['age'])
        age = interp(diameter)
        ages.append(float(age))
    return ages

from empiricaldist import Cdf

ages = interpolate_ages(sims, 15)
cdf = Cdf.from_seq(ages)
print(cdf.median(), cdf.credible_interval(0.9))
22.31854530374061 [13.47056554 34.49632276]

1 - cdf(9.0)
0.9900990099009901
```

- 지름 15 종양의 경우 median year 는 약 22이고 90% credible interval 13-34 를 갖는다.
- 종양이 생긴지 9년이 안되었을 경우 약 1% 미만이다. (`1-cdf(9.0)`)

하지만 이 결과는 두 가지 문제점이 있다.
- 각 구간 성장률은 이전 성장률과 독립이다 가정했지만, 실제로는 이전에 빠르게 자란 종양은 이후에도 빠르게 자라는 경우가 많았다. 즉, 성장률은 시간과 상관관계가 있다고 볼 수 있다.
- 종양의 크기가 구형을 띈다고 가정했다.

(책에서는 다루지 않았지만) 시간에 따른 상관관계를 가진다는 가정하에 시물레이션을 진행했다.
- **결과 해석은 책을 같이 참고해보자.**
- 저자는 종양이 전역 후 생기지 않았을 수 있다고 생각한다.

# Approximate Bayesian Calculation (근사 베이지안 계산)
1장에서 다룬것과 같이 데이터가 모두 확보되었다면 굳이 베이즈 정리를 사용하지 않아도 된다고 이야기한다. 위 과정은 ABC 를 위한 첫번째 단계이고 다음 예제를 살펴보며 나가보도록 한다.

# Counting Cells
> 이 예제는 카메론 데이비슨 필론의 블로그 게시물에서 가져온 것입니다. 이 글에서 그는 생물학자들이 액체 샘플에서 세포의 농도를 추정하기 위해 사용하는 프로세스를 모델링합니다. 그가 제시한 예는 맥주 양조에 사용되는 효모와 물의 혼합물인 '효모 슬러리'의 세포 수를 세는 것입니다.  
> 
> 이 과정에는 두 단계가 있습니다
> - 먼저, 슬러리의 농도가 세포 계수에 적합할 정도로 낮아질 때까지 희석합니다.  
> - 그런 다음 직사각형 그리드에 고정된 양의 액체를 담는 특수 현미경 슬라이드인 혈구 세포계에 소량의 샘플을 넣습니다.  
> 
> 현미경으로 세포와 격자를 볼 수 있으므로 세포를 정확하게 세는 것이 가능합니다.

- example
	- 예를 들어 세포 농도를 알 수 없는 효모 슬러리가 있다고 가정한다.
	- 1mL 로 시작하여 9mL의 물이 담긴 셰이커에 샘플을 넣고 잘 섞어 희석한다.
	- 그런 다음 다시 한 번 희석하고 또 희석한다. 한 번 희석할 때마다 농도가 10배씩 감소한다고 하면, 세 번 희석하면 농도가 1000배 감소한다.
	- 그런 다음 희석된 샘플을 5x5 그리드에 0.0001mL 용량을 가진 혈구계산판에 올린다. 그리드에는 25개의 사각형이 있지만, 그 중 몇 개(예: 5개)만 검사하고 검사한 사각형의 총 세포 수를 결과로 이용한다.
- 위 과정은 간단하지만 오차가 발생할 여지가 있다.  
	- 희석 과정에서 피펫을 사용하여 액체를 측정하기 때문에 측정 오류가 발생할 수 있습니다.  
	- 혈구 분석기의 액체 양은 사양과 다를 수 있습니다.  
	- 샘플링의 각 단계에서 무작위적인 변화로 평균 세포 수보다 많거나 적은 수의 세포를 선택할 수 있다.
- 데이비드 필론은 이런 오차를 나타내는 PyMC 모델을 만들었는데, **이 모델을 살펴본 후 ABC 를 적용해본다.**

```python
total_squares = 25
squares_counted = 5
yeast_counted = 49 # 발견된 총 세포수

# part1
# 효모의 집중도인 year_conc 의 prior distribution
import pymc3 as pm
billion = 1e9 # 이 숫자는 어떻게 나온거지?

with pm.Model() as model:
    yeast_conc = pm.Normal("yeast conc", 
                           mu=2 * billion, sd=0.4 * billion)
    shaker1_vol = pm.Normal("shaker1 vol", 
                               mu=9.0, sd=0.05)
    # shaker2_vol 과 shaker3_vol 은 두/세번째 넣은 물의 부피이다.
    shaker2_vol = pm.Normal("shaker2 vol", 
                               mu=9.0, sd=0.05)
    shaker3_vol = pm.Normal("shaker3 vol", 
                               mu=9.0, sd=0.05)

# part2
# 효모 슬러리에서 가져온 표본의 분포를 구한다.
with model:
    yeast_slurry_vol = pm.Normal("yeast slurry vol",
                                    mu=1.0, sd=0.01)
    shaker1_to_shaker2_vol = pm.Normal("shaker1 to shaker2",
                                    mu=1.0, sd=0.01)
    shaker2_to_shaker3_vol = pm.Normal("shaker2 to shaker3",
                                    mu=1.0, sd=0.01)

# 위에서 구한 sample & shaker 분포를 이용해 final_dilution 을 구한다.
with model:
    dilution_shaker1 = (yeast_slurry_vol / 
                        (yeast_slurry_vol + shaker1_vol))
    dilution_shaker2 = (shaker1_to_shaker2_vol / 
                        (shaker1_to_shaker2_vol + shaker2_vol))
    dilution_shaker3 = (shaker2_to_shaker3_vol / 
                        (shaker2_to_shaker3_vol + shaker3_vol))
    final_dilution = (dilution_shaker1 * 
                      dilution_shaker2 * 
                      dilution_shaker3)

# part3
# 세 번재 shaker 에서 가져온 sample 을 혈구계산판 그리드에 넣는다.
with model:
	# 음의 값을 제외하기 위해 gamma 분포를 이용한다.
    chamber_vol = pm.Gamma("chamber_vol", 
                           mu=0.0001, sd=0.0001 / 20)

# 실제 집중도를 구하기 위해 희석도와 격자 부피를 곱한다.
with model:
	# 이때는 poisson 분포를 이용한다. (?!)
    yeast_in_chamber = pm.Poisson("yeast in chamber", 
        mu=yeast_conc * final_dilution * chamber_vol)

# part4 (final)
# 그리드 안 세포는 squares_counted/total_squares 의 확률로 우리가 측정할 사각형에 들어갈 것이다.
with model:
	# 이때는 binomial 분포를 사용한다. (?!)
    count = pm.Binomial("count", 
                        n=yeast_in_chamber, 
                        p=squares_counted/total_squares,
                        observed=yeast_counted)

# 완성된 분포를 통해 sample 을 가져온다.
options = dict(return_inferencedata=False)

with model:
    trace = pm.sample(1000, **options)

# yeast_conc 의 posterrior 의 summary statistics 를 구한다.
posterior_sample = trace['yeast conc'] / billion
cdf_pymc = Cdf.from_seq(posterior_sample)
print(cdf_pymc.mean(), cdf_pymc.credible_interval(0.9))
2.2712488367301873 [1.8531491 2.7017654]
```

- ml 당 약 23억개 가량으로 90% credible interval 18-27억을 갖는다.
- 위 과정은 아까 언급했던대로 데이비드 필론이 구한 과정을 그대로 따라했다. MCMC 로 충분히 풀 수 있지만 이제부터 ABC 로 세포수를 추정해보자.

# Cell Counting with ABC
- ABC 의 기본 개념
	- prior dist 로 매개변수의 sample 생성
	- sample 의 매개변수 쌍으로 시뮬레이션

```python
with model:
    prior_sample = pm.sample_prior_predictive(10000)

count = prior_sample['count']
print(count.mean())
40.1144

# prior sample 에서 simulation 한 결과인 count 가 관측값 49와 같은 element 만 고른다. (yeast_counted = 49 # 발견된 총 세포수)
mask = (count == 49)
mask.sum()
221

# mask 에서 시뮬레이션에 대한 yeast_conc 값을 선택한다.
posterior_sample2 = prior_sample['yeast conc'][mask] / billion

# posterior dist 의 CDF 를 추정한다.
cdf_abc = Cdf.from_seq(posterior_sample2)
print(cdf_abc.mean(), cdf_abc.credible_interval(0.9))
2.275872303142668 [1.87509925 2.72428803]
```

![](https://i.imgur.com/K0MX7wQ.png)

두 방법론에 의한 분포는 비슷하지만, 표본크기가 작은 ABC 에 노이즈가 더 많다.

# When Do We Get to the Approximate Part? (추정하는 부분은 언제 구할까?)
- ABC 특징
	- 매개변수의 prior distribution
	- 데이터를 생성하는 시뮬레이션
	- 시뮬레이션 결과값이 데이터와 일치하는 것을 받아들여야하는 기준

앞 절에서 count\==49 인 시뮬레이션만 인정하고 나머지는 기각했다. 그 결과 몇백개(221) 샘플만 남았는데, 이는 비효율적이다. 만약 49점에 가까울수록 부분점수를 준다면 어떨까?

```python
n = prior_sample['yeast in chamber']
n.shape
(10000,)

p = squares_counted/total_squares
p

from scipy.stats import binom

# yeast_counted 의 likelihhod 를 구하는데 이항분포를 사용해보자
likelihood = binom(n, p).pmf(yeast_counted).flatten()

# n*p 가 실제값과 가까워지면 likelihood 는 커질 것이다.
plt.plot(n*p, likelihood, '.', alpha=0.03, color='C2')

decorate(xlabel='Expected count (number of cells)',
         ylabel='Likelihood')
```

![](https://i.imgur.com/oGzPW0a.png)

위를 이용해서 가중치로 활용할 수 있을 것 같다.
```python
qs = prior_sample['yeast conc'] / billion
ps = likelihood
posterior_pmf = Pmf(ps, qs)

# 가중치를 부여하기 위해 sorting 한다.
posterior_pmf.sort_index(inplace=True)
posterior_pmf.normalize()

print(posterior_pmf.mean(), posterior_pmf.credible_interval(0.9)) 
2.2723483584950497 [1.85449376 2.70563828]
```

MCMC 로 구했던 값과 유사하다. (line:276)

![](https://i.imgur.com/M8vX6xd.png)

- 분포는 비슷하지만 MCMC 에 노이즈가 다수 보인다.
- 이 예제에서 ABC 는 MCMC 보다 posterior 추정치를 낮게 구하는데 더 적은 연산을 했기 때문에 효율적이다.
	- 일반적인 경우는 아니다.
- 보통의 ABC 는 연산량이 많기 때문에 **최후의 수단**이라고 표현한다.
