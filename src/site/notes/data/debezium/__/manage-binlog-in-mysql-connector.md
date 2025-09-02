---
{"author":"jx2lee","aliases":null,"created":"2024-10-04T20:59:40.000+09:00","last-updated":"2024-10-04 20:59","tags":["debezium","binlog"],"dg-publish":true,"dg-home-link":true,"dg-show-local-graph":true,"dg-show-backlinks":false,"dg-show-toc":false,"dg-show-inline-title":false,"dg-show-file-tree":false,"dg-enable-search":true,"dg-link-preview":true,"dg-show-tags":false,"dg-pass-frontmatter":false,"permalink":"/data/debezium/__/manage-binlog-in-mysql-connector/","dgHomeLink":true,"dgShowLocalGraph":true,"dgEnableSearch":true,"dgLinkPreview":true,"dgPassFrontmatter":true,"noteIcon":""}
---


### 배경
SQL Server 커넥터를 이용한 파이프라인 구축이 필요했습니다. 이러한 상황에서 작업을 함께 진행할 DBA 분과 이야기하던 도중 질문하신 내용에 대한 (내 기준)답변을 제대로 하지 못했다. 기술 원리를 알지 못하고 사용만 하는 모습을 경멸(?)하는 내가, 그러한 행위를 했다는 것에 실망했다. 머릿속에다 집어넣고, 무엇을 찾아보고 정리했는지 공유하고자 노트를 작성한다.

목표는 도큐먼트를 바탕으로 실제 커넥터의 코드레벨에서 확인한다.

### 