---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter02-Bayes-Theorem/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

이전 챕터에서 General Social Suvery 데이터는 베이즈 정리를 이용해 계산이 쉬웠다. 이렇게 완전한 데이터 집합을 가지고 있는 경우 굳이 베이즈 정리를 사용하지 않아도 된다.

즉, **완전하지 않은 데이터에 베이즈 정리를 유용하게 사용할 수 있다.**

# The Cookie Problem
[Urn Problem](https://en.wikipedia.org/wiki/Urn_problem)

$$P(B_1|V) = \frac{P(B_1)~P(V|B_1)}{P(V)}$$
- P(V) 를 구하기 위해 law of total probability 를 이용한다.
$$P(V) = P(B_1)~P(V|B_1) ~+~ P(B_2)~P(V|B_2)$$
$$P(V) = (1/2)~(3/4) ~+~ (1/2)~(1/2) = 5/8$$
- 두 그릇을 선택할 확률이 같고 두 그릇에 같은 수의 쿠키가 들어 있기 때문에 어떤 쿠키를 선택할 확률도 같다. 따라서 위와같은 계산 말고 총 80개 쿠키 중 바닐라 쿠키를 고를 확률로 계산해도 동일하다. $P(V) = 5/8$.
- 마지막으로 베이즈 정리(정리 2)를 이용해 확률은 다음과 같이 계산할 수 있다. $$P(B_1|V) = (1/2)~(3/4)~/~(5/8) = 3/5$$

# Diachronic Bayes (통시적 Bayes)
베이지안 정리를 다른 관점에서 생각해볼 수 있다. 어떤 데이터(D)가 주어졌을 때 가설(H)의 확률을 업데이트할 수 있는 방법을 알려준다.

Diachronic 은 "통시적"이라는 의미로, 시간에 따른 변화와 관련이 있다는 의미이다. 새로운 데이터를 볼 때마다 가설의 확률이 달라진다.

$$P(H|D) = \frac{P(H)~P(D|H)}{P(D)}$$
- $P(H)$ is the probability of the hypothesis before we see the data, called the prior probability, or just **prior**.
	- **사전확률**
- $P(H|D)$ is the probability of the hypothesis after we see the data, called the **posterior**.
	- **사후확률**
- $P(D|H)$ is the probability of the data under the hypothesis, called the **likelihood**.
	- [likelihood](http://rstudio-pubs-static.s3.amazonaws.com/204928_c2d6c62565b74a4987e935f756badfba.html)
- $P(D)$ is the **total probability of the data**, under any hypothesis.

> likelihood 와 probability 의 차이를 찾아보다 두 개념의 차이를 설명하는 링크를 공유한다.
> - [https://wikidocs.net/9088](https://wikidocs.net/9088)
> - [https://esj205.oopy.io/719db4cb-89c2-4420-af5d-c467dfdfa86e](https://esj205.oopy.io/719db4cb-89c2-4420-af5d-c467dfdfa86e)

total probability 를 여러 가설에 적용하면 P(D) 는 다음과 같이 계산할 수 있다.
$$P(D) = P(H_1)~P(D|H_1) + P(H_2)~P(D|H_2)$$
$$P(D) = \sum_i P(H_i)~P(D|H_i)$$
데이터와 사전 확률($P(H)$)를 이용해 사후 확률을 계산하는 과정을 **Bayesian update** 라고 한다.

# Bayes Table

```python
>>> import pandas as pd
>>> table = pd.DataFrame(index=['Bowl 1', 'Bowl 2'])

>>> table['prior'] = 1/2, 1/2
>>> table
        prior
Bowl 1    0.5
Bowl 2    0.5

>>> table['likelihood'] = 3/4, 1/2
>>> table
        prior  likelihood
Bowl 1    0.5        0.75
Bowl 2    0.5        0.50
```
- 이전과 다르게 모든 가설($B_{1}$/$B_{2}$) 에 대한 확률을 계산한다.
	- The chance of getting a vanilla cookie from Bowl 1 is 3/4.
	- The chance of getting a vanilla cookie from Bowl 2 is 1/2.
- likelihood 의 총 합은 1이 되지 않아도 된다.

```python
>>> table['unnorm'] = table['prior'] * table['likelihood']
>>> table
        prior  likelihood  unnorm
Bowl 1    0.5        0.75   0.375
Bowl 2    0.5        0.50   0.250
```

"정규화되지 않은 후행"이기 때문에 결과를 unnorm 이라고 부른다. 각각의 값은 이전 값과 확률의 곱이다.

$$P(B_i)~P(D|B_i)$$
$$P(B_1)~P(D|B_1) + P(B_2)~P(D|B_2) = P(D)$$
```python
>>> prob_data = table['unnorm'].sum()
>>> prob_data
0.625
# 0.625 = 5/8
```

그릇 1의 사후 확률은 0.6이며, 이는 베이지의 정리를 명시적으로 사용하여 얻었다. 보너스로 그릇 2의 사후 확률 0.4 이다.
  
정규화되지 않은 posteriors 를 더하고 나누면 posteriors 가 1이 되도록 강제한다. 이 과정을 "**정규화**"라고 하며, 이 때문에 데이터의 총 확률을 "정규화 상수(normalizing constant)"라고도 한다.

# The Dice Problem
Bayes Table 을 이용해 다음 주사위 문제를 쉽게 풀 수 있다.

> 6면 주사위, 8면 주사위, 12면 주사위가 들어 있는 상자가 있다고 가정합니다. 주사위 중 하나를 무작위로 선택하여 굴린 후 결과가 1이라고 보고합니다. 6면 주사위를 선택했을 확률은 얼마인가요?

```python
>>> table2 = pd.DataFrame(index=[6, 8, 12])
>>>
>>> from fractions import Fraction
>>> table2['prior'] = Fraction(1, 3)
>>> table2['likelihood'] = Fraction(1, 6), Fraction(1, 8), Fraction(1, 12)
>>> table2
```

![](https://i.imgur.com/ozsN9Xu.png)

update 함수를 다음과 같이 정의할 수 있다.
```python
def update(table):
    """Compute the posterior probabilities."""
    table['unnorm'] = table['prior'] * table['likelihood']
    prob_data = table['unnorm'].sum()
    table['posterior'] = table['unnorm'] / prob_data
    return prob_data
>>> prob_data = update(table2)
>>> table2
```

![](https://i.imgur.com/PhHEJTS.png)

# The Monty Hall Problem
다음으로 베이지안 테이블을 사용하여 확률에서 가장 논쟁의 여지가 있는 문제 중 하나를 해결해보자.

> The host, Monty Hall, shows you three closed doors – numbered 1, 2, and 3 – and tells you that there is a prize behind each door.
> 
> One prize is valuable (traditionally a car), the other two are less valuable (traditionally goats).  
> 
> The object of the game is to guess which door has the car. If you guess right, you get to keep the car.
> 
> Suppose you pick Door 1. Before opening the door you chose, Monty opens Door 3 and reveals a goat. Then Monty offers you the option to stick with your original choice or switch to the remaining unopened door.
> 
> To maximize your chance of winning the car, should you stick with Door 1 or switch to Door 2?
> 
> To answer this question, we have to make some assumptions about the behavior of the host
> 1. Monty always opens a door and offers you the option to switch.
> 2. He never opens the door you picked or the door with the car.
> 3. If you choose the door with the car, he chooses one of the other doors at random.

# Summary
- 이 장에서는 베이지의 정리를 명시적(explicity)으로 사용하고 베이지 표를 사용해 쿠키 문제를 해결했다.
	- 이 방법들 사이에 실질적인 차이는 없지만, 베이지안 테이블을 사용하면 특히 가설이 두 개 이상인 문제에서 데이터의 총 확률을 더 쉽게 계산할 수 있다.
- 볼 주사위 문제와 다시는 보지 않기를 바라는 몬티 홀 문제를 풀었다. 
- 몬티 홀 문제 때문에 머리가 아팠다면 여러분은 혼자가 아닙니다. 하지만 이 문제는 까다로운 문제를 해결하기 위한 분할 정복 전략(divide-and-conquer strategy)으로서 베이즈 정리의 힘을 보여준다고 생각한다.  
	- 몬티가 문을 열면 자동차의 위치에 대한 우리의 믿음을 업데이트하는 데 사용할 수 있는 정보를 제공한다.
	- 정보 중 일부는 명백(obvious)하다.
	- 몬티가 3번 문을 열면 차가 3번 문 뒤에 있지 않다는 것을 알 수 있다.
	- 그러나 일부 정보는 더 미묘(subtle)합니다. 차가 2번 문 뒤에 있으면 3번 문을 열 가능성이 높고 1번 문 뒤에 있으면 가능성이 낮다. 따라서 데이터는 도어 2로 변경하는 것이 유리하다고 설명한다. 다음 장에서 이 증거 개념에 대해 다시 설명한다.
- 자세한 내용은 [나무위키](https://namu.wiki/w/%EB%AA%AC%ED%8B%B0%20%ED%99%80%20%EB%AC%B8%EC%A0%9C) 를 참고해도 좋을 것 같다.

# reference
- [Bayes's Theorem](http://allendowney.github.io/ThinkBayes2/chap02.html)
- [posterior과 bayesian - 대학원생이 쉽게 설명해보기](https://hwiyong.tistory.com/27)