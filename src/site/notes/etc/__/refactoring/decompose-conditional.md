---
{"dg-publish":true,"permalink":"/etc/__/refactoring/decompose-conditional/","noteIcon":""}
---


![](https://i.imgur.com/rAb4vHw.png)
```java
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd)) {
  charge = quantity * plan.summerRate;
  }
else {
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
}
```

- if then else
- 조건(if), then, else 가 길어지면 함수로 추출한다.

```java
if (Summer(date)) {
    charge = summerCharge(quantity);
}
else {
	charge = winterCharge(quantity);
}
```

# reference
[https://www.refactoring.com/catalog/decomposeConditional.html](https://www.refactoring.com/catalog/decomposeConditional.html)