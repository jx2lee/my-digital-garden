---
{"author":null,"aliases":null,"created":"2025-06-15T20:25:29.193+09:00","last-updated":"2025-06-15 20:25","tags":null,"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":"ture","dg-show-tags":false,"dg-pass-frontmatter":false,"priority":null,"permalink":"/etc/__/language/rust/","dgHomeLink":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":"ture","dgPassFrontmatter":true,"noteIcon":""}
---


- [[#job knowledge|job knowledge]]


### job knowledge
- [[etc/__/language/rust-mod\|rust-mod]]
- `.rs`: 모듈, 소스 파일, .rs 묶음은 crate(크레이트)
- [unwrap](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap): Returns the contained Some value, consuming the self value: `Option<T>` 타입에 쓰는 메서드로, **값이 반드시 있다고 확신할 때 사용하는 위험한 함수**
- [literals](https://doc.rust-lang.org/reference/tokens.html#literals): single quote 는 char, double quote 는 &'static str 타입 (문자열 슬라이스, &str)
- 역참조해서 값 꺼내기(수동 dereference) 나 복사해서 값 꺼내나(Copy 호출) 성능차이 없음! `*matches.get_one::<T>("name").unwrap()` vs `matches.get_one::<T>("name").copied().unwrap()`
