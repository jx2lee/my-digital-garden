---
{"dg-publish":true,"permalink":"/data/rdb/__/rdb-mysql-privilege/","tags":["rdb","MySQL","privilege"],"noteIcon":"","created":"2024-06-30T00:39:32.606+09:00"}
---


### background


- datahub information 스키마를 이용한 메타데이터 추출 기능을 개발했다.
- `references` 권한을 가진 user([참고](https://tech.socarcorp.kr/data/2022/03/16/metdata-platform-02.html)) 로 메타데이터를 추출했는데, datahub 에서 메타데이터 추출 시 테이블에 대한 select 권한이 필요했다.
- 권한신청을 받고 개발 완료할 시점에 mysql privilege 중 하나인 `references` 권한과 `replicate` 권한을 간단히 정리한다.

### Meaning


**외래 키 제약 조건 생성을 위한 권한**
- 상위 테이블에 대한 REFERENCES 권한 필요

### replicate

replicate client `SHOW MASER STATUS`/`SHOW SLAVE STATUS` 문을 실행할 수 있는 권한이다. 
replicate slave user 는 master 와 연결하고 master bin log 에 대한 업데이트 내역을 확인할 수 있는 권한이다. 두 권한의 공통점은,
- 둘 다 MySQL 복제를 관리하는 데 필요한 권한이며, 주로 복제된 데이터를 다룰 때 사용한다.
- 이 두 권한은 복제된 데이터의 전송/관리가 가능하다.

replicate 뒷 단어가 다른것으로 보아 둘 권한의 차이는,
- **replicate client:**
    - 이 권한은 마스터 서버에 대한 연결을 나타낸다.
    - "replicate client" 권한을 가진 사용자는 마스터 서버에 연결하여 이벤트를 수신할 수 있습니다.
    - 주로 마스터에서 바이너리 로그의 이벤트를 읽어 슬레이브로 전송하는 작업과 관련이 있다.
- **replicate slave:**
    - 이 권한은 슬레이브 서버에서의 작업과 관련이 있다.
    - "replicate slave" 권한을 가진 사용자는 슬레이브 서버에서 마스터로부터 받은 이벤트를 처리하고 적용할 수 있다.
    - 주로 슬레이브 서버에서 복제된 데이터를 관리하는 작업과 관련이 있다.


### references


- [https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_references](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_references)
- [The Replication User Account](https://www.oreilly.com/library/view/mysql-in-a/9780596514334/ch08s03.html)