---
{"author":"jx2lee","aliases":"statement 와 expression","created":"2023-12-20T00:33:04.000+09:00","last-updated":"2024-03-03 00:59","tags":["language","common"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":true,"dg-show-backlinks":true,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/__/language/common-statement-expression/","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---



### Statement
- 의역: 자연어의 문장과 거의 동일하다. Statement 는 완전한 실행 단위를 형성한다. 세미콜론(;)으로 Statement 를 종료하여 명령문으로 만들 수 있다.
- `프로그램의 문장으로, 한 개 값으로 도출할 수 없는` 단위이다.
	- e.g `System.out.println("Hello World!");`


### Expression
![|400](https://i.imgur.com/jbyhnPm.png)

- 의역: 변수, 연산자 및 메서드 호출로 구성된 구성체로, 언어구문에 따라 구성되며 단일 값으로 평가한다.
- `프로그램의 한 문장으로 한 개 값으로 표현할 수`  있는 단위이다.
	- e.g `int score = 9*5`


### Difference
- Statement 는 Expression 을 포함하고 있다. (`Expression ⊂ Statement`)
- 값으로 도출할 수 있는 구문을 Expression, 없는 것을 Statement 이다.
- java if-else 는 Statement, 삼항 연산자는 Statement 면서 Expression 이다.
	- kotlin 에서는 if-else 가 Expression 이며 삼항 연산자를 제공하지 않는다.


### Ref
- [https://docs.oracle.com/javase/tutorial/java/nutsandbolts/expressions.html](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/expressions.html)
- 참고하면 좋을만한: https://shoark7.github.io/programming/knowledge/expression-vs-statement