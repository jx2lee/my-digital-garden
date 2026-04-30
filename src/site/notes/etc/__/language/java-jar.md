---
{"author":"jx2lee","aliases":"Java Jar(Java ARchive)","created":"2026-04-30T21:16:43.758+09:00","last-updated":"2026-04-30 21:16","tags":["java","jar"],"dg-publish":true,"dg-home-link":false,"dg-show-local-graph":false,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":false,"dg-link-preview":false,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/etc/__/language/java-jar/","dgPassFrontmatter":true,"noteIcon":""}
---


> JAR (Java Archive) is a platform-independent file format that aggregates many files into one. 
> *by columbia*

여러 파일을 하나로 묵는 플랫폼 독립적인 파일 형식이다.


> JAR stands for Java ARchive. It's a file format based on the popular ZIP file format and is used for aggregating many files into one. Although JAR can be used as a general archiving tool, the primary motivation for its development was so that Java applets and their requisite components (.class files, images and sounds) can be downloaded to a browser in a single HTTP transaction, rather than opening a new connection for each piece. This greatly improves the speed with which an applet can be loaded onto a web page and begin functioning. The JAR format also supports compression, which reduces the size of the file and improves download time still further. Additionally, individual entries in a JAR file may be digitally signed by the applet author to authenticate their origin.
> 
> JAR is:
> 
> - the only archive format that is cross-platform
> - the only format that handles audio and image files as well as class files
> - backward-compatible with existing applet code
> - an open standard, fully extendable, and written in java
> - the preferred way to bundle the pieces of a java applet
> 
> *by oracle*

- 이 파일 형식은 널리 사용되는 ZIP 파일 형식을 기반이며 여러 파일을 하나로 묶는다.
- 일반적인 압축 도구로 사용할 수 있지만, 주된 목적은 자바 애플릿(applet)과 이에 필요한 구성 요소(.class 파일, 이미지, 사운드)를 각각에 대해 새로운 연결을 열지 않고 단일 HTTP 트랜잭션으로 브라우저에 다운로드할 수 있도록 하기 위해 만들어졌다.
- JAR 형식은 압축을 지원하여 파일 크기를 줄이고 다운로드 시간을 더욱 단축시킨다. 더불어, JAR 파일 내 개별 항목은 애플릿 작성자가 디지털 서명을 하여 출처를 인증할 수 있습니다.

```
> jar tf bundle-2.42.41.jar
...
...

> # check main class
> unzip -p bundle-2.42.41.jar META-INF/MANIFEST.MF
```

### references
- https://docs.oracle.com/javase/6/docs/technotes/guides/jar/jarGuide.html
- https://docs.oracle.com/javase/6/docs/technotes/guides/jar/jar.html
- http://www.cse.yorku.ca/tech/other/jdk1.2.1/docs/guide/jar/manifest.html
- https://ko.wikipedia.org/wiki/JAR_(파일_포맷)