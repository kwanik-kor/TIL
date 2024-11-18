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
- 기본 아이디어는 어떤 Region에는 다른 Region 보다 쓰레기가 많으므로, STW가 수반되더라도 정확히 어떤 작업을 수행해야 할 지 알고 있으므로 효율적이다.
- 기존 GC들은 Old Generation을 정리할 때, Full GC가 발생했지만 G1GC에서는 Young과 Old를 함께 처리할 수 있음

### 2.2.2 Concurrent Marking Phase
힙 영역의 Old Region에서 살아 있는 객체를 식별하여 Garbage 비율을 계산하고, 수집 대상 Region을 선택하기 위해 수행된다. 대부분 병렬로 실행되나, 특정 단계에서는 STW가 발생함
1. Initial Mark
	1. GC 루트에서 참조되는 객체 그래프의 시작점, 스택, 정적 변수 등에서 참조되는 객체를 마킹
		1. GC 루트는 여러가지일 수 있음
			1. JVM 스레스 스택
			2. 전역 변수
			3. JVM 내부 참조(각 클래스의 메타 데이터, JNI)
			4. 활성 모니터
	2. STW
		1. 현재 상태를 고정하기 위함
		2. 애플리케이션 스레드가 동작 중이라면 객체 그래프가 변경될 수 있으므로 정확한 마킹이 불가능할 수 있으므로
		3. Eden Region과 Root 객체만 마킹하므로 빠르게 완료됨
		4. Young GC와 병행되어 추가적인 성능저하 최소화
	3. Young GC가 트리거 될 때 수행
		1. Eden Region이 가득 찬 경우
2. Root Region Scan
	1. Initial Mark에서 탐색된 Survivor Region의 객체들이 참조하는 Old Region 객체를 추적, 마킹
		1. Old Region 중에서 살아있는 객체 식별
		2. Survivor 객체가 offset 역할을 해줌
	2. 병렬로 실행됨에 따라 STW 발생하지 않음
3. ConcurrentMarking
	1. 힙의 모든 Old Region을 병렬로 스캔하여 살아 있는 객체를 마킹
	2. 애플리케이션과 동시에 수행되어 살아 있는 객체의 정보 수집
	3. 객체 참조 그래프를 분석하여 객체 연결성 파악
	4. GC 대상 객체가 발견되지 않은 Region은 이후 단계 처리에서 제외
	5. Old Region의 사용률이 일정 임계치 이상에 도달했을 때 실행
		1. 일반적으로 힙 메모리의 45% 이상 사용된 경우
4. Remark
	1. 애플리케이션 동작 중 객체 참조 관계가 변경된 부분을 확인하고, 추가적으로 살아 있는 객체를 마킹
		1. Initial Mark 시점에 떠 있지 않던 스레드
	2. STW
		1. Concurrent Marking 단계에서 객체 참조 그래프가 변경되었을 가능성이 있으므로
		2. DB 마이그레이션과 비슷한 단계?
	3. Marking 완료
5. Concurrent Cleanup
	1. Garbage가 많은 Region을 수집 대상으로 선택
	2. Garbage 비율이 높은 Region을 GC 대상으로 표시하고, Humongous Region에 있는 객체도 분석
	3. 마킹 작업의 메타데이터 정리
	4. STW
		1. 새로운 GC 대상 Region을 설정하는 작업을 수행하기 위해
6. Evacuation
	1. GC 대상 Region이었으나 Cleanup 단계에서 완전히 비워지지 않은 Region의 살아남은 객체들을 새로운(Available/Unused) Region에 복사, Compaction 수행
	2. STW 발생


> SATB(Snapshot-At-The-Beginning)

+ TAMS
	+ LinkedList
+ Floating... 
	+ 

### 2.2.3 Collection Set 선택

### 2.2.4  Compaction

---
# 3. References


---
# 4. 생각들

1. 그럼 G1GC 이전에는 어떤 GC들이 있었고, 왜 G1GC가 STW를 최소화할 수 있게 된걸까?
2. 왜 G1GC가 기본 GC로 설정된걸까?

1. Region으로 나눈다는 것은 일종의 가상 메모리 관리 기법인 페이징 기법에서 출발한 것이라고 이해할 수 있을까?
	1. 페이징 기법에선 내부 단편화 문제가 발생할 수 있는데, Region으로 나눴을 때 내부 단편화 문제는 없는지

1. Full GC와 STW의 차이
2. 레퍼런스 타입의 정적 변수는 Old Generation으로 가는걸까?
3. ~~힙 메모리를 참조하는 녀석은 레퍼런스 타입의 정적 변수와 스택 영역인데 GC는 대체 이것이 참조되지 않음을 어떻게 알고 이동시키는 것일까?~~
	1. 도달능력(Reachability)을 기준으로 판단하는데, Reachable 상태임은 역으로 어떻게 아는 것일까?

1. 기존의 GC는 Young Generation을 훨씬 적게 잡는데, G1GC에서는 Young Generation인 Eden과 Survivor이 많아질 수 있다. 이슈가 없을까?
	1. Old Generation이 더 컸던 적이 없음
2. 