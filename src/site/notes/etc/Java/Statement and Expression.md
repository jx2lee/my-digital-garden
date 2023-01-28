---
{"dg-publish":true,"permalink":"/etc/Java/Statement and Expression/","dgPassFrontmatter":true}
---


# Statement
- 의역: 자연어의 문장과 거의 동일하다. Statement 는 완전한 실행 단위를 형성한다. 세미콜론(;)으로 Statement 를 종료하여 명령문으로 만들 수 있다.
- `프로그램의 문장으로, 한 개 값으로 도출할 수 없는` 단위이다.
	- e.g `System.out.println("Hello World!");`
# Expression
- 의역: 변수, 연산자 및 메서드 호출로 구성된 구성체로, 언어구문에 따라 구성되며 단일 값으로 평가됩니다.
- `프로그램의 한 문장으로 한 개의 값으로 표현할 수`  있는 단위이다.
	- e.g `int score = 9*5`
# **Difference**
- Statement 는 Expression 을 포함하고 있다.
	- `Expression ⊂ Statement`
- 한 개 값으로 도출할 수 있는 구문을 Expression, 없는 것을 Statement 라고 이해하자.
- java if-else 는 Statement, 삼항 연산자는 Statement 면서 Expression 이다.
	- kotlin 에서는 if-else 가 Expression 이며 삼항 연산자를 제공하지 않는다.
# Ref
- [https://docs.oracle.com/javase/tutorial/java/nutsandbolts/expressions.html](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/expressions.html)