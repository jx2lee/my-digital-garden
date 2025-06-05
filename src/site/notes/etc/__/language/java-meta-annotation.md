---
{"dg-publish":true,"permalink":"/etc/__/language/java-meta-annotation/","dgPassFrontmatter":true,"noteIcon":"","created":"2023-12-20T00:33:04.000+09:00"}
---


# Spring document: Using Meta-annotations and Composed Annotations

> meta-annotation 은 다른 annotation 에 적용할 수 있는 annotation 이다. 예를 들어, `@Service` 는 @Component 로 메타 어노테이션(is meta-annotationed)되어 있다.
> ```@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
> // ...
}```

번역하면 표현이 다소 모호하긴 한데 위 예시에서 `@Component` 가 `@Service` 의 meta-annotation 이다.

# reference
- [Spring: Using Meta-annotations and Composed Annotations](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html#beans-meta-annotations)