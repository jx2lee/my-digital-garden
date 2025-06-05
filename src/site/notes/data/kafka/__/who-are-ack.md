---
{"dg-publish":true,"dg-show-toc":true,"permalink":"/data/kafka/__/who-are-ack/","dgShowToc":true,"dgPassFrontmatter":true,"noteIcon":"","created":"2024-06-30T00:39:32.000+09:00"}
---



> [!info] gcp pubsub 을 이용중인데 ack(acknowledge) 용어를 사용하고 있다. kafka 에서도 사용하는데, 서로 다르게 해석하는지 문서를 확인하며 비교해본다.

### ack in GCP pubsub


GCP 공식 문서에 다음과 같이 정의한다.


### ack in kafka official document


acknowledge 에 대한 개념 정의를 따로 정리해두지 않았다. producer configuration 파트에서 `acks` 옵션에서 간단히 설명되어 있다.

> (원문) The number of acknowledgments the producer requires the leader to have received before considering a request complete.


- acks 은 **프로듀서에서 요청을 완료한 것으로 간주하기 위한 필요한 리더 승인(acknowledgement)** 수
- 사전적 의미는 승인이라는 의미를 갖고 있지만, [승범님의 의견](https://www.popit.kr/kafka-운영자가-말하는-producer-acks/)처럼 ***확인** 이라는 단어로 이해하면 좋을 것 같다.*
- 브로커 입장에서 프로듀서가 보낸 메세지를 “정상적으로 수신했다(메세지를 저장했다)“ 라고 판단가능한 기준이 필요하다.
	- 아무도 확인을 안할것인가(`0`), 파티션 leader 만 확인할 것이냐(`1`), leader & follower 모두가 확인할 것이냐(`all or -1`) 라는 기준을 설정하기 위한 옵션이 producer 의 **acks** 옵션임

### conclusion

TBD

### reference


- [Kafka 운영자가 말하는 Producer ACKS](https://www.popit.kr/kafka-운영자가-말하는-producer-acks/)