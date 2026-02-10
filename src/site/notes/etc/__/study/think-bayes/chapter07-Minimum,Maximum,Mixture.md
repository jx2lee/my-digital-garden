---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter07-Minimum,Maximum,Mixture/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

이번 장에서는 분포 최소값과 최대값을 계산하고 이를 사용해 정방향 문제와 역방향 문제를 풀어본다.

그런 다음 예측을 할 때 다른 분포가 혼합된 분포를 살펴본다.

먼저 분포 작업을 위해 강력한 도구인 누적 분포 함수(cumulative distribution function)를 살펴보자.

# Culmulative Distribution Functions
지금까지는 확률 질량 함수(pmf)를 사용하여 분포를 표현했다. pmf 대안으로 누적 분포 함수(CDF)를 사용할 수 있다.

BayesianEstimation 의 유로 문제의 posterior 분포를 사용해보자
우리가 사용한 uniform prior 은 다음과 같다.
```python
>>> import numpy as np
>>> from empiricaldist import Pmf
>>> 
>>> hypos = np.linspace(0, 1, 101)
>>> pmf = Pmf(1, hypos)
>>> data = 140, 250
```

update 는 다음과 같다.
```python
from scipy.stats import binom

def update_binomial(pmf, data):
    """Update pmf using the binomial distribution."""
    k, n = data
    xs = pmf.qs
    likelihood = binom.pmf(k, n, xs)
    pmf *= likelihood
    pmf.normalize()

>>> update_binomial(pmf, data)
```

CDF는 PMF의 누적 합계이므로 다음과 같이 계산한다.
```python
>>> cumulative = pmf.cumsum()
>>> cumulative
0.00     0.000000e+00
0.01    1.256330e-207
0.02    5.731921e-166
0.03    8.338711e-142
0.04    8.269265e-125
            ...      
0.96     1.000000e+00
0.97     1.000000e+00
0.98     1.000000e+00
0.99     1.000000e+00
1.00     1.000000e+00
Name: , Length: 101, dtype: float64
```

