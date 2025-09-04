> 

## 1. 먼저 Connection Pool은 무엇일까?



## 2. Connection Pool이 왜 필요할까?


## 3. HikariPool은 어떻게 생성되는거지?

### 3.1 생성자 톺아보기
### 3.1.1 SuspendResumeLock

> HikariCP 내부에서 쓰이는 락 구현체로, `ReentrantLock`이 아닌 `Semaphore`기반으로 풀 전체를 일시 정지 하거나 재개할 수 있게 설계된 락

```java
public Connection getConnection(final long hardTimeout) throws SQLException  
{  
   suspendResumeLock.acquire();
   try {
		// 풀에서 커넥션 얻기
   } finally {
	   suspendResumeLock.release();
   }
}
```

### 3.1.2 HouseKeepingExecutorService

> 커넥션 풀의 상태를 주기적으로 점검하기 위한 스케줄링 가능한 쓰레드 풀로, Idle Connection을 관리하고, Connection Validation을 처리하는 역할을 수행함.

- Idle connection 관리
	- 풀에 남아 있는 커넥션 중에서 너무 오래 idle 상태로 있는 것을 닫고 정리
	- 필요 시 새 커넥션을 미리 채워 넣어서 minimumIdle을 보장
- Connection validation
	- 풀에 있는 커넥션이 여전히 살아 있는지 주기적으로 점검
	- 죽은 커넥션이 있으면 폐기하고 새로운 커넥션으로 교체

### 3.1.3 checkFailFast()

> 초기화 시점에 풀을 미리 점검하는 작업으로, 애플리케이션 시작 시점에 DB 연결이 가능한지 미리 확인하는 역할 수행

- initializationTimeout에 따라 지정된 시간만큼 재시도 후 실패 처리
	- 음수인 경우 체크하지 않으며, 0인 경우 즉시 실패
- DB 커넥션을 하나 생성하여, 성공할 경우 minimumIdle에 따라 커넥션을 풀에 추가하거나, 닫음
	- 실패했을 때, 원인이 `ConnectionSetupException`인 경우 곧바로 예외 던짐
		- URL 오타, 인증 실패, 드라이버 문제 -> 아무리 재시도해도 의미가 없기 때문!
	- minimumIdle이 0보다 크다면 이 때 생성한 커넥션을 그대로 사용함

### 3.1.4 메트릭 수집 기능 초기화

> 커넥션 풀의 상태(active connections, idle connections, wait 시간 등)를 외부 모니터링 시스템에 노출할 수 있도록 설정

- MetircsTrackerFactory
	- 구현체를 사용한 모니터링 연동
- MetircRegistry
	- Dropwizard Metrics 기반으로 기본 메트릭 등록

### 3.1.5 Health Check Registry

> Dropwizard HealthCheck 라이브러리와 연동하기 위한 부분으로, DB 연결 상태를 HealthCheck로 등록할 수 있음

### 3.1.6 JMX MBeans 등록

> Hikari 풀 상태를 JMX MBean으로 노출할 수 있는데, 이를 통해 JConsole, VisualVM 같은 툴로 모니터링을 할 수 있음

- **JMX, MBeans가 무엇일까?**

### 3.1.7 Connection Add/Close Executors

> DB 커넥션을 등록하고 종료하는 작업을 담당하는 실행자들을 등록 및 초기화

- addConnectionExecutor
	- 새로운 DB 커넥션을 만드는 작업 담당
	- `LinkedBlockingQueue`를 사용해서 대기 작업 큐잉
	- `CustomDiscardPolicy`는 큐가 꽉 찼을 때 작업을 버리고 로그만 남기는 정책
- closeConnectionExecutor
	- 커텍션 종료 작업 담당
	- 풀에서 유휴 시간이 지난 커넥션을 닫는 등의 작업을 비동기로 수행

### 3.1.8 Leak Detection
> 밀리초 단위 이상으로 커넥션을 반납하지 않으면 로그에 경고를 남기는 역할

- 디버깅, 테스트 시에 유용하지만, 운영에서는 너무 짧게 잡을 경우 false-positive가 생길 수 있음

### 3.1.9 HouseKeeper 등록

> 풀을 정기적으로 점검하는 백그라운드 작업으로 default 30초

- idle timeout 지난 커넥션 닫기
- validation query로 연결 상태 점검
- keepalive 옵션이 켜져 있으면 주기적으로 ping
- **clock drift** 보정

### 3.1.10 blockUntilFilled 옵션

> `-Dcom.zaxxer.hikari.blockUntilFilled=true` JVM 옵션으로 활성화 가능한 옵션이며, 기본적으로 Hikari는 lazy initialization을 하지만 이 옵션 시에는 애플리케이션 시작 전에 minimumIdle만큼 풀이 채워질 때까지 block 처리를 하게 됨

- 일종의 웜업 기능이라고 볼 수 있을 것 같은데, 커넥션을 사용하는 API 콜을 이용한 웜업을 한다면 커넥션 풀이 채워져 있을테니 굳이 사용이 필요한 기능일까?
- 이걸 통해 웜업을 하면 첫 요청에 대한 latency는 얼마나 차이날까?

----
### References
