---
{"dg-publish":true,"permalink":"/etc/__/refactoring/split-loop/","dgShowBacklinks":true,"dgShowLocalGraph":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


![](https://i.imgur.com/clOlDTS.png)

```java
double averageAge = 0;  
double totalSalary = 0;  
for (int i = 0; i < people.length; i++) {  
	averageAge += people[i].age;  
	totalSalary += people[i].salary;  
}  
averageAge = averageAge / people.length;
```

- problem
	- 하나의 반복문에 두가지 작업을 한다.
- solution
	- 하나의 반복문에는 하나의 작업을 하도록 반복문을 나눈다.

```java
double averageAge = 0;  
for (int i = 0; i < people.length; i++) {  
	averageAge += people[i].age;  
	totalSalary += people[i].salary;  
}

double totalSalary = 0;  
for (int i = 0; i < people.length; i++) {  
	averageAge += people[i].age;  
	totalSalary += people[i].salary;  
}

averageAge = averageAge / people.length;
```

# reference
[https://refactoring.com/catalog/splitLoop.html](https://refactoring.com/catalog/splitLoop.html)