---
{"dg-publish":true,"permalink":"/etc/__/refactoring/replace-conditional-with-polymorphism/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


![](https://i.imgur.com/Dm2sntV.png)

```java
class Bird {
  // ...
  double getSpeed() {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }
    throw new RuntimeException("Should be unreachable");
  }
}
```

- object 유형 또는 properties 에 따라 다양한 작업을 수행하는 조건이 있다.
- solution
	- 조건 분기에 해당하는 서브클래스를 생성
	- 서브 클래스 안에 메서드를 생성한다.
		- 클래스에 따라 다형성을 이용해 적절히 구현한다.

```java
abstract class Bird {
  // ...
  abstract double getSpeed();
}

class European extends Bird {
  double getSpeed() {
    return getBaseSpeed();
  }
}
class African extends Bird {
  double getSpeed() {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  double getSpeed() {
    return (isNailed) ? 0 : getBaseSpeed(voltage);
  }
}

// client code
speed = bird.getSpeed();
```

# reference
- [https://refactoring.com/catalog/replaceConditionalWithPolymorphism.html](https://refactoring.com/catalog/replaceConditionalWithPolymorphism.html)
- [https://refactoring.guru/ko/replace-conditional-with-polymorphism](https://refactoring.guru/ko/replace-conditional-with-polymorphism)