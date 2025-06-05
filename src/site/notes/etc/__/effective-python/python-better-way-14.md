---
{"dg-publish":true,"permalink":"/etc/__/effective-python/python-better-way-14/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


함수를 이용해 None을 반환하기보다는 예외를 일으키는 방법에 대해 알아본다.
파이썬 프로그래머들은 보통 None값에 특별한 의미를 부여하는 경우가 있다. 예를 들어 나눗셈을 수행하는 헬퍼함수를 생각해보자. 0으로 나누는 경우는 존재하지 않기 때문에 None을 반환하는 게 자연스럽다.

```python
def divide(a,b):
  try:
    return a / b
  except ZeroDivisionError:
    return None
```

반환 값을 다음과 같이 해석할 수 있다.

```python
result = divide(x, y)
if result is None:
  print('Invalid inputs')
```

하지만 `분자가 0, 즉 나누는 숫자를 0으로 한다면?`
이런 경우 if문으로 결과를 평가할 때 문제가 될 수 있다.조건을 None이 아닌 False로 검사할 수 있기 때문이다.

```python
x, y = 0, 5
result = divide(x, y)
if no result:
  print('Invalid inputs') # wrong!
```

위 예는 `None에 특별한 의미를 부여할 때` 파이썬 코드에서 흔히 하느 실수라고 한다.<br>
이러한 실수를 방지하기 위한 방법은 두 가지로 설명한다.


# 반환 값을 두 개로 나눠 튜플에 담자


튜플의 첫 value는 성공/실패 여부를 알려준다. 두 번째 값은 계산된 실제 결과다.

```python
def divide(a, b):
  try:
    return True, a / b
  except ZeroDivisionError:
    return False, None

success, result = divide(x, y)
if no success:
  print('Invalid inputs')
```

허나 이 방법은 튜플 첫 값을 쉽게 무시할 수 있다(`가령 _을 이용해 무시 가능`).<br>
겉보기에느 잘못된 것 같지 않지만 `그냥 None을 반환하는 것만큼 나쁘다.`

```python
_, result = divide(x, y)
if no result:
  print('Invalid inputs')
```

# None을 반환하지 말자! -> 호출 함수에서 예외 일으키기

**None을 반환하지 않는다**. 즉, **호출하는 함수 내에서 예외를 일으키는 방법을 사용한다.**(`ZeroDivisionError -> ValueError`)

```python
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('Invalid inputs') from e
```

위와 같이 함수를 정의하면 함수의 반환 값을 `조건식으로 검사할 필요가 없다.`

```python
x, y = 0, 10
try:
    result = divide(x, y)
except ValueError:
    print('Invalid inputs')
else:
    print('Result is %.1f' % result)
```

# summary
- None을 반환하는 함수는 None 이나 다른 값이 조건식에서 False로 평가되기 때문에 쉽게 오류를 범할 수 있다.
- 특별한 상황을 알릴 때는 None 대신 예외를 일으키자.

# reference
[Python better way](http://www.yes24.com/Product/goods/25138160)