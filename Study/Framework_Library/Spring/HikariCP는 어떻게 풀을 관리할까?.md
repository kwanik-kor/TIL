## 1. Connection Pool은 무엇일까?



## 2. Connection Pool이 왜 필요할까?


## 3. HikariPool은 어떻게 생성되는거지?

### 3.1 생성자를 통해 살펴보는 구성항목
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

> 커넥션 풀의 상태를 주기적으로 점검하기 위한 스케줄링 가능한 쓰레드 풀로, Idle Connection을 관리하고, 

----
### References
