---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter08-Poisson-Processes/","noteIcon":""}
---

#think-bayes #probability #study

---

이번 장에서는 랜덤 간격(random interval)으로 발생하는 이벤트를 설명하는 데 사용되는 모델인 [포아송 프로세스]()를 소개한다. 푸아송 과정의 예로, 우리가 흔히 '풋볼'이라고 부르는 미국식 영어인 축구의 골 득점을 모델링한다. 게임에서 득점한 골을 사용해 푸아송 과정의 매개 변수를 추정한 다음, posterior 분포를 사용하여 예측한다.
  
그리고 월드컵 문제를 풀어본다.

# The World Cup Problem
2018 FIFA 월드컵 결승전에서 프랑스는 크로아티아를 4골 2대 2로 꺾었다.

- 프랑스가 더 나은 팀이라고 얼마나 확신해야 할까요?
- 크로아티아와 다시 경기를 한다면 프랑스가 다시 우승할 확률은 얼마나 될까요?

이러한 질문에 답하기 위해 모델링을 결정해야한다.

- 먼저, 어떤 팀과 다른 팀이 경기 시 경기당 골로 측정되는 골 득점률이 가정하고, 이를 파이썬 변수 lam 또는 그리스 문자 𝜆("람다"로 발음)로 표시한다.
- 둘째, 골은 경기 어느 순간에도 똑같이 발생할 가능성이 있다고 가정한다. 따라서 90분짜리 게임에서 어느 한 분 동안 득점할 확률은 𝜆/90 입니다.
- 셋째, 한 팀이 같은 1분 동안 두 번 득점하지 않는다고 가정한다.

