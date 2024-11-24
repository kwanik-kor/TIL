> Java의 쓰레드 객체와 운영체제 쓰레드는 어떤 관계가 있을까?

# 1. 운영체제의 스레드
> 프로세스가 실행중인 프로그램이라면, OS는 메모리에 적재될 수 있는 여러 프로세스에 필요한 자원을 할당하고, 스레드는 프로세스가 할당받은 자원을 이용해 프로세스의 작업을 수행하는 역할을 하며, 하나의 실행의 단위를 스레드라고 정의할 수 있음.

**스레드의 구성요소**
- 스레드 ID
- 프로그램 카운터(PC)
- 레지스터 값
- 스택

각각의 스레드는 프로그램 카운터 값과 스택을 가지고 있기 때문에 스레드마다 다음에 실행할 주소를 가질 수 있고, 연산 과정의 임시값을 저장할 수 있다. 멀티프로세서와의 차이점으로는 **자원의 공유 여부**가 있는데, 멀티스레드는 프로세스의 자원을 공유받으므로 동일한 주소 공간의 코드, 데이터, 힙 영역을 공유하고 있음

## 1.1 User-level thread, Kernel-level thread
| **특징**          | **User-Level Thread (ULT)**           | **Kernel-Level Thread (KLT)**         |
| --------------- | ------------------------------------- | ------------------------------------- |
| **관리 주체**       | 사용자 영역의 스레드 라이브러리에 의해 관리              | 운영 체제 커널에 의해 관리                       |
| **컨텍스트 전환**     | 빠름: 커널 모드 전환이 필요하지 않음                 | 느림: 커널 모드 전환이 필요                      |
| **운영 체제 인식 여부** | 운영 체제가 개별 스레드를 인식하지 않음                | 운영 체제가 개별 스레드를 직접 인식하고 관리             |
| **동시성**         | 하나의 프로세스가 실행 중일 때 그 프로세스 내 스레드만 실행 가능 | 다중 프로세서에서 스레드들이 병렬로 실행 가능             |
| **블로킹 I/O 처리**  | 한 스레드가 블로킹되면 전체 프로세스가 블로킹될 수 있음       | 한 스레드가 블로킹되어도 다른 스레드 실행 가능            |
| **포팅 가능성**      | 특정 운영 체제에 종속되지 않음                     | 운영 체제 의존적                             |
| **스케줄링**        | 사용자 라이브러리에서 스케줄링 수행                   | 운영 체제가 스케줄링 수행                        |
| **성능**          | 경량: 커널 호출이 없으므로 더 효율적일 수 있음           | 무겁다: 커널 호출로 인해 오버헤드가 있음               |
| **예제**          | Java Green Threads, Pthreads (사용자 모드) | Linux Kernel Threads, Windows Threads |

### 1.2 Hybrid Model
> ULT와 KLT의 장쩜을 결합한 하이브리드 모델

- Solaris M:N 스레딩 모델
	- 여러 사용자 스레드를 적은 수의 커널 스레드에 매핑하여 효율성을 높임

---
# 2. Java의 스레드

**쓰레드의 상속구조**
```Java
// thread.hpp:96
// Class hierarchy
// - Thread
//   - JavaThread
//     - various subclasses eg CompilerThread, ServiceThread
//   - NonJavaThread
//     - NamedThread
//       - VMThread
//       - ConcurrentGCThread
//       - WorkerThread
//     - WatcherThread
//     - JfrThreadSampler
//     - LogAsyncWriter
//
// All Thread subclasses must be either JavaThread or NonJavaThread.
// This means !t->is_Java_thread() iff t is a NonJavaThread, or t is
// a partially constructed/destroyed Thread.
```

**스레드 실행 흐름**
```Java
// thread.hpp:113
// Thread execution sequence and actions:
// All threads:
//  - thread_native_entry  // per-OS native entry point
//    - stack initialization
//    - other OS-level initialization (signal masks etc)
//    - handshake with creating thread (if not started suspended)
//    - this->call_run()  // common shared entry point
//      - shared common initialization
//      - this->pre_run()  // virtual per-thread-type initialization
//      - this->run()      // virtual per-thread-type "main" logic
//      - shared common tear-down
//      - this->post_run()  // virtual per-thread-type tear-down
//      - // 'this' no longer referenceable
//    - OS-level tear-down (minimal)
//    - final logging
//
// For JavaThread:
//   - this->run()  // virtual but not normally overridden
//     - this->thread_main_inner()  // extra call level to ensure correct stack calculations
//       - this->entry_point()  // set differently for each kind of JavaThread
```

**스레드 종류 및 우선 순위**
```Java
// os.hpp:70
enum ThreadPriority {        // JLS 20.20.1-3
  NoPriority       = -1,     // Initial non-priority value
  MinPriority      =  1,     // Minimum priority
  NormPriority     =  5,     // Normal (non-daemon) priority
  NearMaxPriority  =  9,     // High priority, used for VMThread
  MaxPriority      = 10,     // Highest priority, used for WatcherThread
                             // ensures that VMThread doesn't starve profiler
  CriticalPriority = 11      // Critical thread priority
};

// os.hpp:455
enum ThreadType {
    vm_thread,
    gc_thread,         // GC thread
    java_thread,       // Java, CodeCacheSweeper, JVMTIAgent and Service threads.
    compiler_thread,
    watcher_thread,
    asynclog_thread,   // dedicated to flushing logs
    os_thread
  };
```

---
### References
- [Java Thread에 대해 깊게 이해해보자](https://letsmakemyselfprogrammer.tistory.com/98)
- [Oracle MultiThreading reference](https://docs.oracle.com/cd/E19620-01/805-4031/6j3qv1oed/index.html)\
- [How Java thread maps to os thread](https://medium.com/@unmeshvjoshi/how-java-thread-maps-to-os-thread-e280a9fb2e06)