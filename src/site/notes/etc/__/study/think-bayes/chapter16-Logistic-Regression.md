---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter16-Logistic-Regression/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study 

---

==overview==
- Bayes's Rule on a logarithmic scale
- logistic regression

# Log Odds
> 계산이론 문제

$$O(H|F) = O(H) \frac{P(F|H)}{P(F|not H)}$$
- H: 강의실을 제대로 찾아갔다
- F: 도착 후 강의실에서 들어온 사람은 여학생이다.
- val
	- $O(H) = \frac{1}{10}$
	- $P(F|H) = \frac{17}{100}$
	- $P(F|not H) = \frac{53}{100}$
	- $\frac{P(F|H)}{P(F|not H)} = 17 / 50$
- 위 val 로 posterior odds 를 계산할 수 있다.
	- $O(H|F) = 10 / 3$
	- $O(H|FF) = 10 / 9$
	- $O(H|FFF) = 10 / 27$
	- 계산을 쉽게 하기 위해 likelihood ratio 는 $\frac{1}{3}$ 이라고 가정한다.
- likelihood ratio 는 일정하지만 확률 변동 정도는 다르다.

![](https://i.imgur.com/a1Gr7V3.png)

odds 에 log 를 취해본다.

![](https://i.imgur.com/6zTFoJV.png)
- 해석
	- prob > 0.5
		- odds > 1 && log odds > 0
	- prob < 0.5
		- odds < 1 && log odds < 0
	- log odds diff 간 차이는 동일하다.

$$\log O(H|F) = \log O(H) + \log \frac{P(F|H)}{P(F|not H)}$$
$$\log O(H|F^x) = \log O(H) + x \log \frac{P(F|H)}{P(F|not H)}$$

$$\log O(H | x) = \beta_0 + \beta_1 x$$
- $\beta_0$ and $\beta_1$ are unknown parameters
	- $\beta_0$ 는 절편으로 x=0 일 때의 log odds val
	- $\beta_1$ 는 기울기로 log of likelihood ratio
- This equation is the basis of logistic regression.

# The Space Shuttle Problem (우주 왕복선 문제)

> "1986년 1월 28일, 미국 우주왕복선 프로그램의 25번째 비행은 이륙 직후 우주왕복선 챌린저호의 로켓 부스터 중 하나가 폭발하여 승무원 7명 전원이 사망하는 재앙으로 끝났습니다. 이 사고에 대한 대통령 위원회는 로켓 부스터의 필드 조인트에 있는 오링이 고장났으며, 이 오링이 외부 온도 등 여러 요인에 민감하지 않게 설계된 결함 때문이라고 결론지었습니다. 이전 24회의 비행 중 23회(1회는 해상에서 분실)의 오링 고장에 대한 데이터를 확보했으며, 이 데이터는 챌린저 발사 전날 저녁에 논의되었지만 안타깝게도 손상 사고가 발생한 7회의 비행에 해당하는 데이터만 중요하다고 간주되어 뚜렷한 추세를 보이지 않는 것으로 생각되었습니다."

```python
data.head() # 발사일 / 외부온도 / 손상사고 여부(0, 1)
         Date  Temperature  Damage
0  04/12/1981           66       0
1  11/12/1981           70       1
2     3/22/82           69       0
data.shape
(23, 3)
```

![](https://i.imgur.com/IHK3MdG.png)

$$\log O(H | x) = \beta_0 + \beta_1 x$$
- H: O 링은 손상되었다.
- x: 온도
- $\beta_0$ and $\beta_1$: parameter we will estimate

```python
offset = data['Temperature'].mean().round()
data['x'] = data['Temperature'] - offset # 평균이 0이 되도록 offset(평균) 을 뺀다.
offset
70.0

# non-Bayesian logistic regression
import statsmodels.formula.api as smf

formula = 'y ~ x'
results = smf.logit(formula, data=data).fit(disp=False)
results.params
Intercept   -1.208490
x           -0.232163
dtype: float64

# 위에서 구한 intercept 와 slope 을 이용해 온도 범위의 확률을 구해보자.
inter = results.params['Intercept']
slope = results.params['x']
xs = np.arange(53, 83) - offset

log_odds = inter + slope * xs
odds = np.exp(log_odds)
ps = odds / (odds + 1)

# 위 3개 라인을 한 번에 수행하는 scipy.special 에 expit 함수가 존재한다.
from scipy.special import expit

ps = expit(inter + slope * xs)
```

![](https://i.imgur.com/HGJr7Jw.png)

베이지안으로 접근해본다.

# Prior Distribution
```python
from utils import make_uniform

# lower/upper 는 앞서 non-bayesian 결과를 바탕으로 설정한다.
qs = np.linspace(-5, 1, num=101)
prior_inter = make_uniform(qs, 'Intercept')

qs = np.linspace(-0.8, 0.1, num=101)
prior_slope = make_uniform(qs, 'Slope')

# 두 param 의 joint dist 를 생성한다.
from utils import make_joint

joint = make_joint(prior_inter, prior_slope)

# prior 를 쌓아 사용하는 것이 편리하기 때문에 이전에 사용했던 stack 함수를 활용한다.
# 반복문 돌리기에 야무지다.
from empiricaldist import Pmf

joint_pmf = Pmf(joint.stack())
joint_pmf.head()
Slope  Intercept  probs
-0.8   -5.00        0.000098
       -4.94        0.000098
       -4.88        0.000098
Name: , dtype: float64
```

# Likelihood (가능도)
```python
# 온도 별 사고 건수를 그룹화 한다.
grouped = data.groupby('x')['y'].agg(['count', 'sum'])
grouped
       count  sum
x                
-17.0      1    1
-13.0      1    1
-12.0      1    1
-7.0       1    1
-4.0       1    0
-3.0       3    0
-2.0       1    0
-1.0       1    0
 0.0       4    2
 2.0       1    0
 3.0       1    0
 5.0       2    1
 6.0       2    0
 8.0       1    0
 9.0       1    0
 11.0      1    0

# "이항 분포의 매개변수와 동일성을 유지하기 위해 각 변수명을 ns, ks 로 지정한다" 라는 말이 무슨 말인지 모르겠어요. 중요한것 같진 않지만 다들 어떻게 받아들이셨는지 궁금해요.
ns = grouped['count']
ks = grouped['sum']

# 앞서 non-bayesian regression 으로 추정한 값이 정확하다고 가정한다.
xs = grouped.index
ps = expit(inter + slope * xs)

from scipy.stats import binom

# binom 으로 likelihood 를 계산한다.
likes = binom.pmf(ks, ns, ps)
likes
array([0.93924781, 0.85931657, 0.82884484, 0.60268105, 0.56950687,
       0.24446388, 0.67790595, 0.72637895, 0.18815003, 0.8419509 ,
       0.87045398, 0.15645171, 0.86667894, 0.95545945, 0.96435859,
       0.97729671])

# likes 각 element 는 손상확률이 p 인 n회 발사에서 k회 손상일 일어날 확률이다. 전체 데이터의 likelihood 는 배열의 내적 (내적의 결과는 스칼라)
likes.prod()
0.0004653644508250066

# 가능한 모든 쌍의 likelihood 를 계산한다.
likelihood = joint_pmf.copy()
for slope, inter in joint_pmf.index:
    ps = expit(inter + slope * xs)
    likes = binom.pmf(ks, ns, ps)
    likelihood[slope, inter] = likes.prod() 
```

# The Update (갱신)
```python
posterior_pmf = joint_pmf * likelihood
posterior_pmf.normalize()
```

![](https://i.imgur.com/XvZFsYC.png)

등고선 모양이 대각선 축 기준 -> slope & inter 간 correlation 이 있다. 단, 평균을 0으로 맞추기 위해 x 에 mean 값을 차감했기 때문에 조금은 약할 것이다.

# Marginal Distributions (주변분포)
```python
from utils import marginal

marginal_inter = marginal(joint_posterior, 0)
marginal_slope = marginal(joint_posterior, 1)
```

![](https://i.imgur.com/jXuaBPD.png)

![](https://i.imgur.com/aW1FUjs.png)

# Transforming Distributions (분포 변환)
- intercept: 온도가 70도(*offset*)인 경우 가설(*O링은 손상되었다*) 의 log odds
- 따라서 intercept 주변분포인 marginal_inter 로 log odds 로 해석할 수 있다.

```python
def transform(pmf, func):
    """Transform the quantities in a Pmf."""
    ps = pmf.ps
    qs = func(pmf.qs)
    return Pmf(ps, qs, copy=True)

# log odds 값을 확률로 변환 후 확률 형태의 intercept posterior 를 생성한다.
marginal_probs = transform(marginal_inter, expit)
# == marginal_inter.transform(expit)
```

![](https://i.imgur.com/rFYRkZy.png)

- 화씨 70도에서의 손상 확률의 평균값은 약 22%
	```python
	marginal_probs.mean()
	0.2201937884647988
	```
- x 를 0으로 맞춘 두번째 이유를 설명하는데 잘 이해가 안된다. 손상 확률에 더 잘 대응된다고 하는데..
- slope 의 경우 likelihood ratio 로 변경할 수 있다.
```python
marginal_lr = marginal_slope.transform(np.exp)
marginal_lr.mean()
0.7542914170110268
```

![](https://i.imgur.com/DhZJgvi.png)

```python
expit(marginal_inter.mean()), marginal_probs.mean()
(0.2016349762400815, 0.2201937884647988)

np.exp(marginal_slope.mean()), marginal_lr.mean()
(0.7484167954660071, 0.7542914170110268)

# 큰 차이는 없지만 먼저 log 를 취한 후 summary 하는 것이 rule 이라고 한다.
```

# Predictive Distributions (예측 분포)
**바깥 온도가 화씨 31일 때 O링 손상 확률은 얼마인가?**

```python
sample = posterior_pmf.choice(101)

temps = np.arange(31, 83) # 31: 챌린저호 발사 당시 온도, 83: 관측치 최대 온도
xs = temps - offset

pred = np.empty((len(sample), len(xs)))

for i, (slope, inter) in enumerate(sample):
    pred[i] = expit(inter + slope * xs)
```

![](https://i.imgur.com/X356Raq.png)

```python
low, median, high = np.percentile(pred, [5, 50, 95], axis=0)
```

![](https://i.imgur.com/pQKU8VF.png)

```python
low = pd.Series(low, temps)
median = pd.Series(median, temps)
high = pd.Series(high, temps)

t = 80
print(median[t], (low[t], high[t]))
0.019646769947688696 (0.00030042858873038816, 0.14063812573413423)

t = 60
print(median[t], (low[t], high[t]))
0.826353352980995 (0.4925005624493795, 0.9776865613009676)

t = 31
print(median[t], (low[t], high[t]))
0.9999564692743173 (0.95787193726686, 0.9999999971387292)
```

- 만약 책임자들이 사고 데이터만이 모든 데이터를 고려했다면 비행을 연기할 수 있었을 것이다.
- 데이터 범위를 훨씬 넘어선 범위의 예측값도 고려하여 결과의 불확실성도 염두해 두어야한다.
