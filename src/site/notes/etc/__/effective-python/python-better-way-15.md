---
{"title":"클로저가 변수 스코프와 상호 작용하는 방법","tags":["Python Better Way"],"categories":"Python","date":"2019-08-08","dg-publish":true,"permalink":"/etc/__/effective-python/python-better-way-15/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


숫자 list를 정렬할 때 일정 숫자들을 먼저 정렬하고자 한다. 이는 `사용자 인터페이스를 표현` 하거나, `중요 메세지 또는 예외 이벤트를 먼저 보여줄 때` 유용하다.
일반적인 방법으로는 list의 sort method 에 helper function 을 key 로 넘기는 것이다. helper function의 return 값은 숫자를 정렬하는 사용된다

```python
def sort_priority(values, group):
  def helper(x):
    if x in group:
      return (0, x)
    return (1, x)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)

>>>
[2, 3, 5, 7, 1, 4, 6, 8]
```

위 함수가 동작하는 이유는 다음과 같다.
- **python은 closure를 지원한다.
	- closure란 자신이 정의된 스코프에 있는 변수를 참조하는 함수다.
- 이 때문에 **helper function이 sort_priority의 group에 접근**할 수 있다.

# reference
[파이썬 코딩의 기술](http://www.yes24.com/Product/goods/25138160)