---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter10-Testing/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

유로 동전 데이터 문제를 살펴보자.
문제에 대한 답으로 이 데이터가 동전이 한 쪽으로 기울었다는 것의 증거가 될까? 라는 질문에 답은 못했다. 이 챕터에서 답변을 들어본다.

# Estimation
```python
import numpy as np
from empiricaldist import Pmf
from scipy.stats import binom

xs = np.linspace(0, 1, 101)
uniform = Pmf(1, xs)

k, n = 140, 250
likelihood = binom.pmf(k, n, xs)

posterior = uniform * likelihood
posterior.normalize()
```

![](https://i.imgur.com/wOOjmRM.png)

posterior 평균은 0.56 이고 90% credible interval 은 0.51 ~ 0.61 이다.

```python
print(posterior.mean(), 
      posterior.credible_interval(0.9))
0.5595238095238094 [0.51 0.61]
```

# Evidence
올리버의 혈액형 문제를 떠올려보자.

$$P(D|A) > P(D|B)$$
- A(가설): 올리버가 현장에 혈흔을 남겼다.
- B(가설): 올리버가 현장에 혈흔을 남기지 않았다.
- D: 올리버가 범인이다.
- D|A: 현장에 혈흔을 남겼을 때 올리버가 범인이다.
- D|B: 현장에 혈흔을 남기지 않았을 때 올리버가 범인이다.

$$K = \frac{P(D|A)}{P(D|B)}$$

올리버의 혈액형 문제와 같이 유로 동전 문제를 fair 와 biased 두 가지 가설을 고려하여 likelihood 를 계산해본다.

```python
k = 140
n = 250

like_fair = binom.pmf(k, n, p=0.5)
like_fair
0.008357181724918204
```

이제 동전이 한 쪽으로 기울어졌다고 가정했을 때 이 데이터가 나올 likelihood 를 계산해보자. 만약 **치우치다**의 의미각 앞면이 56% 나올 확률이라고 알고 있었다면, 다음과 같이 이항분포를 사용할 수 있다.
```python
like_biased = binom.pmf(k, n, p=0.56)
like_biased
0.05077815959518337
```

bayes factor 를 구하면 다음과 같다.
```python
K = like_biased / like_fair
K
6.075990838368465
```

- 1보다 크기 때문에 동전이 한 쪽으로 치우졌을 가능성이 약 6배 높다.
- 허나 위 과정에서 가정을 데이터를 컨팅하듯 사용했다.
	- 앞면이 56% 나올 확률이라고 이미 나온 데이터를 기반으로 가정을 했다.
- **치우치다 의 정의가 필요하다.**

# Uniformly Distributed Bias
- 치우치다 라는 의미를 앞면이 나올 확률이 50%가 아닐것이라 하고 다른 값이 나올 가능성이 동일하다고 가정해보자.
	- Hypo1: 동전이 치우치다는 의미는50% 확률이 아니고 다른 확률이 나올 가능성은 동일하다.
- uniform 을 생성하고 50% 영역을 삭제하면 다음과 같다.
```python
biased_uniform = uniform.copy()
biased_uniform[0.5] = 0
biased_uniform.normalize()
```
- 전체 확률은 다음과 같다.
```python
xs = biased_uniform.qs
likelihood = binom.pmf(k, n, xs)

like_uniform = np.sum(biased_uniform * likelihood)
like_uniform
0.0039004919277707355 # 균등하게 치우친(확률이 50%인) 가설 하에서의 확률

K = like_fair / like_uniform
K
2.142596851801362 # fair 가 biased 할 경우보다 likelihood 가 높다.
```

evidence 의 강력함을 확인하기 위해 Bayes's Rule 를 적용해보자.

```python
prior_odds = 1 # prior: 50% 이면 prior odds 는 1
posterior_odds = prior_odds * K 
posterior_odds
2.142596851801362

def prob(o):
    return o / (o+1)

posterior_probability = prob(posterior_odds)
posterior_probability
0.6817918278551091 # 50% -> 68%, 약하다 약해
```

그럼, 위 설정한 가설(Hypo1)을 바꿔보자. 모든 값에 동일한 가능성을 가지지 않는 것을 의미한다고 변경해보자.
- Hypo2: 동전이 치우치다는 의미는 50% 가 아니고 다른 확률이 나올 가능성은 다르다.
- triangle-shaped distribution 을 이용해보자.
```python
ramp_up = np.arange(50)
ramp_down = np.arange(50, -1, -1)
a = np.append(ramp_up, ramp_down)

triangle = Pmf(a, xs, name='triangle')
triangle.normalize()
0.00    0.0000
0.01    0.0004
0.02    0.0008
0.03    0.0012
0.04    0.0016
         ...  
0.96    0.0016
0.97    0.0012
0.98    0.0008
0.99    0.0004
1.00    0.0000
Name: triangle, Length: 101, dtype: float64
```

![](https://i.imgur.com/DvnlYEa.png)

# Bayesian Hypothesis Testing
- 베이지안 가설검정에서는 bayes factor K 를 이용한다.
- 앞서 유로 동전 문제, 9장을 통해 prediction 과 Decision-making 을 진행했다.
	- prediction: 데이터를 기반으로 미래에 무슨일이 있어날 지 예상할 수 있는가
	- Decision-making: prediction 으로 더 나은 decision 을 내릴 수 있는가

베이지안 가설 검정의 예시 중 하나인 bayesian bandits strategy 를 살펴보자.

# Bayesian Bandits
- bayesian bandits 개념 유래 (슬롯머신)
- 슬롯머신 예제
	- 각각의 슬롯머신이 이길 확률은 서로 다르고 이미 정해져 있다. 확률은 얼마인지 모른다.
	- 각 슬롯머신에 대한 prior belief 는 동일하기 때문에 특정 슬롯을 선호할 필요가 없다. 단, 게임을 몇번 해보면 확률을 추정할 수 있다.
	- 추정한 확률을 기반으로 어떤 슬롯머신을 고를지 정할 수 있다.

# Prior Beliefs
```python
xs = np.linspace(0, 1, 101) # 아무 정보가 없기 때문에 uniform
prior = Pmf(1, xs)
prior.normalize()

beliefs = [prior.copy() for i in range(4)] # 4개 슬롯머신이 있다.
```

![](https://i.imgur.com/8kJgacD.png)

# The Update
```python
likelihood = {
    'W': xs,
    'L': 1 - xs
}

def update(pmf, data):
    """Update the probability of winning."""
    pmf *= likelihood[data]
    pmf.normalize()

bandit = prior.copy()

for outcome in 'WLLLLLLLLL': # 1판 이기고 나머지 9판을 모두 졌다. (갱신)
    update(bandit, outcome)
```

![](https://i.imgur.com/k9nmL3A.png)

# Multiple Bandits
```python
actual_probs = [0.10, 0.20, 0.30, 0.40] # 각 확률을 가지는 슬롯머신

from collections import Counter

# count how many times we've played each machine
counter = Counter()

def play(i):
    """슬롯 머신의 인덱스를 가져와 한 번 게임하고 결과를 W/L 로 반환한다.
    
    i: index of the machine to play
    
    returns: string 'W' or 'L'
    """
    counter[i] += 1
    p = actual_probs[i]
    if np.random.random() < p:
        return 'W'
    else:
        return 'L'

# 각 슬롯 머신에서 10번 게임해보자.
for i in range(4):
    for _ in range(10):
        outcome = play(i)
        update(beliefs[i], outcome)
```

![](https://i.imgur.com/O6CLIne.png)

![](https://i.imgur.com/ly9838n.png)
각 credible interval 에 실제 확률이 포함되있는 것을 확인할 수 있다.

# Explore and Exploit
- bayesian bandits strategy 는 탐색과 활용 사이 균형을 잡는다.
- Thompson Sampling
	- 슬롯 머신을 고를 때 각 슬롯을 고를 확률이 가장 나을 확률에 비례하도록 한다.
- case 1
```python
samples = np.array([b.choice(1000) 
                    for b in beliefs])
samples.shape
(4, 1000)

indices = np.argmax(samples, axis=0)
indices.shape
(1000,)

pmf = Pmf.from_seq(indices)
pmf
0    0.071
1    0.422
2    0.147
3    0.360
```

```python
def choose(beliefs):
    """Use Thompson sampling to choose a machine.
    
    Draws a single sample from each distribution.
    
    returns: index of the machine that yielded the highest value
    """
    ps = [b.choice() for b in beliefs]
    return np.argmax(ps)

choose(beliefs)
```

# The Strategy
위 내용을 종합하여 update 함수를 만들어보자.
```python
def choose_play_update(beliefs):
    """Choose a machine, play it, and update beliefs."""
    
    # choose a machine
    machine = choose(beliefs)
    
    # play it
    outcome = play(machine)
    
    # update beliefs
    update(beliefs[machine], outcome)
```

gogo (run bandits algorithm 100!)
```python
beliefs = [prior.copy() for i in range(4)]
counter = Counter()

num_plays = 100

for i in range(num_plays):
    choose_play_update(beliefs)
    
plot(beliefs)
```

![](https://i.imgur.com/B0hn7T5.png)

```python
summarize_beliefs(beliefs)
   Actual P(win)  Posterior mean Credible interval
0            0.1           0.121       [0.0, 0.34]
1            0.2           0.350      [0.19, 0.53]
2            0.3           0.434      [0.32, 0.55]
3            0.4           0.333      [0.19, 0.49]
```

```python
def summarize_counter(counter):
    """Report the number of times each machine was played.
    
    counter: Collections.Counter
    
    returns: DataFrame
    """
    index = range(4)
    columns = ['Actual P(win)', 'Times played']
    df = pd.DataFrame(index=index, columns=columns)
    for i, count in counter.items():
        df.loc[i] = actual_probs[i], count
    return df

summarize_counter(counter)
summarize_counter(counter)
  Actual P(win) Times played
0           0.1            6
1           0.2           18
2           0.3           51
3           0.4           25
```

- 2번 슬롯머신을 가장 많이 땡겼으므로 2번 슬롯머신을 쓰면..?!

# EOD
- 베이지안 벤딧 전략은 posterior 를 의사결정 과정의 일부로 사용하는 전략 중 하나이다.
- 이러한 strategy 는 전통적인 통계방법론 대비 베이지안 방법론이 가진 장점이다.