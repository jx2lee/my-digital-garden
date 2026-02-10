---
{"dg-publish":true,"permalink":"/etc/__/study/think-bayes/chapter01-Probability/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---

#think-bayes #probability #study

---

이 장에서는 조건부 확률(conditional probability)를 시작으로 베이즈 정리(Bayes's Theorem)를 도출하고 실제 데이터를 사용하여 이를 증명한다.

# Linda the Banker
> Linda는 31세, 미혼이고 솔직하며 매우 밝습니다. 철학을 전공했습니다. 학생 시절에는 차별과 사회 정의 문제에 깊은 관심을 가졌고, 반핵 시위에도 참여했습니다. 어느 쪽이 더 가능성이 높을까?
> 
> - (A) 린다 씨는 은행원입니다.
> - (B) 린다 씨는 은행원이며 페미니스트 운동에 적극적입니다.  
  
- 조건부 확률을 설명하기 위한 [가장 좋은 예시](https://en.wikipedia.org/wiki/Conjunction_fallacy)
- 많은 사람들이 두 번째 답을 선택하는데, 아마도 설명과 더 일관성이 있어 보이기 때문일 것이다.
	- 린다를 그냥 은행원이라고 하면 특색이 없어 보이고, 페미니스트라고 하는 것이 일관성 있어 보인다.
- 그러나 두 번째 답변은 질문에서 묻는 것처럼 "더 가능성이 높다"고 할 수 없다.
	- 31세/미혼/솔직/매우밝음/철학전공/학생시절에 부합하는 사람이 1000명이고 그 중 10명이 은행원으로 일한다고 가정한다.
	- 그중 기껏해야 10명 모두 페미니스트일 것이다.
		- 이 경우 두 옵션(A/B)의 확률은 같다.
		- 10명 미만이면 두 번째 옵션의 확률은 낮다. 하지만 두 번째 옵션의 확률이 더 높을 수는 없음
  
> 두 번째 옵션을 선택하려는 경향이 있다면, 당신은 좋은 회사에 있는 것입니다. 생물학자 스티븐 J. 굴드
  
# Probability
- 정의: 확률은 유한 집합의 일부(fraction of a finite set)
	- 예를 들어, 1000명을 대상으로 설문조사를 했는데 그 중 20명이 은행 창구 직원이라면 은행 창구 직원으로 일하는 비율은 0.02 또는 2% 이다.
	- 모집단에서 무작위로 한 사람을 선택하면 그 사람이 은행원이 될 확률은 2% 이다.
	- "무작위로"라는 의미는 데이터 집합의 모든 사람이 선택될 확률이 동일하다는 것을 의미한다.

예시에 대한 내용은 다음과 같다.

```python
import pandas as pd

gss = pd.read_csv('gss_bayes.csv', index_col=0)
gss.head()
```

![](https://i.imgur.com/y1QXXKh.png)
- column
	- polviews: 진보에서 보수에 이르는 다양한 정치적 견해
	- partyid: 정당 소속, 민주당, 무소속 또는 공화당
	- indus10: 응답자가 근무하는 산업코드

# Fraction of Bankers
- 은행과 관련된 산업코드는 6970
- `banker = (gss['indus10'] == 6870)`
	- `banker.sum()` -> 728
- 은행원 비율을 계산하려면 평균 함수(mean)를 사용하여 계산할 수 있다.
	- `banker.mean()`
	- 0.014...
- 데이터 셋에서 무작위로 사람을 선택하면 은행원일 확률은 약 1.5% 이다.

# The Probability Function
```python
def prob(A):
    """Computes the probability of a proposition, A."""    
    return A.mean()
```

- 은행원일 확률은 `prob(banker)` 으로 계산 가능하다.
- 여성일 확률(2)은 `female = (gss['sex'] == 2); prob(female)` 과 같이 계산할 수 있다.
	- 이 데이터 세트에서 여성의 비율이 미국 성인 인구보다 높은 이유는 GSS에 교도소나 군용 주택과 같은 시설에 거주하는 인구가 포함되지 않고 이러한 인구는 남성일 가능성이 더 높기 때문이다.

# Political Views and Parties
- polviews 카테고리는 7 척도로 표시할 수 있다.
	```
	1	Extremely liberal
	2	Liberal
	3	Slightly liberal
	4	Moderate
	5	Slightly conservative
	6	Conservative
	7	Extremely conservative
	```
- '매우 자유주의적', '자유주의적' 또는 '약간 자유주의적'이라고 응답한 사람은 '자유주의적'을 True로 정의하도록 한다.
- `liberal = (gss['polviews'] <= 3)`
	- 약 27%
- 마찬가지로 partyid 는 다음과 같이 인코딩 되어 있다.
	```
	0	Strong democrat
	1	Not strong democrat
	2	Independent, near democrat
	3	Independent
	4	Independent, near republican
	5	Not strong republican
	6	Strong republican
	7	Other party
	```
- '강한 민주주의자' 또는 '강한 민주주의자가 아님'을 선택한 응답자를 포함하도록 민주주의자를 정의하도록 한다.
- `democrat = (gss['partyid'] <= 1)`
	- 약 36%

# Conjunction
- conjunction: `and` 연산자의 또다른 이름이다.
	- conjunction A and B 는 A 와 B 가 모두 True 인 경우 True 이고 나머지는 모두 False 이다.
- 은행원이면서 민주당원일 확률 -> `prob(banker & democrat)`
	- 0.004686...
- A and B == B and A

# Conditional Probability
- 정의: 조건에 따라 달라지는 확률를 나타낸다.
- 3가지 예시를 들며 조건부확률을 계산하는 과정을 설명한다.
	- What is the probability that a respondent is a Democrat, given that they are liberal?
	- What is the probability that a respondent is female, given that they are a banker?
	- What is the probability that a respondent is liberal, given that they are female?
- 조건부확률을 계산하는 함수는 다음과 같다.
	```python
	def conditional(proposition, given):
	    """Probability of A conditioned on given."""
	    return prob(proposition[given])
	```

# Conditional Probability Is Not Commutative
- prob(A & B) 와 prob(B & A) 는 동일하다.
- conditional(A, B) != conditional(B, A)

# Condition and Conjunction
- 조건부확률과 conjunction 결합이 가능하다.
- 자유민주주의자라고 가정할 때 응답자가 여성일 확률은 `conditional(female, given=liberal & democrat)` 와 같이 계산할 수 있다.
	- 약 57% 자유민주주의자가 여성을 의미한다.

# Law of Probability
- conjunction & 조건부 확률에 대한 다음 3가지 정리를 살펴본다.
	- 정리 1: conjunction 를 사용하여 조건부 확률 계산
	- 정리 2: 조건부 확률을 사용하여 conjunction 계산
	- 정리 3: 조건부 확률(A, B)를 사용하여 조건부 확률(B, A) 계산

# Theorem 1
$$P(A|B) = \frac{P({A\,and\,B})}{P(B)}$$

# Theorem 2
$$P(A\,and\,B) = P(A|B)\,P(B)$$

# Theorem 3
- `P(A and B) = P(B and A)` `P(B)*P(A|B) = P(A and B)` 임을 이용해 마지막 정리를 다음과 같이 나타낼 수 있다.
$$P(A|B) = \frac{P(A)\,P(B|A)}{P(B)}$$
- 베이지안 정리
- 고로 위에서 은행원일 때 자유주의자인 확률(`conditional(liberal, given=banker)`)은 다음과 같이 계산할 수 있다.
	- `prob(liberal) * conditional(banker, liberal) / prob(banker)`

# The Law of Total Probability
$$P(A) = P(B_{1}\,and A) + P(B_{2}\,and A)$$
- 단 조건이 있다.
	- 상호 배타적(mutually exclusive): 즉 둘 중 하나만 참일 수 있다
	- 총체적 배타성(collectively exhaustive): 둘 중 하나만 참이어야 함을 의미한다.
- `prob(banker)` 값을 `prob(male & banker) + prob(female & banker)` 와 동일한지 확인한다.
- male 과 female 은 상호 태바적이며 총체적 베타성이다. (MECE)

`P(B1 and A)` 와 `P(B2 and A)` 를 정리 2를 이용해 다음과 같이 나타낼 수 있다.
$$P(A)=P(B_{1})P(A|B_{1}) + P(B_{2})P(A|B_{2})$$

만약 성별처럼 두 종류로 나뉘어지지 않는 여러 개일 경우 다음과 같이 total probability 를 구할 수 있다.
$$P(A)=\sum_{i}P(B_{i})P(A|B_{i})$$
- 단 B 는 mutually exclusive 하고 collectively exhaustive 해야한다.

# Summary
3개 정리와 total probability
$$P(A|B) = \frac{P({A\,and\,B})}{P(B)}$$
$$P(A\,and\,B) = P(A|B)\,P(B)$$
$$P(A|B) = \frac{P(A)\,P(B|A)}{P(B)}$$
$$P(A)=\sum_{i}P(B_{i})P(A|B_{i})$$
- 이 시점에서 "그래서 뭐?"라고 질문할 수 있다.
- 모든 데이터가 있다면 원하는 모든 확률, 결합 또는 조건부 확률을 계산할 수 있으며, 계산하는 것만으로도 가능하다. 이러한 공식을 사용할 필요가 없다. (~~하지만 나는 이러한 정리를 살펴보고 싶은 마음에 스터디를 신청했다.~~)
	- 모든 데이터를 가지고 있다면 가능하다. 하지만 그렇지 않은 경우가 많기 때문에 이러한 공식, 특히 베이즈 정리가 매우 유용할 수 있다.

# reference
- [Probability](http://allendowney.github.io/ThinkBayes2/chap01.html)