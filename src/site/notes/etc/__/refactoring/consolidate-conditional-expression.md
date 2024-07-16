---
{"dg-publish":true,"permalink":"/etc/__/refactoring/consolidate-conditional-expression/","noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


TL;DR: 같은 결과를 내는 조건식이 여러개인 경우, 하나의 조건식으로 합친다.

### before
![](https://i.imgur.com/LwSosuj.png)

```javascript
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```

### after
```javascript
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
	return ((anEmployee.seniority < 2)
			|| (anEmployee.monthsDisabled > 12)
			|| (anEmployee.isPartTime));
}
```
