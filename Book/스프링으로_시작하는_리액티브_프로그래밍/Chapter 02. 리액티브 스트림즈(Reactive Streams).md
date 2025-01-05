# Chapter 02. 리액티브 스트림즈(Reactive Streams)
> 리액티브 스트림즈가 무엇이고, 핵심 컴포넌트인 Publisher와 Subscriber에 대해 이해할 수 있다.

## 2.1 리액티브 스트림즈란(Reactive Streams)?

**정의**
> 데이터 스트림을 Non-Blocking이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양

- 구현체
	- RxJava, Reactor, Akka Streams, Java 9 Flow API 등
---
## 2.2 리액티브 스트림즈 구성요소

| 컴포넌트         | 설명                                                                                                               |
| ------------ | ---------------------------------------------------------------------------------------------------------------- |
| Publisher    | 데이터를 생성하고 통지(발행, 게시, 방출)하는 역할                                                                                    |
| Subscriber   | 구독한 Publisher로부터 통지(발행, 게시, 방출)된 데이터를 전달받아서 처리하는 역할                                                              |
| Subscription | Publisher에 요청할 데이터의 개수를 저장하고, 데이터의 구독을 취소하는 역할                                                                   |
| Processor    | Publisher와 Subscriber의 기능을 모두 가지고 있음<br>Subscriber로서 다른 Publisher를 구독할 수 있고, Publisher로서 다른 Subscriber가 구독할 수 있음 |

- 데이터 전달 과정
	- **subscribe**: Subscriber는 전달받을 데이터를 구독
	- **onSubscribe**: Publisher는 데이터를 통지할 준비가 되었음을 Subscriber에게 알림
	- **Subscription.request**: Subscriber는 전달받기 원하는 데이터 개수를 Publisher에게 요청
		- 각기 다른 스레드에서 비동기적으로 상호작용하는 경우가 많으므로 Subscriber가 처리할 수 있는 개수 만큼을 요청
	- **onNext**: Publisher는 Subscriber로부터 요청받은 만큼의 데이터 통지
	- **onComplete**: 완료된 경우(Publisher -> Subscriber)
	- **onError**: 에러가 발생한 경우(Publisher -> Subscriber)

---
## 2.3 코드로 보는 리액티브 스트림즈 컴포넌트

### 2.3.1 Publisher

```java
// Java 9 Flow API

@FunctionalInterface  
public static interface Publisher<T> {  
    /**  
     * Adds the given Subscriber if possible.  If already     
     * subscribed, or the attempt to subscribe fails due to policy     
     * violations or errors, the Subscriber's {@code onError}  
     * method is invoked with an {@link IllegalStateException}.  
     * Otherwise, the Subscriber's {@code onSubscribe} method is  
     * invoked with a new {@link Subscription}.  Subscribers may  
     * enable receiving items by invoking the {@code request}  
     * method of this Subscription, and may unsubscribe by     
     * invoking its {@code cancel} method.  
     *     * @param subscriber the subscriber  
     * @throws NullPointerException if subscriber is null  
     */    
     public void subscribe(Subscriber<? super T> subscriber);  
}
```
- 개념상으로는 Subscriber가 Publisher를 구독하는 것이 맞으나, Java에서는 Publisher가 subscribe 메서드의 매개변수인 Subscriber를 등록하는 형태로 구독이 이루어짐

### 2.3.2 Subscriber
```java
// Java 9 Flow API

/**  
 * A receiver of messages.  The methods in this interface are 
 * invoked in strict sequential order for each {@link  
 * Subscription}.  
 * 
 * @param <T> the subscribed item type  
 */
 public static interface Subscriber<T> {  
    /**  
     * Method invoked prior to invoking any other Subscriber     
     * methods for the given Subscription. If this method throws     
     * an exception, resulting behavior is not guaranteed, but may     
     * cause the Subscription not to be established or to be cancelled.     
     *     
     * <p>Typically, implementations of this method invoke {@code  
     * subscription.request} to enable receiving items.  
     *     
     * @param subscription a new subscription  
     */    
     public void onSubscribe(Subscription subscription);  
  
    /**  
     * Method invoked with a Subscription's next item.  If this     
     * method throws an exception, resulting behavior is not     
     * guaranteed, but may cause the Subscription to be cancelled.     
     *     
     * @param item the item  
     */    
     public void onNext(T item);  
  
    /**  
     * Method invoked upon an unrecoverable error encountered by a     
     * Publisher or Subscription, after which no other Subscriber     
     * methods are invoked by the Subscription.  If this method     
     * itself throws an exception, resulting behavior is     
     * undefined.     
     *     
	 @param throwable the exception  
     */    
     public void onError(Throwable throwable);  
  
    /**  
     * Method invoked when it is known that no additional     
     * Subscriber method invocations will occur for a Subscription     
     * that is not already terminated by error, after which no     
     * other Subscriber methods are invoked by the Subscription.     
     * If this method throws an exception, resulting behavior is     
     * undefined.     
     * */    
     public void onComplete();  
}
```
- onSubscribe
	- 구독 시작 시점에 어떤 처리를 하는 역할
	- Publisher에게 요청할 데이터의 개수를 지정하거나 구독 해지 
	- Subscription을 통해서 처리함
