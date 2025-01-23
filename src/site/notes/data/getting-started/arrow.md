---
{"dg-publish":true,"permalink":"/data/getting-started/arrow/","tags":["arrow"],"dgHomeLink":true,"dgShowBacklinks":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---



### definition
> Apache Arrow is a development platform for in-memory analytics. It contains a set of technologies that enable big data systems to process and move data fast. It specifies a standardized language-independent columnar memory format for flat and hierarchical data, organized for efficient analytic operations on modern hardware.
> 
> The project is developing a multi-language collection of libraries for solving systems problems related to in-memory analytical data processing. This includes such topics as:
> 
> - Zero-copy shared memory and RPC-based data movement
> - Reading and writing file formats (like CSV, Apache ORC, and Apache Parquet)
> - In-memory analytics and query processing

- 인메모리 분석을 위한 개발 플랫폼
- 데이터를 빠르게 처리하고 이동할 수 있도록 하는 일련의 기술을 포함합니다.
- 최신 하드웨어에서 효율적인 분석 작업을 수행할 수 있도록 구성된 flat 및 계층형 데이터를 위해 표준화된 language-independent 컬럼형 메모리 형식을 지원합니다.
- purpose: 인메모리(in-memory) 분석 데이터 처리와 관련된 시스템 문제를 해결한다고 합니다.
	- multi-language 라이브러리
- features
	- 제로 카피 공유 메모리(Zero-copy shared memory) 및 RPC 기반 데이터 처리
	- 파일 포맷 읽기 및 쓰기(예: CSV, Apache ORC, Apache Parquet)
	- 인메모리 분석 및 쿼리 처리(query processing)


### cookbooks
- 자바를 이용한 quickstart
- [installation](https://arrow.apache.org/docs/java/install.html)

> intellij 사용 시 주의사항
> - 아래와 같이 실행옵션에서 environment 를 설정해야 오류가 발생하지 않아요.
> - `_JAVA_OPTIONS="--add-opens=java.base/java.nio=ALL-UNNAMED"`
> 	![|500](https://i.imgur.com/UuSH1hp.png)


#### Create a ValueVector
ValueVector 는 동일한 유형값들의 시퀀스를 나타내요. 열 형식(columnar format)에서는 `배열, arrays` 이라고도 해요.

### References
Arrow Flight SQL
- https://arrow.apache.org/docs/format/FlightSql.html#
- https://www.slideshare.net/slideshow/apache-arrow-flight-overview/100340338
- https://www.dremio.com/blog/an-introduction-to-apache-arrow-flight-sql
- https://www.dremio.com/blog/arrow-flight-sql-a-universal-jdbc-driver