![](https://i.imgur.com/ppnDYcm.png)

CDF의 범위는 항상 0에서 1 사이다. (*최대값이 임의의 확률이 될 수 있는 PMF와 대조*)

cumsum의 결과는 시리즈이므로 괄호 연산자를 이용해 인덱싱 할 수 있다.
```python
>>> cumulative[0.61]
0.9638303193984255
```

- 0.61 보다 작거나 같을 확률이 총 96% 임을 의미한다.

다른 방법으로, 확률을 조회하고 해당 사분위수를 구하려면 보간법(interpolation)을 사용하면 된다.

```python
>>> from scipy.interpolate import interp1d
>>> 
>>> ps = cumulative.values
>>> qs = cumulative.index
>>> 
>>> interp = interp1d(ps, qs)
>>> interp(0.96)
array(0.60890171)
```

- 이 분포의 96 백분위수는 0.61 이다.

empiricaldist 는 누적 분포 함수를 나타내는 Cdf 클래스를 제공한다. Pmf가 주어지면 다음과 같이 Cdf를 계산할 수 있다.

```python
>>> cdf = pmf.make_cdf()
```

make_cdf는 np.cumsum을 사용하여 확률의 누적 합계를 계산한다.

```python
>>> cdf[0.61]
0.9638303193984255
```

그러나 분포에 없는 수량을 조회하면 KeyError가 발생한다. 이 문제를 방지하려면 괄호를 사용하여 Cdf 를 함수로 호출할 수 있다. 인수가 Cdf에 나타나지 않으면 수량 간에 보간된다.

```python
>>> cdf(0.615)
array(0.96383032)
```

다른 방법으로 quantile 를 사용해 누적 확률을 조회하고 해당 수량을 얻을 수 있다.

```python
>>> cdf.quantile(0.9638303)
array(0.61)
```

Cdf는 또한 주어진 확률을 포함하는 신뢰구간을 계산하는 `credible_interval` 을 제공한다.

```python
>>> cdf.credible_interval(0.9)
array([0.51, 0.61])
```

CDF와 PMF는 동일한 정보를 제공한다는 점에서 동일하며, 언제든지 둘 중 하나에서 다른 것으로 변환할 수 있다. CDF가 주어지면 PMF 를 얻을 수 있다.

```python
>>> pmf = cdf.make_pmf()
```

make_pmf는 np.diff 를 사용하여 연속 누적 확률 간 차이를 계산하여 pmf 를 반환한다. 
  
Cdf 객체가 유용한 이유 중 하나는 사분위수를 효율적으로 계산하기 때문이다. 또 다른 이유는 다음 섹션에서 살펴볼 것처럼 최대값 또는 최소값의 분포를 쉽게 계산할 수 있다.

# Best Three of Four
던전앤드래곤에서 각 캐릭터는 힘, 지능, 지혜, 민첩, 체질, 카리스마 등 여섯 가지 속성을 가지고 있다.

새로운 캐릭터를 생성하려면 각 속성에 대해 6면 주사위 4개를 굴려서 가장 좋은 3개를 합산한다. 예를 들어 힘의 경우 주사위를 굴려서 1, 2, 3, 4가 나오면 캐릭터의 힘은 2, 3, 4의 합 **9**가 된다.

연습 삼아 위 분포를 알아보자. 그런 다음 각 캐릭터에 대해 가장 좋은 속성의 분포를 알아낼 것이다.

이전 장에서 주사위를 굴린 결과를 나타내는 Pmf를 만드는 make_die 함수와 Pmf 객체의 시퀀스를 가져와 그 합의 분포를 계산하는 add_dist_seq 함수를 이용한다.

다음은 6면 주사위와 이에 대한 3개의 참조가 있는 시퀀스를 나타내는 Pmf 다.

```python
>>> from utils import make_die
>>> 
>>> die = make_die(6)
>>> dice = [die] * 3
```

세 주사위 합 분포를 계산한다.
```python
>>> from utils import add_dist_seq
>>> 
>>> pmf_3d6 = add_dist_seq(dice)
```

![](https://i.imgur.com/yXTG7HJ.png)

주사위 4개를 굴려 가장 좋은 3개를 더하는 합계 분포를 계산하는 것은 조금 더 복잡하다. 10,000번 주사위 굴림을 시뮬레이션하여 분포를 추정해 본다.
  
먼저 10,000개 행과 4개 열로 구성된 1에서 6까지의 임의의 값 배열을 만든다.

```python
>>> n = 10000
>>> a = np.random.randint(1, 7, size=(n, 4))
```

각 행에서 상위 결과 세 개를 찾기 위해 행을 오름차순으로 정렬하고 3개 값을 가져온다. (to `t`)
```python
>>> a.sort(axis=1)
>>> t = a[:, 1:].sum(axis=1)
```

pmf 객체로 생성하고 이를 그래프화 해보면 다음과 같다.

```python
>>> pmf_best3 = Pmf.from_seq(t)
```

![](https://i.imgur.com/8mn9woI.png)

4개 중 가장 좋은 3개를 선택하는 것이 더 높은 pmf 값을 생성하는 경향이 있다.  
  
다음으로 4개의 주사위 중 가장 좋은 3개의 합으로 이루어진 최대 6개의 속성에 대한 분포를 구해 본다.

# Maximum
분포의 최댓값 또는 최소값을 계산하려면 누적 분포 함수를 활용할 수 있다. 먼저 4개 값 중 가장 큰 3개의 합을 구한 분포의 Cdf를 계산한다.

```python
>>> cdf_best3 = pmf_best3.make_cdf()
```

Cdf(x)는 x보다 작거나 같은 수량에 대한 확률합이라는 것을 상기해보자. 즉, **분포에서 선택한 임의의 값이 x보다 작거나 같을 확률**이다.

이제 이 분포에서 6개 값을 뽑는다고 가정한다. 6개의 값이 모두 x보다 작거나 같을 확률은 Cdf(x)를 6의 거듭제곱한 값으로, 다음과 같이 계산한다.

```python
>>> cdf_best3**6
3     4.665600e-20
4     7.290000e-16
5     1.779785e-13
6     2.775208e-11
7     7.586503e-10
8     1.285500e-08
9     1.313824e-07
10    5.945878e-07
11    1.613507e-06
12    4.716487e-06
13    6.728533e-06
14    3.963545e-06
15    1.018136e-06
16    1.281003e-07
17    4.962498e-09
18    1.875537e-11
Name: , dtype: float64
```

6개 값 모두 x보다 작거나 같으면, 최대값이 x보다 작거나 같다는 의미이다. 다음과 같이 Cdf 객체로 변환할 수 있다.

```python
>>> from empiricaldist import Cdf
>>> 
>>> cdf_max6 = Cdf(cdf_best3**6)
```

pmf 객체로 변환은 다음과 같다.

```python
>>> pmf_max6 = cdf_max6.make_pmf()
```

![](https://i.imgur.com/7BcGNey.png)

- 대부분의 캐릭터는 12보다 큰 속성을 하나 이상 가지고 있으며, 약 10% 는 18점의 속성을 가지고 있다.

위에서 살펴본 3개 CDF 분포를 그려보자.
![](https://i.imgur.com/NcvEFja.png)

Cdf는 동일한 계산을 수행하는 max_dist를 제공하므로 다음과 같이 Cdf 최대값을 계산할 수도 있다.
```python
>>> cdf_max_dist6 = cdf_best3.max_dist(6)
```

# Minimum
이전 섹션에서는 캐릭터의 최고 속성 분포를 계산했다. 이제 최악의 분포를 계산한다.
  
최솟값 분포를 계산하기 위해 다음과 같이 계산할 수 있는 complementary CDF를 사용한다.

```python
>>> prob_gt = 1 - cdf_best3
```

변수 이름에서 알 수 있듯이 **complementary CDF 는 분포값이 x보다 클 확률**이다. 분포에서 6개의 값을 뽑는 경우 6개가 모두 x를 초과할 확률은 다음과 같다.

```python
>>> prob_gt6 = prob_gt**6
```

6개 속성 모두 X를 초과하면 최소값이 X를 초과한다는 의미이므로 `prob_gt6`은 최소값의 complementary CDF 이다. 즉, 다음과 같이 CDF 최소값을 계산할 수 있다.

```python
>>> prob_le6 = 1 - prob_gt6
>>> # cdf 객체로 변환하고 그래프로 표현해보자
>>> cdf_min6 = Cdf(prob_le6)
```

![](https://i.imgur.com/Xxi9ZoW.png)

Cdf는 동일한 계산을 수행하는 min_dist를 제공하므로 다음과 같이 Cdf 최소값을 구할 수 있다.

```python
>>> cdf_min_dist6 = cdf_best3.min_dist(6)
>>> # min_dist 함수로 구한 분포와 비교하면 차이가 거의 없다.
>>> np.allclose(cdf_min_dist6, cdf_min6)
True
```

# Mixture
이 섹션에서는 혼합된 분포를 계산하는 방법을 소개한다. 몇 가지 간단한 예제를 통해 무엇을 의미하는지 설명한 다음, 이러한 혼합 분포가 예측에 어떻게 사용되는지 살펴본다.

다음은 던전 앤 드래곤에서 영감을 받은 또 다른 예시이다.

> - 캐릭터가 한 손에는 dagger 를, 다른 한 손에는 단검으로 무장했다고 가정해 보겠습니다.
> - 각 라운드 동안 무작위로 선택된 두 무기 중 하나로 몬스터를 공격합니다.  
> - dagger 은 4면 주사위만큼 피해를 입히고, 단검은 6면 주사위만큼 피해를 입힙니다.  
> 
> 각 라운드에서 입히는 피해의 분포는 어떻게 되나요?

이 질문에 답하기 위해 4면 주사위와 6면 주사위를 나타내는 Pmf를 만든다.
```python
>>> d4 = make_die(4)
>>> d6 = make_die(6)
```

1포인트 피해를 입힐 확률을 계산해보자.
- dagger 로 공격했다면 1/4
- 단검으로 공격했다면 1/6
  
두 무기 중 하나를 선택할 확률은 1/2이므로 총 확률(total probability)은 평균(average)이 된다.

```python
>>> prob_1 = (d4(1) + d6(1)) / 2
>>> prob_1
0.20833333333333331
```

숫자 2, 3, 4 경우 확률은 같지만, 5와 6은 4면 주사위로는 불가능한 결과이기 때문에 확률이 다르다.
```python
>>> prob_6 = (d4(6) + d6(6)) / 2
>>> prob_6
0.08333333333333333
```

혼합분포를 계산하기 위해 가능한 결과를 반복하여 그 확률을 계산할 수 있지만, + 연산자를 사용해 동일한 계산을 할 수 있다. 히스토그램으로 표현해보자.
```python
>>> mix1 = (d4 + d6) / 2
```

![](https://i.imgur.com/8fw1yOX.png)

이제 세 마리 몬스터와 싸우고 있다고 가정한다.
- 한 마리는 몽둥이를 가지고 있으며, 4면 주사위로 한 번의 피해를 입힙니다.
- 한 마리는 철퇴를 가지고 있으며, 6면 주사위로 한 번의 피해를 입힙니다.
- 마지막 한 마리는 지팡이를 가지고 있는데, 역시 6면 주사위로 한 번 던져 피해를 입힌다.

근접 공격은 무질서하기 때문에 매 라운드마다 무작위로 선택된 몬스터 중 한 마리의 공격을 받는다. 몬스터가 가하는 피해의 분포를 구하려면 다음과 같이 분포의 가중 평균(weighted average)을 계산하면 된다. 그래프도 함께 그려보자.

```python
>>> mix2 = (d4 + 2*d6) / 3
```

![](https://i.imgur.com/QDZrrdy.png)

이 섹션에서는 분포에 확률을 더하는 + 연산자를 사용했는데, 합계분포를 계산하는 Pmf.add_dist와 혼동하지 마세요.

차이점을 설명하기 위해 Pmf.add_dist를 사용하여 라운드당 총 피해량의 혼합 분포를 계산해보고 그래프를 그려본다.

```python
>>> total_damage = Pmf.add_dist(mix1, mix2)
>>>
>>> total_damage.bar(alpha=0.7)
>>> decorate_dice('Total damage inflicted by both parties')
```

![](https://i.imgur.com/vSRdJXM.png)

# General Mixtures
이전 섹션에서는 임시방편으로 mixture 를 계산했다. 이제 좀 더 일반적인 시선으로 살펴본다.

롤플레잉 게임뿐만 아니라 실제 문제에 대한 예측(prediction)을 생성한다.

전투에 세 마리의 몬스터가 추가로 참여하며, 각 몬스터는 8면 주사위로 한 번의 피해를 입히는 전투 도끼를 가지고 있다고 가정한다. 하지만 한 라운드당 한 몬스터만 무작위로 선택되어 공격하므로, 몬스터가 가하는 피해는 혼합(mixture)된다.
- 4면 주사위 1개
- 6면 주사위 2개
- 8면 주사위 3개

무작위로 선택된 몬스터를 Pmf로 표현해보자.
```python
>>> hypos = [4,6,8]
>>> counts = [1,2,3]
>>> pmf_dice = Pmf(counts, hypos)
>>> pmf_dice.normalize()
>>> pmf_dice
4    0.166667
6    0.333333
8    0.500000
Name: , dtype: float64
```

pmf_dice 분포는 주사위를 굴릴 면의 수와 각 면을 굴릴 확률을 나타낸다. 예를 들어, 여섯 몬스터 중 한 마리가 dagger 를 가지고 있다면 확률은 $\frac{1}{6}$ 이므로 4면 주사위를 굴릴 확률은 $\frac{1}{6}$ 이다.
  
다음으로 주사위를 표현하기 위한 Pmf 객체를 생성한다.
```python
>>> dice = [make_die(sides) for sides in hypos]
>>> dice
[1    0.25
2    0.25
3    0.25
4    0.25
Name: , dtype: float64, 1    0.166667
2    0.166667
3    0.166667
4    0.166667
5    0.166667
6    0.166667
Name: , dtype: float64, 1    0.125
2    0.125
3    0.125
4    0.125
5    0.125
6    0.125
7    0.125
8    0.125
Name: , dtype: float64]
```

혼합분포를 계산하기 위해 pmf_dice 확률을 가중치로 사용하여 주사위의 가중 평균을 계산한다.(*간결하게 표현하려면 분포를 Pandas DataFrame 에 넣는 것이 좋다*)

```python
>>> import pandas as pd
>>> pd.DataFrame(dice)
1         2         3         4         5         6      7      8
0.250000  0.250000  0.250000  0.250000       NaN       NaN    NaN    NaN
0.166667  0.166667  0.166667  0.166667  0.166667  0.166667    NaN    NaN
0.125000  0.125000  0.125000  0.125000  0.125000  0.125000  0.125  0.125
```

결과는 각 분포의 가능한 각 결과에 대한 하나의 열이 있는 데이터 프레임이다. 모든 행의 길이가 같지 않기 때문에 Pandas는 여분의 공백을 "숫자가 아님"을 나타내는 특수 값 NaN으로 채운다. fillna를 사용하여 NaN 값을 0으로 바꿀 수 있다.  
  
다음 단계는 각 행에 pmf_dice의 확률을 곱하는 것인데, 열을 따라 분포가 흐르도록 행렬을 바꾸면 더 쉽다.
```python
>>> df = pd.DataFrame(dice).fillna(0).transpose()
>>> df
1  0.25  0.166667  0.125
2  0.25  0.166667  0.125
3  0.25  0.166667  0.125
4  0.25  0.166667  0.125
5  0.00  0.166667  0.125
6  0.00  0.166667  0.125
7  0.00  0.000000  0.125
8  0.00  0.000000  0.125
```

가중치(pmf_dice)를 이용해 가중 평균을 계산해보자.
```python
>>> df *= pmf_dice.ps
>>> df
1  0.041667  0.055556  0.0625
2  0.041667  0.055556  0.0625
3  0.041667  0.055556  0.0625
4  0.041667  0.055556  0.0625
5  0.000000  0.055556  0.0625
6  0.000000  0.055556  0.0625
7  0.000000  0.000000  0.0625
8  0.000000  0.000000  0.0625
```

그리고 가중치 분포를 합산한다.
```python
>>> df.sum(axis=1)
1    0.159722
2    0.159722
3    0.159722
4    0.159722
5    0.118056
6    0.118056
7    0.062500
8    0.062500
dtype: float64
```

위 과정들을 한번에 계산하기 위한 함수를 다음과 같이 정의한다.
```python
def make_mixture(pmf, pmf_seq):
    """Make a mixture of distributions."""
    df = pd.DataFrame(pmf_seq).fillna(0).transpose()
    df *= np.array(pmf)
    total = df.sum(axis=1)
    return Pmf(total)

>>> mix = make_mixture(pmf_dice, dice)
```

![](https://i.imgur.com/Oj79qNN.png)

`make_mixture`가 간결하고 효율적이라서 Pandas 를 사용했다. 이 장의 마지막에 있는 연습에서는 혼합물을 가지고 연습해 볼 수 있으며, 다음 장에서 make_mixture를 다시 사용할 예정이다.
