> Java의 쓰레드 객체와 운영체제 쓰레드는 어떤 관계가 있을까?

# 1. 운영체제의 스레드
> 프로세스가 실행중인 프로그램이라면, OS는 메모리에 적재될 수 있는 여러 프로세스에 필요한 자원을 할당하고, 스레드는 프로세스가 할당받은 자원을 이용해 프로세스의 작업을 수행하는 역할을 하며, 하나의 실행의 단위를 스레드라고 정의할 수 있음.

**스레드의 구성요소**
- 스레드 ID
- 프로그램 카운터(PC)
- 레지스터 값
- 스택

 각각의 스레드는 프로그램 카운터 값과 스택을 가지고 있기 때문에 스레드 마다 다음에 실행할 주소를 가질 수 있고, 연산 과정의 임시값을 저장할 수 있다. 멀티 프로세서와의 차이점으로는 **자원의 공유 여부**가 있는데, 멀티스레드는 프로세스의 자원을 공유받으므로 동일한 주소 공간의 코드, 데이터, 힙 영역을 공유하고 있음.

컨텍스트 스위칭은 안하는 것에 비해 상대적으로 비싸다.

- PCB, TCB
- Process 컨텍스트 스위칭 -> MMU

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
> ULT와 KLT의 장점을 결합한 하이브리드 모델

- Solaris M:N 스레딩 모델
	- 여러 사용자 스레드를 적은 수의 커널 스레드에 매핑하여 효율성을 높임
- 요즘의 하이브리드란?
	- Virtual Thread

---
# 2. Java의 스레드
> Java의 스레드는 Hybrid Model로 Java Application 레벨의 스레드는 ULT지만 JVM은 KLT를 사용한다. 초기에는 Green Thread의 Many-To-One 모델이었으나 현재는 OS 정책에 따라 처리되고 있음. 

![[Pasted image 20241125202833.png]]

## 2.1 Java의 스레드가 네이티브 스레드에 매핑되는 과정
```Java
// 1) Thread 생성, start 호출
Thread thread = new Thread().start()

// 2) 내부적으로 네이티브 메서드인 start0() 호출
// 해당 메서드를 통해 운영체제에 스레드를 생성하라고 요청
// 운영체제는 해당 요청을 처리하고 Java 스레드를 커널 스레드와 연결
// JVM은 스레드의 실행 가능한 코드를 네이티브 호출로 전달하여 실행을 준비
	// POSIX 스레드와 같은 운영체제 API 호출
	// 생성한 객체는 JavaThreadWrapper로 불러옴
	// JavaThreadWrapper를 통해 JVM에 대한 접근 권한과, 자바 스레드에 대한 참조가 가능해짐
// Runnable 상태로 변경
```

### 2.1.1 JavaThread vs NonJavaThread
1. JavaThread
   Java 애플리케이션 코드에서 생성된 스레드, Java Thread 클래스나 Runnable 인터페이스를 사용하여 생성된 스레드는 JVM 내부적으로 JavaThread로 관리됨
   - Java 애플리케이션 코드와 직접 연관된 스레드
   - ~~JVM이 JavaThread를 관리~~하면서, Java 프로그램의 라이프사이클과 연동
	   - 생성, 삭제 정도
   - Java 런타임 환경과 밀접하게 연결
	   - Java의 스레드 상태(NEW, RUNNABLE, BLOCKED, WAITING)와 밀접하게 연결
	   - JVM 스케줄러와 협력하여 작동
	   - 스택 영역과 네이티브 메서드 스택을 함께 사용
	   - JVM 인터프리터, JIT 컴파일러 혹은 네이티브 코드를 거쳐 호출
	   - Java 객체와 연관된 모든 참조는 JavaThread에 의해 관리되므로, GC 동작 시, JavaThread의 로컬 변수 및 스택 참조가 반드시 스캔되어야 함

2. NonJavaThread
   JVM 내부에서만 사용되며, Java 애플리케이션 코드와 직접적으로 연관되지 않은 스레드. JVM 내부 작업을 처리하기 위해 생성된 스레드로 간주할 수 있음
    - Java 애플리케이션에 의해 생성되지 않고, JVM 자체에서 관리
    - JVM이 내부적으로 사용하는 작업(GC, JIT 컴파일러, 코드 인터프리터)에 사용
    - 제어할 수 없음
	    - 네이티브 메서드 스택만 사용
	    - JVM 내부 데이터를 처리하므로, Java 객체 참조를 가질 필요가 없어 가비지 컬렉션 대상이 아님

JVM
- process
- JVM이 스레드의 상태 변경을 할 수 없음
- JVM은 스레드에 대해 아무것도 할 수 없음


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
### 질문

1. 어떤 형태로 학습하는가?
	1. 키워드들을 뽑아낸 다음, 키워드 기반의 검색을 함
	2. 최신 기술인 경우 만든 사람이 쓴 글이나 발표를 찾아봄

---
### 핵심
- 스레드의 행동을 예측하지 마라
- 차주
	- 하고 싶은 프로젝트에 대한 것들
	- 프로젝트 주제 준비, 유스케이스 설계
	- HashMap -> put, remove, get, resize
		- HashMap 코드 구현 자체를 이해해보자
		- HashMap은 실제 면접에서 많이 물어봄

---
### References
- [Java Thread에 대해 깊게 이해해보자](https://letsmakemyselfprogrammer.tistory.com/98)
- [Oracle MultiThreading reference](https://docs.oracle.com/cd/E19620-01/805-4031/6j3qv1oed/index.html)\
- [How Java thread maps to os thread](https://medium.com/@unmeshvjoshi/how-java-thread-maps-to-os-thread-e280a9fb2e06)
- https://medium.com/@may1998/java-thread%EC%99%80-os-thread-1ee766ff3393