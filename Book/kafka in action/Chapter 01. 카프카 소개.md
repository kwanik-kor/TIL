# 1.1 카프카의 특징
[분산 스트리밍 플랫폼](https://kafka.apache.org/intro)
- 메시지 큐처럼 레코드를 읽고 쓴다.
- 내결함성(fault tolerance)으로 레코드를 저장
- 스트림이 발생할 때 처리

Delivery
- at-least-once-semantics(최소 한 번 시맨틱)
	- 메시지는 수신확인이 될 때까지 재발송
- at-most-once-semantics(최대 한 번 시맨틱)
	- 메시지는 단 한 번 보내며 실패하더라도 재발송하지 않음
- exactly-once-semantics(정확히 한 번 시맨틱)
	- 메시지는 이 메시지의 컨슈머에게 단 한 번만 보임