- onNext
	- Publisher가 통지한 데이터를 처리하는 역할
- onError
	- Publisher가 통지 처리 과정에서 에러가 발생했을 때 처리하는 역할
- onComplete
	- Publisher가 데이터 통지 완료를 알릴 때 호출

### 2.3.3 Subscription
```java
/**  
 * Message control linking a {@link Publisher} and {@link  
 * Subscriber}.  Subscribers receive items only when requested,  
 * and may cancel at any time. The methods in this interface are 
 * intended to be invoked only by their Subscribers; usages in * other contexts have undefined effects. 
 */
public static interface Subscription {  
    /**  
     * Adds the given number {@code n} of items to the current  
     * unfulfilled demand for this subscription.  If {@code n} is  
     * less than or equal to zero, the Subscriber will receive an     
     * {@code onError} signal with an {@link  
     * IllegalArgumentException} argument.  Otherwise, the  
     * Subscriber will receive up to {@code n} additional {@code  
     * onNext} invocations (or fewer if terminated).  
     *     
     * @param n the increment of demand; a value of {@code  
     * Long.MAX_VALUE} may be considered as effectively unbounded  
     */    
     public void request(long n);  
  
    /**  
     * Causes the Subscriber to (eventually) stop receiving     
     * messages.  Implementation is best-effort -- additional     
     * messages may be received after invoking this method.     
     * A cancelled subscription need not ever receive an     
     * {@code onComplete} or {@code onError} signal.  
     */    
     public void cancel();  
}
```

- request
	- Publisher에게 데이터 개수를 요청
- cancel
	- Publisher에게 데이터 요청 취소

---
## 2.4 리액티브 스트림즈 관련 용어 정의

- Signal
	- Publisher와 Subscriber 간에 주고받는 상호작용
	- `onSubscribe` `onNext`, `onComplete`, `onError`, `request` , `cancel`
- Demand
	- Subscriber가 Publisher에게 요청하는 뎅ㅣ터
	- Publisher가 아직 Subscriber에게 전달하지 않은 Subscriber가 요청한 데이터
- Emit
	- Publisher가 Subscriber에게 데이터를 통지하는 것(내보내는 것)
- Upstream/Downstream
- Sequence
	- Publisher가 emit하는 데이터의 연속적인 흐름을 정의해 놓은 것
	- Operator 체인 형태로 정의
- Operator
	- 연산자
- Source
	- '최초의'의미로 사용됨

### 2.5 리액티브 스트림즈의 구현 규칙
1. Publisher가 Subscriber에게 보내는 onNext signal의 총 개수는 항상 해당 Subscriber의 구독을 통해 요청된 데이터의 총 개수보다 더 작거나 같아야 한다.
2. Publisher는 요청된 것보다 적은 수의 onNext signal을 보내고 onComplete 또는 onError를 호출하여 구독을 종료할 수 있다.
3. Publisher의 데이터 처리가 실패하면 onError signal을 보내야 한다.
4. Publisher의 데이터 처리가 성공적으로 종료되면 onComplete signal을 보내야 한다.
5. Publisher가 Subscriber에게 onError 또는 onComplete signal을 보내는 경우 해당 Subscriber의 구독은 취소된 것으로 간주되어야 한다.
6. 일단 종료 상태 signal을 받으면 (onError, onComplete) 더 이상 signal이 발생되지 않아야 한다.
7. 구독이 취소되면 Subscriber는 결국 signal을 받는 것을 중지해야 한다.