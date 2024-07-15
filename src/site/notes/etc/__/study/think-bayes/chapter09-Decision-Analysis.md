---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter09-Decision-Analysis/","noteIcon":""}
---

#think-bayes #probability #study

---

이 장에서는 게임 쇼 '더 프라이스 이즈 라이트'에서 영감을 얻은 문제를 플어본다. (베이지안 의사 결정 분석이라는 유용한 프로세스와 함께)  
  
이전 예제에서와 마찬가지로 데이터와 사전 분포를 사용하여 사후 분포를 계산한 다음, 사후 분포를 사용하여 입찰이 포함된 게임에서 최적의 전략을 선택한다.
  
커널 밀도 추정(KDE)을 사용하여 사전 분포를 추정하고 정규 분포를 사해 확률을 계산한다.
  
# The Price Is Right Problem
> 2007년 11월 1일, 레티아와 나다니엘이라는 참가자가 미국 텔레비전 게임 쇼인 '더 프라이스 이즈 라이트'에 출연했습니다. 이들은 "쇼케이스"라는 게임에서 경쟁을 펼쳤는데, 이 게임의 목적은 상품 컬렉션의 가격을 맞추는 것이었습니다. 오버하지 않고 실제 가격에 가장 근접한 참가자가 상품을 획득합니다.  
> 
> 나다니엘이 1등을 차지했습니다. 그의 쇼케이스에는 식기 세척기, 와인 캐비닛, 노트북 컴퓨터, 자동차가 포함되어 있었습니다. 그는 26,000달러를 입찰했습니다.  
> 
> 레티아의 쇼케이스에는 핀볼 머신, 비디오 아케이드 게임기, 당구대, 바하마 크루즈 여행권이 포함되어 있었습니다. 그녀는 \$21,500을 입찰했습니다.  
> 
> 나다니엘의 쇼케이스의 실제 가격은 \$25,347이었습니다. 그의 입찰가가 너무 높았기 때문에 그는 낙찰되었습니다.


- 참가자들은 상품을 보기 전에 쇼케이스의 가격에 대해 어떤 prior 믿음을 가져야 할까요?
- 상품을 본 후 참가자들은 어떻게 prior 를 업데이트해야 할까?
- 사후 분포에 따라 참가자는 얼마에 입찰해야 하나?
	- 의사 결정 분석(decision analysis)

# The Prior
```python
>>> df.head(3)
   Showcase 1  Showcase 2    Bid 1    Bid 2  Difference 1  Difference 2
0     50969.0     45429.0  42000.0  34000.0        8969.0       11429.0
1     21901.0     34061.0  14000.0  59900.0        7901.0      -25839.0
2     32815.0     53186.0  32000.0  45000.0         815.0        8186.0
```
- 처음 두 열인 쇼케이스 1과 쇼케이스 2는 달러로 표시된 물건값
- 다음 두 열은 참가자가 제시한 입찰가
- 마지막 두 열은 실제 값과 입찰가의 차이

