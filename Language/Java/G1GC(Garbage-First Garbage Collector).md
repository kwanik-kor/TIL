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

## 2.1 기본적인 GC의 매커니즘

> 여러 GC 기법이 존재하지만, GC들의 기본 매커니즘은 동일하다.

- 식별(Mark)
	- Root Space로부터 그래프 순회를 통해 연결된 객체들을 찾아내어 어떤 객체를 참조하고 있는지 찾아서 마킹
- 제거(Sweep)
	- 참조하고 있지 않은 Unreachable 객체를 Heap에서 제거
- 압축(Compact)
	- Sweep 후에 분산된 객체들을 Heap의 시작 주소로 모아 메모리가 할당된 부분과 그렇지 않은 부분으로 압축

## 2.2 G1GC의 동작 단계

### 2.2.1 Region 관리
- 최대 2048개의 Region으로 나누어서 관리
- 기본 아이디어는어떤 Region에는 다른 Region 보다 쓰레기가 많으므로, STW가 수반되더라도 정확히 어떤 작업을 수행해야 할 지 알고 있으므로 효율적이다.
- 기존 GC들은 Old Generation을 정리할 때, Full GC가 발생했지만 G1GC에서는 Young과 Old를 함께 처리할 수 있음

### 2.2.2 Concurrent Marking Phase
1. Initial Mark
	1. 
2. ConcurrentMarking
3. Remark
4. Concurrent Cleanup
5. Evacuation

### 2.2.3 Collection Set 선택

### 2.2.4  Compaction





---
# 3. References


---
# 4. 생각들

1. 그럼 G1GC 이전에는 어떤 GC들이 있었고, 왜 G1GC가 STW를 최소화할 수 있게 된걸까?
2. 왜 G1GC가 기본 GC로 설정된걸까?
3. Region으로 나눈다는 것은 일종의 가상 메모리 관리 기법인 페이징 기법에서 출발한 것이라고 이해할 수 있을까?
	1. 페이징 기법에선 내부 단편화 문제가 발생할 수 있는데, Region으로 나눴을 때 내부 단편화 문제는 없는지
4. Full GC와 STW의 차이
5. 레퍼런스 타입의 정적 변수는 Old Generation으로 가는걸까?
6. ~~힙 메모리를 참조하는 녀석은 레퍼런스 타입의 정적 변수와 스택 영역인데 GC는 대체 이것이 참조되지 않음을 어떻게 알고 이동시키는 것일까?~~
	1. 도달능력(Reachability)을 기준으로 판단하는데, Reachable 상태임은 역으로 어떻게 아는 것일까?
7. 기존의 GC는 Young Generation을 훨씬 적게 잡는데, G1GC에서는 Young Generation인 Eden과 Survivor이 많아질 수 있다. 이슈가 없을까?