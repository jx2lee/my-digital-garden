---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter05-Estimating-Counts/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

이 장에서는 인구 수를 세거나 인구의 크기를 추정하는 것과 관련된 문제를 다룬다. 다시 말하지만, 일부 예제는 우스꽝스러워 보일 수 있지만 독일 탱크 문제와 같은 일부 예제는 때로는 삶과 죽음의 상황에서 실제 적용이 가능하다.

# The Train Problem
> "철도는 기관차에 1...N 순서로 번호를 매깁니다. 어느 날 번호가 60인 기관차를 보게 됩니다. 철도에 몇 대의 기관차가 있는지 추정하세요. (goal: N)"

- 이 관찰을 바탕으로 우리는 철도에 60개 이상의 기관차가 있다는 것을 알고 있다.
- 하지만 몇 대가 더 있을까? 베이지안 추론을 적용하기 위해 이 문제를 두 단계로 나눌 수 있습니다.
	- 데이터를 보기 전에 N 에 대해 무엇을 알고 있었습니까?
	- 주어진 N 값에 대해 데이터(60번 기관차)를 볼 확률은 얼마인가?  
- **첫 번째 질문에 대한 답은 prior 이다.**
- **두 번째 질문에 대한 답은 likelihood 이다.**
  
prior 을 선택할 근거가 많지 않으므로 간단한 것부터 시작한 다음 대안을 고려해본다. N 이 1에서 1000 사이의 모든 값일 가능성이 똑같다고 가정해보자.
  
사전 분포는 다음과 같다.

```python
import numpy as np
from empiricaldist import Pmf

hypos = np.arange(1, 1001)
prior = Pmf(1, hypos)
```

이제 likelihood 을 알아보자. 𝑁 기관차로 구성된 차량에서 60번 기관차를 볼 확률은 얼마인가? 모든 기관차를 볼 확률이 똑같다고 가정하면 특정 기관차를 볼 확률은 1/𝑁 이다.

다음은 업데이트를 수행하는 함수이다.

```python
def update_train(pmf, data):
    """Update pmf based on new data."""
    hypos = pmf.qs
    likelihood = 1 / hypos
    impossible = (data > hypos)
    likelihood[impossible] = 0
    pmf *= likelihood
    pmf.normalize()
```

이 함수는 이전 장의 주사위 문제의 업데이트 함수와 동일하다. **likelihood 관점에서 주사위 문제와 비슷하다.**

업데이트 해보자
```python
data = 60
posterior = prior.copy()
update_train(posterior, data)
```