# Kernel Density Estimation
이 샘플을 사용하여 상품가격의 prior 분포를 추정할 수 있다. 이를 수행하는 방법은 [커널 밀도 추정(KDE)](https://mathisonian.github.io/kde/)으로, 샘플을 사용하여 평활 분포(smooth distribution)를 추정하는 것이다.

> kernel density estimation 은 kernel 이라는 함수(원점을 중심으로 대칭이며 적분값이 1)로 밀도를 추정하는 방법 중 하나이다.
> - https://blog.mathpresso.com/mathpresso-%EB%A8%B8%EC%8B%A0-%EB%9F%AC%EB%8B%9D-%EC%8A%A4%ED%84%B0%EB%94%94-14-%EB%B0%80%EB%8F%84-%EC%B6%94%EC%A0%95-density-estimation-38fd7ef729bb
> - https://en.wikipedia.org/wiki/Kernel_density_estimation

다음 함수는 샘플을 가져와 KDE를 만들고, 주어진 수량 시퀀스(qs)에서 평가한 후 결과를 정규화된 PMF로 반환한다.
```python
from scipy.stats import gaussian_kde
from empiricaldist import Pmf

def kde_from_sample(sample, qs):
    """Make a kernel density estimate from a sample."""
    kde = gaussian_kde(sample)
    ps = kde(qs)
    pmf = Pmf(ps, qs)
    pmf.normalize()
    return pmf

import numpy as np

>>> qs = np.linspace(0, 80000, 81)
>>> prior1 = kde_from_sample(df['Showcase 1'], qs)
```

![](https://i.imgur.com/cwozuUs.png)

![](https://i.imgur.com/Y7HIQKb.png)

# Distribution of Error
생각해볼거리
- 어떤 데이터를 고려해야하나?
- 각 가설 금액에 대해 conditional likelihood 를 계산할 수 있을까?

이러한 질문에 답하기 위해 각 참가자를 오차 특성이 알려진 가격 추측 도구로 모델링한다. 이 모델에서는 참가자가 상품을 볼 때 각 상품의 가격을 추측하고 가격을 합산한다. 이를 총 guess 라고 할 것이다.
  
이제 우리가 답해야 할 질문은 "실제 가격이 price 일 경우 참가자의 추측이 맞을 likelihood 는 얼마인가?" 이다.

error = guess - price 로 정의하면 "참가자의 추측이 error 만큼 빗나갈 likelihood 는 얼마인가"
  
데이터 세트 각 쇼케이스에서 참가자의 입찰가와 실제 가격의 차이를 살펴본다.

```python
>>> sample_diff1 = df['Bid 1'] - df['Showcase 1']
>>> sample_diff2 = df['Bid 2'] - df['Showcase 2']
>>> 
>>> qs = np.linspace(-40000, 20000, 61)
>>> kde_diff1 = kde_from_sample(sample_diff1, qs)
>>> kde_diff2 = kde_from_sample(sample_diff2, qs)
```

![](https://i.imgur.com/s9afIWb.png)

입찰가가 너무 높은 것보다 너무 낮은 경우가 더 많은 것 같은데, 이는 당연한 일이다. 게임 규칙에 따라 과다 입찰하면 패배하기 때문 🔥
 
정규분포 형태로 모델링되어 평균과 포준편차로 요약이 가능하다.

```python
mean_diff1 = sample_diff1.mean()
std_diff1 = sample_diff1.std()

print(mean_diff1, std_diff1)
-4116.3961661341855 6899.909806377117
```

이러한 error 를 사용하여 참가자의 오류 분포를 모델링할 수 있다. 몇 가정이 필요하다.
- 참가자는 전략적으로 입찰하기 때문에 저가로 입찰하며, 평균적으로 추측값은 평균적으로 정확하다고 가정한다. 즉, 참가자들의 error 평균은 0
- 그러나 차이의 분포와 error 분포는 동일하다고 가정한다. 따라서 차이의 표준 편차를 error 의 표준 편차로 사용한다.

이러한 가정을 바탕으로 매개 변수 0과 std_diff1을 사용하여 정규 분포를 만든다.

```python
from scipy.stats import norm

error_dist1 = norm(0, std_diff1)
```

error 가 -100 인 pdf 값은 다음과 같다.
```python
error = -100
error_dist1.pdf(error)
5.781240564008691e-05
```

# Update
- 당신은 1번 참가자
- 물건들을 보고 총 가격을 23,000 달러라고 추측했다.

```python
guess1 = 23000
error1 = guess1 - prior1.qs # error = guess - price
likelihood1 = error_dist1.pdf(error1)

posterior1 = prior1 * likelihood1
posterior1.normalize()
```

![](https://i.imgur.com/FGDbv42.png)

- posterior 는 살짝 왼쪽으로 이동했다

```python
prior1.mean(), posterior1.mean()
(30299.488817891375, 26192.024002392536)
```

23,000 으로 추측한 후 update 한 결과 26,000 으로 예상하게 되었다.

> 위와 동일한 방법으로 문제 9-2 를 풀 수 있다. error_dist2 로 갈아끼우면 된다.

# Probability of Winning
player 1/2 의 posterior 를 구했으니 전략을 고민해본다.

먼저 플레이어 1의 관점에서 플레이어 2가 초과 입찰할 확률을 계산해본다.

```python
def prob_overbid(sample_diff):
    """Compute the probability of an overbid."""
    return np.mean(sample_diff > 0)

prob_overbid(sample_diff2)
0.29073482428115016
```

이제 플레이어 1이 \$5000만큼 적게 입찰한다고 가정해보자. 플레이어 2가 더 많이 적게 입찰할 확률은 얼마일까?

```python
def prob_worse_than(diff, sample_diff):
    """Probability opponent diff is worse than given diff."""
    return np.mean(sample_diff < diff)

prob_worse_than(-5000, sample_diff2)
0.38338658146964855
```

입찰가와 실제 가격의 차이가 주어졌을 때 플레이어 1이 이길 확률을 계산할 수 있다. (위 두개 함수의 결과값이 확률을 더하면 됨)

```python
def compute_prob_win(diff, sample_diff):
    """Probability of winning for a given diff."""
    # if you overbid you lose
    if diff > 0:
        return 0
    # if the opponent overbids, you win
    p1 = prob_overbid(sample_diff)
    # or of their bid is worse than yours, you win
    p2 = prob_worse_than(diff, sample_diff)
    # p1 and p2 are mutually exclusive, so we can add them
    return p1 + p2

compute_prob_win(-5000, sample_diff2)
0.6741214057507987
```

다양한 차이에 대한 확률을 살펴 보자.
```python
xs = np.linspace(-30000, 5000, 121)
ys = [compute_prob_win(x, sample_diff2) 
      for x in xs]
```

![](https://i.imgur.com/KG1w2H5.png)

# Decision Analysis
이전 섹션에서는 특정 금액만큼 저가로 입찰했다는 가정 하에 낙찰 확률을 계산했다.

실제로 참가자는 실제 가격을 모르기 때문에 자신이 얼마나 과소 입찰했는지 알 수 없다.

하지만 실제 가격에 대한 자신의 믿음을 나타내는 posterior 분포를 가지고 있으며, 이를 사용하여 주어진 입찰가로 낙찰될 확률을 추정할 수 있다.

```python
def total_prob_win(bid, posterior, sample_diff):
    """Computes the total probability of winning with a given bid.

    bid: your bid
    posterior: Pmf of showcase value
    sample_diff: sequence of differences for the opponent
    
    returns: probability of winning
    """
    total = 0
    for price, prob in posterior.items():
        diff = bid - price
        total += prob * compute_prob_win(diff, sample_diff)
    return total
```

$$P(win) = \sum_{price} P(price) ~ P(win ~|~ price)$$

1번 참가자가 25,000 을 입찰하고 posterior1 을 사용할 때 이길 확률은 다음과 같다.

```python
total_prob_win(25000, posterior1, sample_diff2)
0.4842210945439812
```

각 입찰가격 별 이길 확률을 구해보자.
```python
bids = posterior1.qs

probs = [total_prob_win(bid, posterior1, sample_diff2) 
         for bid in bids]

prob_win_series = pd.Series(probs, index=bids)
0.0        2.971297e-01
1000.0     2.983131e-01
2000.0     2.998251e-01
3000.0     3.017554e-01
4000.0     3.042448e-01
               ...     
76000.0    9.298362e-30
77000.0    8.740531e-32
78000.0    6.621979e-34
79000.0    4.043155e-36
80000.0    1.982049e-38
Length: 81, dtype: float64
```

![](https://i.imgur.com/IKVgMpV.png)

```python
prob_win_series.idxmax()
21000.0
prob_win_series.max()
0.6136807192359472
```

23,000 으로 추측하고 posterior1 을 구했다. posterior1 의 평균은 약 26,000 이고 이길 확률이 가장 높은 입찰가는 21,000 이다.

# Maximizing Expected Gain

💥 이기는 것이 전부는 아니다
입찰가가 250달러 이하로 낮으면 모두 낙찰될 수 있다. 따라서 입찰가를 약간 높이는 것이 좋다. 입찰가를 높이면 초과 입찰하여 낙찰될 확률이 높아지지만 두 쇼케이스 모두 낙찰될 확률도 높아진다.  
  
```python
def compute_gain(bid, price, sample_diff):
    """Compute expected gain given a bid and actual price."""
    diff = bid - price
    prob = compute_prob_win(diff, sample_diff)

    # if you are within 250 dollars, you win both showcases
    if -250 <= diff <= 0:
        return 2 * price * prob # 2 게임 모두 이기니까 곱절
    else:
        return price * prob # 
```

실제 금액이 35,000 이고 30,000 을 부른 경우 평균 23,600 이익을 낼 수 있다.
```python
compute_gain(30000, 35000, sample_diff2)
23594.249201277955
```

금액 별 가중치를 두어 계산해보자
```python
def expected_gain(bid, posterior, sample_diff):
    """Compute the expected gain of a given bid."""
    total = 0
    for price, prob in posterior.items():
        total += prob * compute_gain(bid, price, sample_diff)
    return total

expected_gain(21000, posterior1, sample_diff2)
16923.59933856512
```

입찰가 별 예상 수익을 그래프로 그려본다.

```python
bids = posterior1.qs

gains = [expected_gain(bid, posterior1, sample_diff2) for bid in bids]

expected_gain_series = pd.Series(gains, index=bids)
0.0        7.767439e+03
1000.0     7.793931e+03
2000.0     7.827968e+03
3000.0     7.871440e+03
4000.0     7.927339e+03
               ...     
76000.0    1.407195e-24
77000.0    1.341315e-26
78000.0    1.030105e-28
79000.0    6.373663e-31
80000.0    3.171278e-33
Length: 81, dtype: float64

expected_gain_series.idxmax()
22000.0
```

![](https://i.imgur.com/ce6Nc4z.png)

**23,000 이라고 추측한 것을 데이터로 하여 우승 확률이 가장 높은 입찰가는 21,000, 수익을 최대하 하는 입찰가는 22,000 이다.**

# EOD
이번장을 통해 저자가 하고 싶은 말을 한 줄로 정리했다.
> *베이지안 추정은 고전적 방법인 빈도주의와 달리 의사결정 할 수 있는 정보를 제공한다*
