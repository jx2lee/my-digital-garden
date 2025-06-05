---
{"dg-publish":true,"author":"jx2lee","aliases":"Split loop","dg-home-link":false,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":false,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/__/refactoring/split-loop/","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
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