물론 이러한 가정 중 어느 것도 현실 세계에서 완전히 맞진 않는다. 조지 박스가 말했듯이 "모든 모델은 틀렸지만 일부는 유용합니다." (https://en.wikipedia.org/wiki/All_models_are_wrong).

위 경우 이러한 가정이 사실이라면 적어도 대략적으로나마 게임에서 득점한 골의 수가 푸아송 분포를 따르기 때문이다.

# The Poisson Distribution
게임에서 득점한 골 수가 골 득점률 𝜆 인 포아송 분포를 따르는 경우, 𝑘 골을 득점할 확률은 다음과 같다.
(음수가 아닌 모든 값에 대해 𝑘)

$$\lambda^k \exp(-\lambda) ~/~ k!$$

SciPy 는 푸아송 분포를 나타내는 푸아송 객체를 제공한다. 다음과 같이 𝜆=1.4로 푸아송 객체를 만들 수 있다.

```python
>>> from scipy.stats import poisson
>>> 
>>> lam = 1.4
>>> dist = poisson(lam)
>>> type(dist)
<class 'scipy.stats._distn_infrastructure.rv_discrete_frozen'>
```

"frozen" 무작위 변수를 나타내는 객체가 생성되고 포아송 분포의 pmf를 제공한다.

```python
>>> k = 4
>>> dist.pmf(k)
0.039471954028253146
```

- 경기당 평균 득점률이 1.4골이라면 한 경기에서 4골을 넣을 확률은 약 4%라는 것을 의미한다.

다음 함수를 사용하여 푸아송 분포를 나타내는 Pmf 를 생성해보자.
```python
from empiricaldist import Pmf

def make_poisson_pmf(lam, qs):
    """Make a Pmf of a Poisson distribution."""
    ps = poisson(lam).pmf(qs)
    pmf = Pmf(ps, qs)
    pmf.normalize()
    return pmf
```

- 포아송 PMF를 평가하는 골 득점률(lam)과 수량(qs)을 매개변수로 받는다
- PMF 객체를 반환한다.

예를 들어, 다음은 0에서 9까지의 k 값에 대해 계산된 lam=1.4에 대한 골 득점 분포는 다음과 같다.
```python
>>> import numpy as np
>>> 
>>> lam = 1.4
>>> goals = np.arange(10)
>>> pmf_goals = make_poisson_pmf(lam, goals)
```

![](https://i.imgur.com/QSZfj2F.png)

- 가장 가능성이 높은 스코어는 0, 1, 2이며, 이보다 높은 값도 가능하지만 가능성은 점점 낮아진다. (7 이상의 값은 무시가능)
- 분포는 골 득점률(람다)을 알면 골 수를 예측할 수 있음을 보여준다.
  
반대로 골 수가 주어지면 골 득점률에 대해 무엇을 말할 수 있을까?
이 질문에 답하기 위해서는 점수를 보기 전에 가능한 범위와 그 확률을 나타내는 람다(lam)의 사전 분포에 대해 생각해 볼 필요가 있다.

# The Gamma Distribution
축구 경기를 본 적이 있다면 람다에 대한 정보를 알고 있다. 대부분의 게임에서 팀은 몇 골씩 득점한다. 드물게 한 팀이 5골 이상을 득점하는 경우도 있지만 10골 이상을 득점하는 경우는 거의 없다.

이전 [월드컵 데이터](https://www.statista.com/statistics/269031/goals-scored-per-game-at-the-fifa-world-cup-since-1930/)를 사용하여 각 팀이 경기당 평균적으로 약 1.4골을 득점하는 것으로 추정한다. 따라서 람다의 평균을 1.4로 설정할 것이다. 못하는 팀이 잘하는 팀과의 경기에서는 실점이 더 많을 것으로 예상하고, 잘하는 팀이 못하는 팀과의 경기에서 실점이 더 적을 것으로 예상한다.

골 득점률의 분포를 모델링하기 위해 [감마 분포](https://en.wikipedia.org/wiki/Gamma_distribution)를 사용할 거이다.
- 골 득점률은 연속적이고 음수가 아니며, 감마 분포가 이런 종류에 적합하다.
- 감마 분포에는 평균을 나타내는 알파라는 매개 변수 하나만 있다. 따라서 원하는 평균을 가진 감마 분포를 쉽게 만들 수 있다.
- 감마 분포의 형태는 우리가 축구에 대해 알고 있는 것을 고려할 때 합리적인 선택일 것이다.

SciPy는 감마 분포를 나타내는 객체를 생성하는 감마를 제공한다. 그리고 감마 객체는 감마 분포의 확률 밀도 함수 pdf를 제공한다.
```python
>>> from scipy.stats import gamma
>>> 
>>> alpha = 1.4
>>> qs = np.linspace(0, 10, 101)
>>> ps = gamma(alpha).pdf(qs)
```
- 매개 변수인 알파는 분포의 평균이다.
- q는 0에서 10 사이의 가능한 lam 값이다.
- ps는 정규화되지 않은 확률로 생각할 수 있는 확률 밀도(probability densities)이다.

이를 정규화하려면 Pmf에 넣고 `normalize`를 호출한다.
```python
>>> from empiricaldist import Pmf
>>> 
>>> prior = Pmf(ps, qs)
>>> prior.normalize()
```

![](https://i.imgur.com/8XKpQTG.png)

- 골 득점에 대한 prior 정보을 나타낸다.
- 램은 보통 2 미만, 가끔 6까지 높지만 그 이상은 거의 없다.

그리고 평균이 약 1.4라는 것을 확인할 수 있다.
```python
>>> prior.mean()
1.4140818156118378
```

다음은 업데이트를 해본다.

# The Update
골 득점률 𝜆 이 주어지고 여러 골을 넣을 확률 𝑘 을 계산해 달라는 요청을 받았다고 가정해보자.
예를 들어 𝜆가 1.4이면 한 경기에서 4골을 넣을 확률은 다음과 같다.
```python
>>> lam = 1.4
>>> k = 4
>>> poisson(lam).pmf(4)
0.039471954028253146
```

이제 𝜆에 대해 값의 배열이 있다고 가정하면, 다음과 같이 각 가상 값 lam에 대한 likelihood 을 계산할 수 있다.
```python
>>> lams = prior.qs
>>> k = 4
>>> likelihood = poisson(lams).pmf(k)
>>> likelihood
array([0.00000000e+00, 3.77015591e-06, 5.45820502e-05, 2.50026149e-04,
       7.15008049e-04, 1.57950693e-03, 2.96358283e-03, 4.96792214e-03,
       7.66854765e-03, 1.11145981e-02, 1.53283100e-02, 2.03065231e-02,
       2.60231799e-02, 3.24324189e-02, 3.94719540e-02, 4.70665182e-02,
       5.51312092e-02, 6.35746276e-02, 7.23017337e-02, 8.12163834e-02,
       9.02235222e-02, 9.92310359e-02, 1.08151269e-01, 1.16902230e-01,
       1.25408499e-01, 1.33601886e-01, 1.41421844e-01, 1.48815687e-01,
       1.55738624e-01, 1.62153659e-01, 1.68031356e-01, 1.73349519e-01,
       1.78092787e-01, 1.82252178e-01, 1.85824592e-01, 1.88812285e-01,
       1.91222339e-01, 1.93066121e-01, 1.94358757e-01, 1.95118616e-01,
       1.95366815e-01, 1.95126749e-01, 1.94423652e-01, 1.93284180e-01,
       1.91736036e-01, 1.89807621e-01, 1.87527718e-01, 1.84925212e-01,
       1.82028837e-01, 1.78866956e-01, 1.75467370e-01, 1.71857150e-01,
       1.68062503e-01, 1.64108654e-01, 1.60019753e-01, 1.55818804e-01,
       1.51527608e-01, 1.47166728e-01, 1.42755463e-01, 1.38311841e-01,
       1.33852618e-01, 1.29393291e-01, 1.24948121e-01, 1.20530156e-01,
       1.16151272e-01, 1.11822205e-01, 1.07552602e-01, 1.03351061e-01,
       9.92251884e-02, 9.51816428e-02, 9.12261916e-02, 8.73637629e-02,
       8.35984979e-02, 7.99338039e-02, 7.63724057e-02, 7.29163965e-02,
       6.95672867e-02, 6.63260522e-02, 6.31931800e-02, 6.01687122e-02,
       5.72522885e-02, 5.44431858e-02, 5.17403564e-02, 4.91424640e-02,
       4.66479169e-02, 4.42549001e-02, 4.19614048e-02, 3.97652554e-02,
       3.76641357e-02, 3.56556121e-02, 3.37371552e-02, 3.19061603e-02,
       3.01599651e-02, 2.84958667e-02, 2.69111367e-02, 2.54030347e-02,
       2.39688206e-02, 2.26057656e-02, 2.13111624e-02, 2.00823331e-02,
       1.89166374e-02])
```

posterior 를 구하려면 prior 에 방금 계산한 확률을 곱하고 결과를 정규화하면 된다! 이를 캡슐화한 함수를 살펴보자.
```python
def update_poisson(pmf, data):
    """Update Pmf with a Poisson likelihood."""
    k = data
    lams = pmf.qs
    likelihood = poisson(lams).pmf(k)
    pmf *= likelihood
    pmf.normalize()
```

이 예제에서는 프랑스가 4골을 넣었으므로 이전 데이터를 복사하여 업데이트 한다.
```python
>>> france = prior.copy()
>>> update_poisson(france, 4)
>>> france
0.0     0.000000
0.1     0.000003
0.2     0.000053
0.3     0.000260
0.4     0.000755
          ...   
9.6     0.000009
9.7     0.000008
9.8     0.000007
9.9     0.000006
10.0    0.000005
Name: , Length: 101, dtype: float64
```

![](https://i.imgur.com/QDzLJHO.png)

- k=4 라는 값(프랑스가 이전에 4골을 넣은 데이터) lam의 값이 높을수록 가능성이 높고 낮을수록 가능성이 낮다고 생각하게 만든다.
- 따라서 posterior 분포가 오른쪽으로 이동한다.

마찬가지로 크로아티아 posterior 분포를 구해본다.
```python
>>> croatia = prior.copy()
>>> update_poisson(croatia, 2)
>>> croatia
0.0     0.000000e+00
0.1     1.154111e-03
0.2     4.987243e-03
0.3     1.080490e-02
0.4     1.764472e-02
            ...     
9.6     3.699161e-07
9.7     3.104885e-07
9.8     2.605416e-07
9.9     2.185748e-07
10.0    1.833229e-07
Name: , Length: 101, dtype: float64
```

![](https://i.imgur.com/HQGGyFK.png)

프랑스와 크로아티아의 posterior 분포 평균을 구하면 다음과 같다.
```python
>>> print(croatia.mean(), france.mean())
1.6999765866755225 2.699772393342308
```

- posterior 분포의 평균은 약 1.4 이다.
- 크로아티아가 2골을 넣은 후의 posterior 평균은 1.7로, prior 분포와 데이터의 중간값에 가까워 진다.
- 마찬가지로 프랑스가 4골을 넣은 후의 posterior 평균은 2.7 이다.
  
이러한 결과는 베이지안 업데이트의 전형적인 결과로, posterior 분포 위치는 prior 분포와 데이터의 절충점이다. (compromise)

# Probability of Superiority
이제 각 팀에 대한 posterior 분포를 알았으니 첫 번째 질문에 답할 수 있다
- 프랑스가 더 나은 팀이라고 얼마나 확신할 수 있을까요?

이 모델에서 "더 나은"이란 상대팀에 대해 더 높은 골 득점률을 갖는 것을 의미한다. posterior 분포를 사용하여 프랑스의 분포에 무작위로 추출한 값이 크로아티아의 분포에서 추출한 값을 초과할 확률을 계산할 수 있다.

이를 수행하는 한 가지 방법은 두 분포의 모든 값 쌍을 열거하고 한 값이 다른 값을 초과할 총 확률을 더하는 것이다.

```python
def prob_gt(pmf1, pmf2):
    """Compute the probability of superiority."""
    total = 0
    for q1, p1 in pmf1.items():
        for q2, p2 in pmf2.items():
            if q1 > q2:
                total += p1 * p2
    return total

>>> prob_gt(france, croatia)
0.7499366290930155
```

Pmf 는 동일하게 `prob_gt` 함수를 제공한다.
```python
>>> Pmf.prob_gt(france, croatia)
0.7499366290930174
```

(`Pmf.prob_gt`는 루프가 아닌 배열 연산자를 사용하기 때문에 결과가 약간 다르다고 한다.)

어느 쪽이든 결과는 75% 에 가깝다. 따라서 한 경기를 기준으로 볼 때 프랑스가 실제로 더 나은 팀이라고 어느 정도 확신할 수 있다.

물론 이 결과는 골 득점률이 일정하다는 가정을 전제로 한 결과라는 점을 기억해야 한다. 실제로는 한 골 차로 뒤지고 있는 팀이 경기 막판으로 갈수록 더 공격적으로 플레이하여 득점할 가능성도 높아지지만 추가 골을 내줄 가능성도 높아질 수 있다.

# Predicting the Rematch
같은 팀이 다시 경기를 한다면 크로아티아가 이길 확률은 얼마나 될까?
이 질문에 답하기 위해 한 팀이 득점할 것으로 예상되는 골 수인 'posterior predictive 분포'를 생성한다.

골 득점률 lam을 알고 있다면, 골 분포는 매개 변수 lam 을 가지는 포아송 분포가 될 것이다. lam 을 모르기 때문에 골 분포는 다양한 값의 lam을 가진 푸아송 분포가 혼합된(mixture) 형태이다.

먼저 lam의 각 값에 대해 하나씩 Pmf 객체 시퀀스를 생성한다.
```python
>>> pmf_seq = [make_poisson_pmf(lam, goals) 
	           for lam in prior.qs]
```

![](https://i.imgur.com/xJnjVeY.png)

predictive 분포는 이러한 Pmf 객체들의 mixture 로, posterior 확률로 가중치가 부여됩니다. mixture 계산을 위해 make_mixture를 이용할 것이다.
```python
>>> from utils import make_mixture
>>> 
>>> pred_france = make_mixture(france, pmf_seq)
>>> pred_france
0    0.112030
1    0.201777
2    0.215467
3    0.177493
4    0.124612
5    0.078434
6    0.045603
7    0.024969
8    0.013047
9    0.006568
```

![](https://i.imgur.com/RJZ23lK.png)
이 분포는 두 가지 불확실성의 원천을 나타낸다.
- lam 의 실제 값을 알 수 없고
- 알더라도 다음 경기의 골 수를 알 수 없다는 없다.
  
크로아티아의 predictive dist 는 다음과 같다.
```python
>>> pred_croatia = make_mixture(croatia, pmf_seq)
>>> pred_croatia
0    0.251958
1    0.285581
2    0.209489
3    0.125770
4    0.067152
5    0.033190
6    0.015534
7    0.006984
8    0.003045
9    0.001297
```

![](https://i.imgur.com/Oh1s9Xl.png)

이러한 분포를 사용하여 프랑스가 재경기에서 승리, 패배 또는 무승부를 거둘 확률을 계산할 수 있다.
```python
>>> win = Pmf.prob_gt(pred_france, pred_croatia)
>>> win
0.5703522415934519
>>> lose = Pmf.prob_lt(pred_france, pred_croatia)
>>> lose
0.26443376257235873
>>> tie = Pmf.prob_eq(pred_france, pred_croatia)
>>> tie
0.16521399583418947
```

프랑스가 동점 상황에서 절반을 이긴다고 가정할 때, 프랑스가 재경기에서 승리할 확률은 약 65% 이다.
```python
>>> win + tie/2
0.6529592395105466
```

이는 75% 인 확률보다 약간 낮은 수치다. 골 득점률보다 한 경기의 결과에 대한 확신이 낮기 때문에 이는 당연한 결과다. 프랑스가 더 나은 팀이라고 해도 경기에서 질 수도 있다.

# The Exponential Distribution
이 노트의 마지막에 있는 연습 문제에서는 월드컵 문제를 변형한 다음 문제를 풀어볼 수 있다.

> 2014 FIFA 월드컵에서 독일은 브라질과 준결승전을 치렀습니다. 독일은 11분 후와 23분에 다시 한 번 득점했습니다. 이 경기에서 독일이 90분 후에 몇 골을 넣을 것으로 예상하십니까? 독일이 5골을 더 넣을 확률은 얼마였나요(실제로 그렇게 되었음)?

이 버전에서는 데이터가 정해진 기간 동안의 골 수가 아니라 골 사이 시간이라는 점에 유의해야한다.

이와 같은 데이터의 likelihood 을 계산하기 위해 포아송 프로세스 이론을 다시 활용할 수 있습니다. 각 팀의 골 득점률이 일정하다면, 골 사이의 시간은 지수 분포를 따를 것으로 예상할 수 있다.

골 득점률이 𝜆인 경우, 골 간격이 𝑡일 확률은 지수 분포 PDF 에 비례한다.

$$𝜆exp(-𝜆𝑡)$$

𝑡는 연속적인 양이므로 이 식의 값은 확률이 아니라 확률 밀도이다. 하지만 데이터의 확률에 비례하므로 베이지안 업데이트에서 likelihood 로 사용할 수 있다.

SciPy는 지수 분포를 나타내는 객체를 생성하는 지수 함수를 제공한다. 그러나 예상하는 방식으로 lam 을 매개변수로 사용하지 않기 때문에 작업하기가 어색하다.

```python
def expo_pdf(t, lam):
    """Compute the PDF of the exponential distribution."""
    return lam * np.exp(-lam * t)
```

지수 분포가 어떻게 생겼는지 알아보기 위해 다시 한 번 lam이 1.4라고 가정하면 다음과 같이 𝑡의 분포를 계산한다.
```python
>>> lam = 1.4
>>> qs = np.linspace(0, 4, 101)
>>> ps = expo_pdf(qs, lam)
>>> pmf_time = Pmf(ps, qs)
>>> pmf_time.normalize()
25.616650745459093
```

![](https://i.imgur.com/vJe6FfS.png)

- 직관적이지 않지만 사실 골을 넣을 가능성이 가장 높은 시점은 바로 그 순간이다. 그 이후에는 연속되는 각 간격의 확률이 조금 낮아진다.  
- 골 득점률 1.4 로 한 팀이 골을 넣는 데 한 경기 이상 걸릴 가능성은 있지만 두 경기 이상 걸릴 가능성은 거의 없다.

# Summary
이 장에서는 세 가지 분포을 소개하므로 정리하기 어려울 수 있다.

- 시스템이 포아송 모델의 가정을 만족하는 경우, 일정 기간 동안의 이벤트 수는 0에서 무한대까지의 정수를 갖는 이산 분포인 푸아송 분포를 따른다. 실제로는 일반적으로 유한한 한계를 초과하는 낮은 확률의 양은 무시할 수 있다.
- 또한 포아송 모델에서 이벤트 사이의 간격은 0에서 무한대까지의 양을 가진 연속 분포인 지수 분포를 따른다. 연속적이기 때문에 확률 질량 함수(PMF)가 아닌 확률 밀도 함수(PDF)로 나타낸다. 그러나 지수 분포를 사용하여 데이터의 가능성을 계산할 때 밀도를 정규화하지 않은 확률로 취급할 수 있다.
- 푸아송 분포와 지수 분포는 𝜆 또는 lam으로 표시되는 이벤트 비율로 매개변수화 된다.
- 𝜆 의 이전 분포에는 0에서 무한대까지의 연속 분포인 감마 분포를 사용했지만, 이산적이고 경계가 있는 PMF로 근사화했다. 감마 분포에는 𝛼 또는 알파로 표시되는 하나의 매개변수가 있으며, 이는 평균을 나타낸다.

감마 분포를 선택한 이유는 그 모양이 골 득점률에 대한 배경 지식과 일치하기 때문이다. 다른 분포를 사용할 수도 있지만, ConjugatePriors 에서 감마 분포가 특히 좋은 선택이 될 수 있음을 살펴볼 것이다.
