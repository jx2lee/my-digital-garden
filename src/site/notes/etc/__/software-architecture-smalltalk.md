---
{"author":"jx2lee","aliases":"소프트웨어 아키텍처 세미나","created":"2023-12-20T00:33:04.000+09:00","last-updated":"2023-08-31 15:05","tags":null,"dg-publish":true,"permalink":"/etc/__/software-architecture-smalltalk/","dgPassFrontmatter":true,"noteIcon":""}
---


> [!tldr]
> Software Architecture 세미나 잘 듣고 포인트 잘 정리해보자

**chapter 1 - 저번주 복습**

---


- software architecture
    - perspective, structures, relationships, and views
    - architecture pattern > tactics (e.g heartbeat, ping...) > design pattern
    - requirements: 개집보단 빌딩짓는데 아키텍처를 설계하는것이 좋음
    - 그래서 왜 필요한데
    - architect vs developer
        - the ivory tower

**chapter 2**

---

- architecture drivers (quality attribute requirements)
- reproducible, 정량적 수치가 중요해요.
- availability
- testability
- Usability
- QnA
    - is cost a quality attriute? &rarr; 둘 다 될 수 있다.
- 다음 시간은 ... 
    - tactics 다룰 예정

**chapter 3**


---


![](https://i.imgur.com/iv99NLE.png)
- 장표에서 개선되어야할 부분
    - Response measure 가 2개이다.
        - measure 를 하나로 정하고 2개 표가 존재해야한다.
    - IoT product, IoT system 등 모호한 용어가 존재한다.


**chapter 4**

---

view
- perspective, 관점
- 시스템 요소들과 그 사이 관계들을 묶어 표현한 것 (*하나의 뷰로만 아키텍처를 완벽히 표현할 수 없다*)
- compile 전후로 static/dynamic perspective 로 나눔
    - mixed perspective
    - 일단은 나눠서 구분하자


view-type
- 모듈(module) 뷰타입
    - concept
        - 모듈 열거
        - 모듈 간 relation (**중요!**)
    - modularization
    - element
        - module
    - relation
        - is-part-of
        - depends-on
        - is-a (Lion is an animal)
- 표기법
    - UML (권장)
    - Dependency Structure Matrix
    - ERD
- style
    - module decomposition style
    - uses style
    - generalization style

> [!notice]
> 숙제: system context (input/output) 를 작성해보고 model-view 를 작성한다.
> 
> 아직 못했다. 다음주(10/12)까지 한 번 해보자


**chapter 5** (5/10/2023)

---

컴포넌트와 커넥터(C&C) 뷰타입
- 컴포넌트: 연산 요소나 데이터 저장공간
- 커넥터: 두 컴포넌트 사이 상호작용이 일어나는 공간
- 관계와 속성
- 용도
- 대소문자 변환 시스템의 C&C 뷰와 모듈 뷰: 모듈뷰에서 보이는 것과 CNC 뷰에서 보이는 것은 다르기 때문에 모두 작성한다.
- UML, plantiUML 추천
    - 아래는 plantuml
    ```plantuml
    Bob -> Alice : 안녕?
    Alice -> Wonderland: hello
    Wonderland -> next: hello
    next -> Last: hello
    Last -> next: hello
    next -> Wonderland : hello
    Wonderland -> Alice : hello
    Alice -> Bob: hello
    ```

**chapter 6** (12/10/2023)

---

컴포넌트와 커넥터(C&C) 뷰타입
- 공유 데이터 스타일(shared data style)
- 발행 구독 스타일(publish-subscribe style)
- 클라이언트/서버 스타일(clinet-server style)
- service-oriented
- peer to peer
- broker
- model-view-controller
- context/problem/solution


**chapter 7** (19/10/2023)

---

컴포넌트와 커넥터(C&C) 뷰타입 - 요약 \
할당 뷰
- implementation style (e.g MVVM)
- work assignment style
- map-reduce style(pattern)
- multi-tier pattern 

요약
- 모듈 뷰 타입 스타일: 분할/사용/일반화/계층 스타일
- C&C 뷰 타입 스타일: 파이프/필터, 공유 데이터, 발행 구독, 클라이언트/서버

architecture design
- always design by considering its next larger context
- be hierarchical
- element 와 relation 을 설계하는 것이 architecture design
- tactic tree
- design process
    - system context
    - quality attributes
    - architectural pattern
    - physical perspective
- 참고자료
    - design tactics: https://www.se.rit.edu/~swen-440/slides/instructor-specific/Kuehl/Lecture%2019%20Design%20Tactics.pdf


**chapter 9 (9/11/2023)**

---

- understand, PMD
- Lattix (Impact 관련)
  - 12% → 100개 파일의 있는 경우 영향을 받는 파일이 12개 → 기능 추가하기 무섭다.
  - 낮을수록 기능 개발하기 쉽다.
- 상호참조비율
- SA? → shared understanding
  - element: null perpective
  - relationship: QA(Quality Attributes), perpective
- perspective type
  - static
  - dynamic
  - physical