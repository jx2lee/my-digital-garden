---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter06-Odds-and-Addends/","noteIcon":""}
---

#think-bayes #probability #study

---

degree of certainty 를 표현하는 새로운 방법과 새로운 형태의 베이즈 정리(==Bayes's Rule==)를 소개한다. 또한 evidence 의 중요한 개념과 강도를 정량화할 수 있는 방법에 대해서도 설명한다.

이 장의 두 번째로 addends 에 대해 살펴본다. 합계, 차이, 곱셈 및 기타 연산의 분포를 계산하는 함수를 정의하는데, 베이지안 업데이트를 일부 사용한다.

# Odds
확률을 나타내는 한 가지 방법은 0과 1 사이의 숫자를 사용하는 것이지만, 이것이 유일한 방법은 아니다. 축구 경기나 경마에 베팅을 해본 적이 있다면 배당률이라는 또 다른 표현을 접해 보셨을 거다.

"배당률은 3 대 1입니다"와 같은 표현을 들어 보셨을 수도 있지만 그것이 무엇을 의미하는지 모를 수도 있다. odds 는 ==사건이 발생할 확률과 발생하지 않을 확률의 비율==을 말한다.

```python
def odds(p):
    return p / (1-p)
```

예를 들어, 팀이 이길 확률이 75%인 경우, 이길 확률이 질 확률이 3배이므로 팀이 유리한 확률은 3 대 1이다.

```python
>>> odds(0.75)
3.0
```

확률은 소수점 형식으로 쓸 수 있지만 정수 비율로 쓰는 것이 일반적이다. 따라서 "3 대 1"은 3:1 로 표현한다.

확률이 낮을 때는 찬성 확률보다는 반대 배당(odds against)을 사용하는 것이 일반적이다. 예를 들어, 내 말이 이길 확률이 10%인 경우 유리한 배당률은 1:9 이다.
하지만 이 경우에 반대로 9:1 이라 표현한다.

```python
>>> odds(0.1)
0.11111111111111112
>>> odds(0.9)
9.000000000000002
```

odds 를 이용해 확률을 구하는 식은 다음과 같이 표현할 수 있다.

$$\frac{p}{1-p} = o$$
$$ p * (1 + o) = o$$
$$p = \frac{o}{1+o}$$


```python
def prob(o):
    return o / (o+1)

>>> prob(3/2)
0.6
```

또는 분자와 분모로 odds 을 나타내는 경우 다음과 같이 표현할 수 있다.

```python
def prob2(yes, no):
    return yes / (yes + no)

>>> prob2(3,2)
0.6
```

probability 와 odds 는 동일한 정보를 다르게 표현한 것으로, 둘 중 하나를 알면 다른 하나를 계산할 수 있다. 하지만 odds 로 작업할 때 더 쉬운 계산도 있고, 나중에 살펴볼 로그 확률을 사용하면 훨씬 더 쉬운 계산도 있다.

# Bayes's Rule
$$P(H|D) = \frac{P(H)~P(D|H)}{P(D)}$$

위 확률 형식으로 된 베이즈 정리를 odds 형식으로 표현하면 다음과 같다.
$$\mathrm{odds}(A|D) = \mathrm{odds}(A)~\frac{P(D|A)}{P(D|B)}$$

이를 **bayes's rule** 이라 하며, 사후 odds 는 사전 odds 에 likelihood ratio 를 곱한 값이다.

쿠키 문제로 돌아가 보면 쿠키가 담긴 그릇이 두 개 있다고 가정해 해보자. 볼 1에는 바닐라 쿠키 30개와 초콜릿 쿠키 10개가 들어 있고, 볼 2에는 각각 20개가 들어 있다. 두 그릇 중 하나를 무작위로 선택하고 보지 않고 무작위로 쿠키를 선택한다. 선택한 쿠키가 바닐라라면 그릇 1에서 나왔을 확률은 얼마인가?

prior 확률은 50% 이므로 prior odds 는 1입니다. likelihood ratio 는 34/12 또는 3/2 이다. 따라서 사후 확률은 3/2 이며, 이는 확률 3/5 에 해당한다.

```python
>>> prior_odds = 1
>>> likelihood_ratio = (3/4) / (1/2)
>>> post_odds = prior_odds * likelihood_ratio
>>> post_odds
1.5
>>> post_prob = prob(post_odds)
>>> post_prob
0.6
```

만약 하나의 쿠키와 초콜릿을 추가로 선택했다면 update 를 통해 posterior 를 구할 수 있다.
```python
>>> likelihood_ratio = (1/4) / (1/2)
>>> post_odds *= likelihood_ratio
>>> post_odds
0.75
>>> post_prob = prob(post_odds)
>>> post_prob
0.42857142857142855
```

# Oliver's Blood
bayes's rule 을 사용하여 MacKay의 정보 이론, 추론 및 학습 알고리즘에 나오는 또 다른 문제를 풀어보자.

> 두 사람이 범죄 현장에서 자신의 혈액 흔적을 남겼습니다. 용의자 올리버는 검사 결과 'O형' 혈액형을 가진 것으로 밝혀졌습니다. 두 사람의 혈액형은 'O형'(지역 인구에서 빈도가 60%인 흔한 유형)과 'AB형'(빈도가 1%인 희귀 유형)으로 밝혀졌습니다. 이러한 데이터[현장에서 발견된 흔적]가 올리버가 [현장에서 혈액을 남긴] 사람 중 한 명이라는 명제에 유리한 증거를 제공하나요?

- 이 질문에 답하기 위해서는 데이터가 가설에 찬성하거나 반대하는 증거를 제공한다는 것이 무엇을 의미하는지 생각해 볼 필요가 있다.
- 직관적으로, 데이터에 비추어 가설이 이전보다 더 가능성이 높으면 데이터가 가설에 유리하다고 말할 수 있다.
- 쿠키 문제에서는 prior odds 가 1이며, 이는 확률 50%에 해당한다. 사후 확률(posterior odds)은 3/2 , 즉 확률 60%입니다. 따라서 바닐라 쿠키는 보울 1에 유리한 증거다.

Bayes's Rule 은 직관을 더욱 정확하게 만드는 방법을 제공한다.

$$\mathrm{odds}(A|D) = \mathrm{odds}(A)~\frac{P(D|A)}{P(D|B)}$$

양변을 odds(A) 로 나누면 식이 다음과 같이 바뀐다.

$$\frac{\mathrm{odds}(A|D)}{\mathrm{odds}(A)} = \frac{P(D|A)}{P(D|B)}$$

- 좌변은 사후 odds 와 사전 odds 의 비율이다.
- 우변은 베이즈 계수(Bayes factor)라고도 하는 likelihood 의 비율(likelihood ratio) 이다.
- 베이즈 계수가 1보다 크면 데이터가 B 에서보다 A 에서 더 가능성이 높다는 것을 의미한다. 즉, 데이터에 비추어 볼 때 이전보다 확률이 더 커졌다는 것이다.
- 베이즈 계수가 1보다 작으면 데이터가 𝐵보다 𝐴에서 가능성이 낮다는 의미이므로 𝐴에 유리한 확률이 낮아진다.
- 베이즈 계수가 정확히 1이면 두 가설 하에서 데이터의 가능성이 동일하므로 확률은 변하지 않는다.
- 위 문제에 적용해보자.
	- 올리버가 범죄 현장에 혈액을 남긴 사람 중 한 명이라면 그는 'O' 샘플에 해당하며, 이 경우 데이터의 확률은 인구 중 무작위 구성원이 'AB형' 혈액을 가지고 있을 확률로 1%가 된다.
	- 올리버가 현장에 혈액을 남기지 않았다면 두 가지 경우로 살펴볼 수 있다. 모집단에서 무작위로 두 명을 선택하면 한 명은 'O형', 한 명은 'AB형'을 찾을 확률은 얼마나 될까?
		- 첫 번째 사람은 'O'형이고 두 번째 사람은 'AB'형일 수 있다.
		- 또는 첫 번째 사람이 'AB'이고 두 번째 사람이 'O'일 수도 있다.
	- 두 조합의 확률은 (0.6)(0.01) 로 0.6% 이므로 총 확률은 그 두 배인 1.2% 다 (`0.6% * 2`). 따라서 **올리버가 현장에 혈액을 남긴 사람 중 한 명이 아니라면 데이터는 조금 더 가능성이 높다.**

이러한 확률을 사용하여 likelihood ratio 을 계산할 수 있다.

```python
>>> like1 = 0.01
>>> like2 = 2 * 0.6 * 0.01
>>> 
>>> likelihood_ratio = like1 / like2
>>> likelihood_ratio
0.8333333333333334
```

likelihood ratio 가 1보다 작기 때문에 혈액 검사는 올리버가 현장에 피를 남겼다는 가설에 대한 유리한 증거가 된다.

하지만 이는 약한 증거다. 예를 들어, 사전 odds 가 1(즉, 50% 확률)인 경우 사후 odds 은 0.83 이며 45% 확률(prob)값을 갖는다.

```python
>>> post_odds = 1 * like1 / like2
>>> prob(post_odds)
0.45454545454545453
```

따라서 증거는 그다지 유리하지 않다.
  
이 예는 약간 인위적이지만 가설과 일치하는 데이터가 반드시 가설에 유리한 것은 아니라는 반직관적인 결과를 보여준다.
  
이 결과가 여전히 마음에 걸리면 다음과 같은 사고 방식이 도움이 될 수 있다. 데이터는 일반적인 사건인 'O형' 과 희귀한 사건인 'AB형' 로 구성한다. 올리버가 공통 사건을 설명한다면 희귀 사건은 설명할 수 없다. 올리버가 'O형' 혈액을 설명하지 않는다면 인구에서 'AB형' 혈액을 가진 사람을 찾을 수 있는 기회가 두 번 있다. 그리고 그 2의 계수가 차이를 만든다. (?)

> 🚨🚨 마지막 설명하는 부분이 조금 이해되지 않는다. 이야기하면 좋을 것 같다.

# Addends
분포 합계와 다른 연산 결과의 분포에 대해 알아본다. 먼저 입력이 주어지고 출력의 분포를 계산하는 정방향(forward) 문제부터 시작한다. 그런 다음 출력이 주어지고 입력의 분포를 계산하는 역방향(inverse) 문제를 다룬다.

첫 번째 예로 주사위 두 개를 굴려서 합산하면 합계의 분포는 어떻게 될까? 다음 함수를 사용하여 주사위의 가능한 결과를 나타내는 Pmf 를 다음과 같이 만들어본다.

```python
>>> import numpy as np
>>> from empiricaldist import Pmf
>>> 
>>> def make_die(sides):
>>>     outcomes = np.arange(1, sides+1)
>>>     die = Pmf(1/sides, outcomes)
>>>     return die
>>> 
>>> die = make_die(6)
```

![](https://i.imgur.com/Qi5F9v9.png)

- 만약 두 개의 주사위를 굴려서 더하면, 2부터 12까지 11가지 결과가 나온다.
- 두 수의 합이 나올 확률이 모두 같지 않다.

```python
def add_dist(pmf1, pmf2):
    """Compute the distribution of a sum."""
    res = Pmf()
    for q1, p1 in pmf1.items():
        for q2, p2 in pmf2.items():
            q = q1 + q2
            p = p1 * p2
            res[q] = res(q) + p
    return res
```

- `add_dist` function
	- 루프를 통과할 때마다 q는 한 쌍의 수량의 합을 얻고, p는 그 쌍의 확률을 얻는다.
	- 동일한 합이 두 번 이상 나타날 수 있으므로 각 합에 대한 총 확률을 더한다.
- `res[q] = res(p) + p`
	- 오른쪽은 소괄호를 사용하는데, q가 표시되지 않으면 0을 반환한다.
	- 왼쪽은 대괄호를 사용하여 업데이트 한다.
- Pmf 라이브러리에서는 동일한 기능을 하는 add_dist 함수를 제공하고 이는 아래와 같이 사용할 수 있다.
	- `twice = die.add_dist(die)` or `twice = Pmf.add_dist(die, die)`
		```python
		twice = die.add_dist(die)
		twice2 = Pmf.add_dist(die, die)
		twice == twice2
		2     True
		3     True
		4     True
		5     True
		6     True
		7     True
		8     True
		9     True
		10    True
		11    True
		12    True
		Name: , dtype: bool
		```

두 주사위 합의 분포는 다음과 같이 나온다.
![](https://i.imgur.com/geNPkzU.png)

pmf 배열로 이루어진 입력을 받아 분포합계를 구하는 함수는 다음과 같이 구현할 수 있다.
```python
>>> def add_dist_seq(seq):
    """Compute Pmf of the sum of values from seq."""
    total = seq[0]
    for other in seq[1:]:
        total = total.add_dist(other)
    return total
```

3개 주사위를 던지는 경우 두 수의 합을 나타내는 분포를 함께 나타내면 다음과 같다.

```python
>>> dice = [die] * 3
>>> thrice = add_dist_seq(dice)
```

![](https://i.imgur.com/7UTYOFR.png)

주사위 1개, 2개 합, 3개 합 분포는
- 단일 주사위의 분포는 1에서 6까지 균일하다.
- 두 주사위의 합계는 2에서 12 사이의 삼각형 분포를 갖는다.
- 주사위 3개 합은 3에서 18 사이의 종 모양 분포를 갖는다.

이 예는 합계의 분포가 적어도 일부 조건에서는 종 모양의 정규 분포에 수렴한다는 [중심 극한 정리](https://ko.wikipedia.org/wiki/%EC%A4%91%EC%8B%AC_%EA%B7%B9%ED%95%9C_%EC%A0%95%EB%A6%AC)를 보여준다.

# Gluten Sensitivity

- 2015년에 글루텐 민감성 진단을 받은 사람들이 블라인드 챌린지에서 글루텐 밀가루와 비글루텐 밀가루를 구별할 수 있는지 테스트한 [논문](https://onlinelibrary.wiley.com/doi/full/10.1111/apt.13372)이 있다.
	- 35명의 피험자 중 12명은 글루텐 밀가루를 섭취하는 동안 증상발현을 기준으로 글루텐 밀가루를 정확하게 식별했다.
	- 나머지 17명은 증상에 따라 글루텐이 함유되지 않은 밀가루를 잘못 식별했으며, 6명은 구분할 수 없었다.
- 저자는 "환자의 1/3에서만 증상 재발을 유도한다"고 결론지었다.

글루텐에 민감한 환자가 한 명도 없었다면 환자 중 일부는 우연히 글루텐 밀가루를 식별할 것으로 예상할 수 있기 때문에 이 결론은 이상하게 보일 수 있다. 그렇다면 이 데이터에 근거하여 글루텐에 민감한 사람은 몇 명이고 추측하는 사람은 몇 명일까?

- 베이즈 정리를 사용하여 이 질문에 답할 수 있지만, 먼저 몇 가지 모델링 결정(modeling descisions)이 필요하다. 문제를 풀기 전 가정은 다음과 같다.
	- 글루텐에 민감한 사람은 문제 조건에서 글루텐 밀가루를 정확하게 식별할 확률이 95% 이다.
	- 글루텐에 민감하지 않은 사람은 우연히 글루텐 밀가루를 식별할 확률이 40%이고, 다른 밀가루를 선택하거나 구분하지 못할 확률은 60% 이다.

이러한 특정 값은 임의적이지만 결과는 이러한 선택에 민감하지 않다.

이 문제는 두 단계로 해결한다. 먼저, 1) 실험자가 민감하다 가정하고 데이터의 분포를 계산한다. 그런 다음 2) likelihood 을 사용하여 민감한 환자 수의 사후 분포를 계산한다.

첫 번째는 순방향(forward problem) 문제이고 두 번째는 역방향(inverse) 문제입니다.

## The Forward Problem

35명 중 10명이 글루텐에 민감하다고 가정해보자.
```python
n = 35
num_sensitive = 10
num_insensitive = n - num_sensitive
```

각 민감한 피험자는 글루텐 가루를 식별할 확률이 95%이므로 올바른 식별 횟수는 이항 분포(binomial distribution)를 따른다.

이항 분포를 나타내는 Pmf를 만들기 위해 TheBinomialDistribution 에 정의한 make_binomial을 사용한다.

```python
from utils import make_binomial

dist_sensitive = make_binomial(num_sensitive, 0.95)
dist_insensitive = make_binomial(num_insensitive, 0.40)
```

Pmf.add_dist 를 이용해 총 식별자 수의 분포를 계산할 수 있다.
```python
dist_total = Pmf.add_dist(dist_sensitive, dist_insensitive)
```

![](https://i.imgur.com/9qXL3si.png)

- 민감한 실험자 대부분은 글루텐을 정확하게 식별할 것으로 예상한다.
- 민감하지 않은 25명의 피험자 중 10명 정도는 우연히 글루텐 가루를 식별할 것으로 예상한다.
- 따라서 총 20명 정도 정답을 맞출 것으로 예상한다.

**민감한 실험자 수만 주어진다면 분포를 계산할 수 있다.**

## The Inverse Problem
데이터가 주어지면 민감한 피험자 수의 posterior 분포를 계산한다.

방법은 다음과 같다. num_sensitive의 가능한 값을 반복하여 각각의 데이터 분포를 계산한다.

```python
import pandas as pd

table = pd.DataFrame()
for num_sensitive in range(0, n+1):
    num_insensitive = n - num_sensitive
    dist_sensitive = make_binomial(num_sensitive, 0.95)
    dist_insensitive = make_binomial(num_insensitive, 0.4)
    dist_total = Pmf.add_dist(dist_sensitive, dist_insensitive)    
    table[num_sensitive] = dist_total
```

- 이 루프는 num_sensitive의 가능한 값을 갖는다.
- 각 값에 대해 올바른 식별의 총 개수 분포를 계산하고 그 결과를 판다스 데이터프레임의 열로 저장한다.

```python
table[0].plot(label='num_sensitive = 0')
table[10].plot(label='num_sensitive = 10')
table[20].plot(label='num_sensitive = 20', ls='--')
table[30].plot(label='num_sensitive = 30', ls=':')
    
decorate(xlabel='Number of correct identifications',
         ylabel='PMF',
         title='Gluten sensitivity')
```

![](https://i.imgur.com/v8ITfyy.png)

이제 이 테이블을 사용하여 likelihood 를 계산할 수 있다.

```python
likelihood1 = table.loc[12]
```

- `loc` 은 데이터 프레임에서 행을 선택하는 함수다.
- 인덱스가 12인 행에는 num_sensitive 각 가상(hypothetical) 값에 대해 12개 올바른 식별 확률이 존재한다.

```python
>>> hypos = np.arange(n+1)
>>> prior = Pmf(1, hypos)
>>> posterior1 = prior * likelihood1
>>> posterior1.normalize()
0.4754741648615132
```

비교를 위해 또 다른 가능한 결과인 20개 정답에 대한 posterior 결과도 계산해보자.

```python
1.7818649765887375
```

12, 20개 정답에 대한 posterior 분포를 그리면 다음과 같다.

![](https://i.imgur.com/5RuEVNx.png)

```python
>>> posterior1.max_prob()
0
>>> posterior2.max_prob()
11
```

- 12명이 정답을 맞춘 경우 가장 가능성이 높은 결론은 글루텐에 민감한 실험자는 아무도 없다.
- 20명이 정답을 맞춘 경우 가장 가능성이 높은 결론은 피험자 중 11~12명이 글루텐에 민감하다.

# Summary
- 베이즈 법칙(Bayes's Rule)과 증거(evidence), 그리고 확률 비율(likelihood ratio) 또는 베이즈 계수(Bayes factor)를 사용하여 증거의 강도를 정량화하는 방법에 대해 알아봤다.  
- 합계의 분포를 계산하는 add_dist에 대해 알아보았다. 이 함수를 사용하면 시스템의 매개변수가 주어지면 데이터의 분포를 계산하거나, 데이터가 주어지면 매개변수의 분포를 계산하는 정방향 및 역방향 문제를 해결할 수 있었다.
