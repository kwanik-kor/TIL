# 1. 개요
> Java 9 이후의 기본 GC로 설정된 G1GC는 JVM의 힙 메모리를 효율적으로 관리하기 위해 설계된 최신 가비지 컬렉터로 STW 시간을 최소화하기 위해 설계되었음

- **분리된 힙 메모리 관리**
	- 힙 메모리를 고정 크기의 Region(영역)으로 나누어 관리
	- 각각의 Region은 역할에 따라 Eden, Survivor, Old로 동작하거나 Humongous 객체를 저장
- 예측 가능한 GC 시간
	- STW 시간을 최소화하기 위해 설계
	- **Pause Time Goal**을 설정할 수 있어, 애플리케이션의 응답성을 유지함녀서 GC 수행
- 동시처리
	- 대부분의 GC 작업이 애플리케이션 스레드와 병렬로 수행
- Humongous 객체 지원
	- 크기가 Region 크기의 50% 이상인 객체는 특별히 Humongous Region에 저장
- Copying GC
	- 객체를 복사하며 압축을 수행하여 메모리 단편화를 줄임


---
# 2. 동작방식



---
# 3. References


---
# 4. 생각들

1. 그럼 G1GC 이전에는 어떤 GC들이 있었고, 왜 G1GC가 STW를 최소화할 수 있게 된걸까?
2. 왜 G1GC가 기본 GC로 설정된걸까?
3. Region으로 나눈다는 것은 일종의 가상 메모리 관리 기법인 페이징 기법에서 출발한 것이라고 이해할 수 있을까?
4. Full GC와 STW의 차이