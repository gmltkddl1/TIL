# SQS란
- AWS에서 제공하는 완전관리형 메세지 큐 서비스로 분산 서비스간의 비동기 통신과 서비스 디커플링을 위해 사용된다
## 장점
- 서비스간 직접 호출을 없애 결합도를 낮춤 (느슨한 결합)
- 트래픽의 급증시 버퍼 역할
- producer나 consumer가 죽어도 다른 쪽은 정상 동작 (내결함성)
## 표준 큐 vs FIFO 큐
표준 큐
- 순서 보장 X
- 최소 1회 전송
- 처리량 무제한
- 가격 상대적으로 저렴
- 사용처: 로그나 순서가 중요하지 않을 떄
FIFO
- 순서 보장
- 1회 전송 보장
- 처리량 제한 (300TPS)
- 상대적으로 비쌈
- 사용처: 결제 주문 등 정합성이 중요할때
## visiblity timeout
- consumer가 메세지를 소비하면 일정 시간 동안 다른 소비자에게 보이지 않음
- 성공시 제거
- 실패시 timeout이후에 다른 consumer에게 보임
- 실제 처리 시간이 visiblity timeout보다 길면 중복 처리 발생 가능
- ChangeMessageVisibility API로 동적 연장 가능
## DLQ
- 처리에 반복적으로 실패한 메세지를 보관하는 큐
- maxReceiveCount로 최대 재시도 횟수 설정
- DLQ는 원복 큐와 같은 타입 (표준 - 표준, FIFO - FIFO)
## Long polling
- 아무 설정 없을 경우 short polling
- WaitTimeSeconds=20으로 long polling 설정
- 트래픽이 적을 때 비용 절약 가능
## SNS + SQS fan out
- SNS가 하나의 이벤트를 여러 큐에서 넣어준다
- 하나의 이벤트에 대해 독립적으로 처리 가능
- SNS 필터 정책으로 원하는 메세지만 수신 가능
- Producer 코드 수정없이 consumer 추가 가능

FAQ
Q. SQS와 Kafka의 차이는?
SQS는 완전 관리형으로 운영 부담이 없고, 메시지 소비 후 삭제됩니다. Kafka는 메시지를 보존하며 여러 컨슈머 그룹이 독립적으로 재소비 가능하고, 대용량 스트리밍에 적합합니다. 단순한 작업 큐는 SQS, 이벤트 스트리밍/재처리가 필요하면 Kafka를 선택합니다.
Q. 중복 처리를 어떻게 방지하나요?
FIFO 큐의 MessageDeduplicationId를 활용하거나, 소비자 측에서 멱등성(Idempotency) 을 보장하는 로직을 구현합니다. (예: DB에 처리 완료 여부 기록 후 체크)
Q. 메시지 순서가 중요한데 표준 큐를 써야 한다면?
메시지에 시퀀스 번호를 붙여 소비자가 순서를 직접 관리하거나, FIFO 큐로 전환을 검토합니다.
Q. SQS 사용 시 비용을 줄이려면?
Long Polling 사용, 배치 전송(send_message_batch, 최대 10개), 불필요한 빈 폴링 제거
