---
{"dg-publish":true,"permalink":"/data/rdb/__/rdb-mysql-numeric-type-attributes/","tags":["rdb","numeric type","unsigned"],"noteIcon":"","created":"2024-06-30T00:39:32.606+09:00"}
---


> [!info] MySQL 5.7 문서를 기반으로 작성했다.
> - tldr; 음수값이 없고 더 큰 범위의 정수 타입의 데이터를 사용할 경우 UNSIGNED 를 이용하자

### meaning


> All integer types can have an optional (nonstandard) `UNSIGNED` attribute. An unsigned type can be used to permit only nonnegative numbers in a column or when you need a larger upper numeric range for the column. For example, if an [`INT`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.html "11.1.2 Integer Types (Exact Value) - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT") column is `UNSIGNED`, the size of the column's range is the same but its endpoints shift up, from `-2147483648` and `2147483647` to `0` and `4294967295`.

- Mysql 모든 정수 유형은 `UNSIGNED` 속성을 갖는다.
- `UNSIGNED` 타입은 열에 음수가 아닌 숫자만 허용하거나, 숫자 범위가 커야 할 때 사용한다.
    - 예를 들어, INT 열이 UNSIGNED 인 경우 범위 크기는 동일하다.
    - 단, 시작점과 끝점이 `-2147483648 & 2147483647` 에서 `0 & 4294967295` 로 변경된다.

> Floating-point and fixed-point types also can be `UNSIGNED`. As with integer types, this attribute prevents negative values from being stored in the column. Unlike the integer types, the upper range of column values remains the same.

- 부동 소수점 및 고정 소수점 타입도 `UNSIGNED` 속성을 가질 수 있다.
- 정수형과 마찬가지로 음수 값이 열에 저장되는 것을 방지한다.
    - 정수 유형과 달리 열 값의 상위 범위는 동일하게 유지된다.
- 단, UNSIGNED 는 FLOAT/DOUBLE 및 DECIMAL 타입의 열에서는 deprecated 될 예정이다. ([8.3 문서](https://dev.mysql.com/doc/refman/8.3/en/numeric-type-attributes.html))

### reference
- [Numeric Type Attributes](https://dev.mysql.com/doc/refman/5.7/en/numeric-type-attributes.html)