![](https://i.imgur.com/yRHXuQz.png)
당연히 60 미만의 𝑁 값은 모두 제거된다. 가장 높은 pmf 값은 60 이다.

```python
>>> posterior.max_prob()
60
```

가장 높은 숫자를 가진 열차를 우연히 봤을 확률이 얼마나 될까? 그럼에도 확률을 최대화하려면 60 을 맞혀야 한다.

다른 대안으로 **사후 분포의 평균을 계산**하는 방법이 있다. 가능한 수량 집합 𝑞𝑖 와 그 확률 𝑝𝑖 이 주어지면 분포의 평균은 다음과 같습니다.

$$\mathrm{mean} = \sum_i p_i q_i$$

```python
>>> np.sum(posterior.ps * posterior.qs)
333.41989326370776
>>> posterior.mean()
333.41989326370776
```

즉, 333대 열차가 있을것으로 기대할 수 있다.
사후 평균을 추정치로 사용하면 장기적으로 평균 제곱 오차(MSE, mean squared error)를 최소화할 수 있다.

# Sensitivity to the Prior
이전 섹션에서 사용한 사전은 1에서 1000까지 균일하지만, 균일 분포 또는 특정 상한을 선택하는 것에 대한 정당성을 제시하지 않았다. 사후분포가 선행 분포에 민감한지 궁금할 수 있다. **데이터가 너무 적고 관측값이 하나뿐인 경우라면 그렇다**.

```python
import pandas as pd

df = pd.DataFrame(columns=['Posterior mean'])
df.index.name = 'Upper bound'

for high in [500, 1000, 2000]:
    hypos = np.arange(1, high+1)
    pmf = Pmf(1, hypos)
    update_train(pmf, data=60)
    df.loc[high] = pmf.mean()
    
df
```

![](https://i.imgur.com/oGPrTi5.png)

상한(upper bound)을 변경하면 사후 평균이 크게 달라지는데 이는 좋지 않다. 사후가 사전에 민감한 경우 두 가지 방법으로 진행할 수 있다.

- ==더 많은 데이터를 확보한다.==
- ==더 많은 배경 정보를 얻고 더 나은 prior 을 선택한다.==

**데이터가 많을수록 prior 에 기반한 posterior 분포가 수렴하는 경향이 있다.** 예를 들어 열차 60 외 30과 90번 열차가 있다고 가정해보자. 사후확률은 다음과 같이 계산할 수 있다.

$$p(D|H_{60}) * p(D|H_{30}) *p(D|H_{90})$$

```python
df = pd.DataFrame(columns=['Posterior mean'])
df.index.name = 'Upper bound'

dataset = [30, 60, 90]

for high in [500, 1000, 2000]:
    hypos = np.arange(1, high+1)
    pmf = Pmf(1, hypos)
    for data in dataset:
        update_train(pmf, data)
    df.loc[high] = pmf.mean()
    
df
```

![](https://i.imgur.com/593kIu5.png)

상한에 따라 차이는 더 줄었다. 분명히 세 개의 열차로는 posterior 에 수렴하기 충분하지 않다.

# Power Law Prior
더 많은 데이터를 사용할 수 없는 경우 다른 옵션은 더 많은 배경 정보를 수집하여 prior 을 개선하는 것이다. 기관차가 1000대인 기차 운영회사와 1대만 있는 회사가 같은 확률이라고 가정하는 것은 합리적이지 않다.

- 조금만 노력하면 관찰 대상 지역에서 기관차를 운영하는 회사 목록을 찾을 수 있을 것이다. 또는 철도 운송 전문가를 인터뷰하여 회사의 일반적인 규모에 대한 정보를 수집할 수도 있다.
- 하지만 철도 경제학에 대해 자세히 알아보지 않더라도 몇 가지 추측을 할 수 있다. 대부분의 분야에는 소기업이 많고 중기업은 적으며 대기업은 한두 개에 불과하다.
- 실제로 로버트 액스텔이 사이언스(http://www.sciencemag.org/content/293/5536/1818.full.pdf)에서 보고한 것처럼 기업 규모의 분포는 **멱 법칙**을 따르는 경향이 있다.

이 법칙에 따르면 기관차가 10개 미만인 회사가 1000개라면 기관차가 100개인 회사는 100개, 1000개인 회사는 10개, 1만 개의 기관차를 보유한 회사는 1개가 있을 수 있다.

수학적으로 power law 는 주어진 규모인 𝑁를 가진 회사의 수가 $(1/N)^{\alpha}$에 비례한다는 것을 의미하며, 여기서 𝛼 는 1에 가까운 수다.

$$PMF(x) \propto (\frac{1}{x})^\alpha$$

이와 같은 식으로 앞서 power law prior 를 구성해보자.

```python
alpha = 1.0
ps = hypos**(-alpha)
power = Pmf(ps, hypos, name='power law')
power.normalize()
```

비교를 위해 사전분포(uniform)은 다음과 같다.

```python
hypos = np.arange(1, 1001)
uniform = Pmf(1, hypos, name='uniform')
uniform.normalize()
```

![](https://i.imgur.com/SCiM2uU.png)
![](https://i.imgur.com/U6ZeSWc.png)

power law 는 높은 값에 대한 사전 확률을 낮추기 때문에 사후 평균과 상한의 민감도가 낮아졌다.  

power law prior 을 사용하여 세 개의 열차를 관찰할 때 사후 평균이 상한(upper bound)에 따라 어떻게 달라지는지 살펴보자.

```python
df = pd.DataFrame(columns=['Posterior mean'])
df.index.name = 'Upper bound'

alpha = 1.0
dataset = [30, 60, 90]

for high in [500, 1000, 2000]:
    hypos = np.arange(1, high+1)
    ps = hypos**(-alpha)
    power = Pmf(ps, hypos)
    for data in dataset:
        update_train(power, data)
    df.loc[high] = power.mean()
    
df
```

![](https://i.imgur.com/tdOngb6.png)

이제 그 차이가 훨씬 작아졌다. 실제로 상한을 위 숫자보다 크게 설정해도 평균은 134에 수렴한다.

# Credible Intervals
지금까지 사후 분포를 요약하는 두 가지 방법, 즉 사후 확률이 가장 높은 값(MAP)과 사후 평균(posterior mean)을 살펴봤다. 이 두 가지 모두 점 추정(point estimates), 즉 우리가 관심 있는 수량을 추정하는 단일 값이다.
  
사후 분포를 요약하는 또 다른 방법은 [백분위수](https://en.wikipedia.org/wiki/Percentile_rank)를 사용하는 것이다. 표준화 시험(like 수능)을 치른 적이 있다면 백분위수에 익숙할 것이다. 예를 들어, 백분위수가 90이라면 시험을 본 사람의 90%와 비슷하거나 그보다 더 잘했다는 의미다.
  
x라는 값이 주어지면 x보다 작거나 같은 모든 값을 찾아서 확률을 더하면 백분위수 순위를 계산할 수 있다.
  
Pmf는 이 계산을 수행하는 메서드를 제공한다. 예를 들어, 회사의 열차 수가 100대 미만일 확률을 계산할 수 있습니다.

```python
>>> power.prob_le(100)
0.2937469222495771
```

앞서 power law 와 3개 열차 데이터 집합(30, 60, 90)을 사용하면 약 29% 이다. 따라서 100개의 열차는 29번째 백분위수다.
  
반대로 특정 백분위수를 계산하고 싶다고 하자. 예를 들어 분포의 중앙값이 50번째 백분위수라고 가정한다. 총합이 0.5를 초과할 때까지 확률을 더하여 계산할 수 있다.

```python
def quantile(pmf, prob):
    """Compute a quantile with the given prob."""
    total = 0
    for q, p in pmf.items():
        total += p
        if total >= prob:
            return q
    return np.nan
```

- 반복문은 분포의 수량과 확률을 반복하는 항목을 사용한다.
	- 루프 내부에서 수량의 확률을 순서대로 합산한다.
	- 합계가 확률과 같거나 초과하면 해당 수량을 반환한다.
- 이 함수는 백분위수가 아닌 사분위수를 계산하기 때문에 사분위수(quantile)라고 한다.
	- 차이점은 확률을 지정하는 방식
	- prob가 0에서 100 사이의 백분율인 경우 해당 수량을 백분위수(percentile)라고 부른다.
	- 확률이 0과 1 사이인 경우 해당 수량을 사분위수(qunatile)라고 한다.

50 백분위수를 구하는 방법은 다음과 같다.

```python
>>> quantile(power, 0.5)
113
```

113개 열차가 posterior 분포의 중앙값이다. Pmf는 동일한 작업을 수행하는 사분위수라는 메서드를 제공한다. 다음과 같이 호출하여 5 & 95번째 백분위수를 계산하면 다음과 같다.

```python
>>> power.quantile([0.05, 0.95])
array([ 91., 243.])
```
- 해석은 다음과 같이 할 수 있다.
	- 열차 수가 91보다 작거나 같을 확률은 5% 이다.
	- 열차 수가 243보다 클 확률은 5% 이다.
- 따라서 열차 수가 91에서 243 사이(91 < <=243)에 속할 확률은 90% 이다. **이 간격을 90% 신뢰 구간이라고 한다.**

또한 Pmf 는 주어진 확률을 포함하는 간격(interval)을 계산하는 credible_interval을 제공한다.

```python
>>> power.credible_interval(0.9)
array([ 91., 243.])
```

# The German Tank Problem
제2차 세계대전 중 런던 주재 미국 대사관 경제전과는 통계 분석을 통해 독일의 탱크 및 기타 장비 생산량을 추정했다.

서방 연합군은 개별 탱크의 섀시 및 엔진 일련 번호가 포함된 일지, 재고, 수리 기록을 확보했다.

이 기록을 분석한 결과 일련번호는 제조업체와 전차 유형별로 100개씩 블록으로 할당되었고, 각 블록의 번호는 순차적으로 사용되었으며, 각 블록의 모든 번호가 사용된 것은 아니라는 사실을 발견했다. 따라서 독일 전차 생산량을 추정하는 문제는 100개의 번호 블록 내에서 위에 풀어본 기차 문제의 형태로 바라볼 수 있었다.

이를 바탕으로 미국과 영국의 분석가들은 다른 형태의 정보에 의한 추정치보다 훨씬 낮은 추정치를 산출했다. 그리고 전쟁이 끝난 후, 이 추정치가 훨씬 더 정확했다는 기록이 나왔습니다.

이들은 타이어, 트럭, 로켓 및 기타 장비에 대해서도 유사한 분석을 수행하여 정확하고 실행 가능한 경제 정보를 산출해냈다.

독일 탱크 문제는 역사적으로도 흥미롭지만, 통계적 추정을 실제로 적용한 좋은 예이기도 합니다.

이 문제에 대한 자세한 내용은 [위키백과 페이지](https://en.wikipedia.org/wiki/German_tank_problem)와 러글스와 브로디, "[제2차 세계대전의 경제 정보에 대한 경험적 접근](https://web.archive.org/web/20170123132042/https://www.cia.gov/library/readingroom/docs/CIA-RDP79R01001A001300010013-3.pdf)"를 참조하길 바란다.

# Informative Priors
베이지안들 사이에서는 사전 분포를 선택하는 데 두 가지 접근 방식이 있다.

- 문제에 대한 배경 정보를 가장 잘 나타내는 prior 를 선택할 것을 권장하며, 이를 정보적(informative)이라고 한다.
	- 문제점은 사람들이 다른 정보를 가지고 있거나 다르게 해석할 수 있다는 것이다. 따라서 정보적 선행은 자의적(arbitrary)으로 보일 수 있다.
- 이와 반대로 비정보적 prior(uninformative prior) 을 사용할 수 있는데, 이는 가능한 한 제한을 두지 않고 데이터가 **스스로 말하도록 한다**. 경우에 따라 예상 수량에 대한 최소한의 prior 정보를 나타내는 등 몇 가지 속성을 가진 고유 prior 를 식별할 수 있다.

정보가 없는 prior 정보는 더 객관적으로 보이기 때문에 매력적이다. 하지만 저자는 일반적으로 infomrative prior 를 선호한다. 왜? 첫째, **베이지안 분석은 항상 모델링을 기반**으로 한다. prior 을 선택하는 것은 이러한 결정 중 하나이지만 유일한 결정은 아니며 가장 주관적일 수도 있다. 따라서 정보가 없는 prior 이 더 객관적이라 하더라도 전체 분석은 여전히 주관적일 수 있다.

또한 대부분의 실제 문제에서는 데이터가 많거나 많지 않은 두 가지 상황 중 하나에 처할 가능성이 높다. 데이터가 많은 경우, informative 나 uninformative 나 동일한 결과를 내므로 prior 선택은 중요하지 않다. 데이터가 많지 않은 경우 관련 배경 정보(예: power law distribution)를 이용하면 차이를 분명히 알 수 있다.

그리고 독일 탱크 문제에서처럼 결과에 따라 생사를 결정해야 하는 상황이라면, 적게 아는 척하면서 객관성이라는 환상을 유지하기보다는 **모든 정보를 최대한 활용**해야 한다.

# Summary
이 장에서는 주사위 문제와 같은 확률 함수를 가지고 있으며 독일 탱크 문제에 적용할 수 있는 기차 문제를 소개했다. 예제의 목표는 개체 수(estimate a count) 또는 모집단의 크기를 추정한다.
