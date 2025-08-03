---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter03-Distribution/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

이번 장에서는 여러분의 인내심을 시험하는 위험을 무릅쓰고 '확률 질량 함수'(probability mass function)를 나타내는 Pmf 를 사용해 문제를 한 번 더 풀어본다. 이것이 무엇을 의미하는지, 그리고 베이지안 통계에 왜 유용한지 설명한다
  
Pmf 를 사용하여 좀 더 어려운 문제를 해결하고 베이지안 통계에 한 걸음 더 다가가자.

# Distributions
> A distribution is simply a collection of data, or scores, on a variable. Usually, these scores are arranged in order from smallest to largest and then they can be presented graphically. (Page 6, [Statistics in Plain English](http://amzn.to/2FTs5TB), Third Edition, 2010.)
> 
- **통계에서 분포는 가능한 결과와 그에 해당하는 확률집합을 의미한다.**
	- 예를 들어 동전을 던지면 동일한 확률로 두 가지 결과
	- 6면 주사위를 굴리면 가능한 결과의 집합은 숫자 1에서 6까지이며 각 결과와 관련된 확률은 1/6
- 분포를 표현하기 위해 empiricaldist라는 라이브러리를 사용
- "경험적(empiricaldist)" 분포는 이론적 분포와 달리 데이터 기반
- 책 전체에서 이 라이브러리를 사용

# pmf (probability mass functions)
- 분포의 결과가 불연속적인(discrete) 경우, 가능한 각 결과에서 해당 확률로 매핑하는 함수인 확률 질량 함수(PMF)로 distribution 을 표현할 수 있다.
	- [probability mass function](https://ko.wikipedia.org/wiki/%ED%99%95%EB%A5%A0_%EC%A7%88%EB%9F%89_%ED%95%A8%EC%88%98)
- `empiricaldist` 는 확률 질량 함수를 나타내는 Pmf 클래스를 제공한다.

```python
>>> from empiricaldist import Pmf
>>> 
>>> coin = Pmf()
>>> coin['heads'] = 1/2
>>> coin['tails'] = 1/2
>>> coin
```

![](https://i.imgur.com/NpzAOUD.png)

배열을 입력으로 하는 Pmf.from_seq 함수를 이용해 확률을 계산할 수 있다.

```python
>>> die = Pmf.from_seq([1,2,3,4,5,6])
>>> die
```

![](https://i.imgur.com/GUR8kD5.png)

특정 단어에서 문자가 출현할 확률도 구할 수 있다.

```python
letters = Pmf.from_seq(list('Mississippi'))
letters
```

![](https://i.imgur.com/0OhfL38.png)


# The Cookie Problem Revisited (with pmf)
앞서 살펴본 쿠키 문제를 pmf 를 이용해 살펴본다.

```python
>>> prior = Pmf.from_seq(['Bowl 1', 'Bowl 2'])
>>> prior
```

![](https://i.imgur.com/kYLcCxD.png)

- 각 가설에 대한 사전 확률을 포함하는 분포를 사전 분포(prior distribution)라 한다.

```python
>>> likelihood_vanilla = [0.75, 0.5]
>>> posterior = prior * likelihood_vanilla
>>> posterior
```

![](https://i.imgur.com/Lsp2Xsk.png)

- posterior distribution: 각 가설에 대한 사후 확률을 나타내는 분포 (==사후분포==)

# 101 Bowls (with pmf)
앞서 살펴본 쿠키 문제를 확장해서 풀어본다. (2 Bowls -> 101 Bowls)
- uniform: 모든 가설의 사전 확률이 동일한 분포
- MAP(maximum a posteriori probability): 후행 확률이 가장 높은 값을 "최대 후행 확률" 이라한다.

# The Dice Problem (with pmf)

# Updating Dice
위 문제에서 여러번 계산하던 것을 하나의 함수로 정의한다.

```python
def update_dice(pmf, data):
    """Update pmf based on new data."""
    hypos = pmf.qs
    likelihood = 1 / hypos
    impossible = (data > hypos)
    likelihood[impossible] = 0 # 주사위 숫자보다 큰 경우 
    pmf *= likelihood
    pmf.normalize()
```

# Summary
- 가설과 그 확률을 표현하는 데 사용하는 Pmf 를 제공하는 empiricaldist 모듈을 살펴봤다.
- empiricaldist는 pandas 를 기반으로 하며, Pmf 클래스는 Series 클래스를 상속하고 확률 질량 함수에 특화된 추가 기능을 제공한다.
	- 이 책에서는 코드를 간소화하고 가독성을 높이기 위해 empiricaldist의 Pmf 및 기타 클래스를 사용할 예정이다. 하지만 pandas 로 직접 동일한 작업을 수행할 수도 있다.
- 이전 장에서 살펴본 쿠키 문제와 주사위 문제를 푸는 데 Pmf를 사용했다. Pmf를 사용하면 여러 데이터 조각으로 순차적 업데이트를 쉽게 수행할 수 있다.
- 또한 두 개의 그릇이 아닌 101개의 그릇을 사용하여 보다 일반적인 쿠키 문제도 해결했다. 그런 다음 사후 확률이 가장 높은 MAP를 계산했다.

# reference
- [Distributions](http://allendowney.github.io/ThinkBayes2/chap03.html)
- [probability mass function](https://ko.wikipedia.org/wiki/%ED%99%95%EB%A5%A0_%EC%A7%88%EB%9F%89_%ED%95%A8%EC%88%98)
- [probability density function](https://ko.wikipedia.org/wiki/%ED%99%95%EB%A5%A0_%EC%A7%88%EB%9F%89_%ED%95%A8%EC%88%98)