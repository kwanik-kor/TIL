> HashMap에 대해 깊게 알아본다.

# 1. Fields
> HashMap에 선언된 필드 목록

```Java
/**
 * The table, initialized on first use, and resized as necessary. When allocated, length is always a power of two. (We also tolerate length zero in some operations to allow bootstrapping mechanics that are currently not needed.)
 */
transient Node<K,V>[] table;

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used for keySet() and values().
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * The number of key-value mappings contained in this map.
 */
transient int size;

/**
 * The number of times this HashMap has been structurally modified Structural modifications are those that change the number of mappings in the HashMap or otherwise modify its internal structure (e.g., rehash). This field is used to make iterators on Collection-views of the HashMap fail-fast. (See ConcurrentModificationException).

 */
transient int modCount;

/**
 * The next size value at which to resize (capacity * load factor).
 */
int threshold;

/**
 * The load factor for the hash table.
 */
 
final float loadFactor;
```

**알아봐야 할 것들**
- 왜 내부 필드가 serialize 과정에서 제외되는 transient로 처리되어 있을까?
	- 직력화를 할 때, 전체 table 배열 자체를 직렬화하는 것 보다 키-값 쌍을 차례로 기록하는 것이 더 효율적이기 때문
- 

---
# 2. put
```Java
/**
 * Associates the specified value with the specified key in this map. If the map  * previously contained a mapping for the key, the old value is replaced.
 * 
 * @param key - key with which the specified value is to be associated
 * @param value - value to be associated with the specified key
 * @return the previous value associated with key, or null if there was no   
 * mapping for key. (A null return can also indidcate that the map previously 
 * associated null with key.)
 */
public V put(K key, V value) {  
    return putVal(hash(key), key, value, false, true);  
}

/**
 * Implements Map.put and related methods
 * 
 * @param hash - hash for key
 * @param key - the key
 * @param value - the value to put
 * @param onlyIfAbsent - if true, don't change existing value
 * @param evict - if false, the table is in creation mode.
 * 
 * @return previous value, of null if none
 */
public V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {

}
```

---
## 2.1 put의 기본적인 동작 순서

### 2.1.1 초기화
> 내부 필드로 선언된 table이 Null이거나 tab의 길이가 0인 경우 리사이징 후 초기화 처리

```Java
Node<K,V>[] tab; 
Node<K,V> p; 
int n, i;

if ((tab = table) == null || (n = tab.length) == 0) {
	n = (tab = resize()).length;
}
```

<details>
	<summary>table이 null이거나 tab의 길이가 0인 경우는 언제일까?</summary>
	<p>
		1. 초기화 상태
		2. 
	</p>
</details>

#### 2.1.1.1 Node
> `Map.Entry<K, V>` 의 구현체
- Java 7의 Entry 클래스와 내용이 같지만 Linked List 대신 Tree를 사용할 수 있도록 하위 클래스인 TreeNode가 있다는 것이 Java 7 Hash Map과의 차이점

#### 2.1.1.2 resize
> HashMap이 확장될 때 실행되는 핵심 함수로, HashMap의 크기 변경은 저장된 데이터의 충돌 최소화와 성능 유지를 위해 설계된 알고리즘으로 이루어짐

```Java
// 기존 해시 테이블의 크기 및 정보를 가져옴
Node<K,V>[] oldTab = table;  
int oldCap = (oldTab == null) ? 0 : oldTab.length;  
int oldThr = threshold;  
int newCap, newThr = 0;

// 기존 테이블 크기가 0보다 큰 경우
if (oldCap > 0) {  
	// 최대 용량 이상이라면 확장하지 않고 반환
	// 최대 용량이 2^30 인 이유는 hash가 int로 구현되기 때문
    if (oldCap >= MAXIMUM_CAPACITY) {  
        threshold = Integer.MAX_VALUE;  
        return oldTab;  
    }  
    // 기존 크기가 작다면 임계값을 두 배로 조정
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&  
             oldCap >= DEFAULT_INITIAL_CAPACITY)  
        newThr = oldThr << 1; // double threshold  
}  
// 테이블은 0이지만 임계치가 지정된 경우 새 임계치로 매핑
else if (oldThr > 0) // initial capacity was placed in threshold  
    newCap = oldThr;  
// 기본값으로 설정(16, 75%)
else {               // zero initial threshold signifies using defaults  
    newCap = DEFAULT_INITIAL_CAPACITY;  
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  
}

// 새 임계값 계산
if (newThr == 0) {  
    float ft = (float)newCap * loadFactor;  
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?  
              (int)ft : Integer.MAX_VALUE);  
}

// 새 해시 테이블 생성
threshold = newThr;  
@SuppressWarnings({"rawtypes","unchecked"})  
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];  
table = newTab;

// 기존(Old Table)이 있는 경우
if (oldTab != null) {  
    for (int j = 0; j < oldCap; ++j) {  
        Node<K,V> e;  
        if ((e = oldTab[j]) != null) {  
            oldTab[j] = null;  
            if (e.next == null)  
                newTab[e.hash & (newCap - 1)] = e;  
            else if (e instanceof TreeNode)  
                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
            else { // preserve order  
                Node<K,V> loHead = null, loTail = null;  
                Node<K,V> hiHead = null, hiTail = null;  
                Node<K,V> next;  
                do {  
                    next = e.next;  
                    if ((e.hash & oldCap) == 0) {  
                        if (loTail == null)  
                            loHead = e;  
                        else  
                            loTail.next = e;  
                        loTail = e;  
                    }  
                    else {  
                        if (hiTail == null)  
                            hiHead = e;  
                        else  
                            hiTail.next = e;  
                        hiTail = e;  
                    }  
                } while ((e = next) != null);  
                if (loTail != null) {  
                    loTail.next = null;  
                    newTab[j] = loHead;  
                }  
                if (hiTail != null) {  
                    hiTail.next = null;  
                    newTab[j + oldCap] = hiHead;  
                }  
            }  
        }  
    }  
}  
return newTab;
// 
```





### 2.1.2  해시 값 기반의 인덱스 계산 및 첫 번째 노드 확인
> 

```Java
if ((p = tab[i = (n - 1) & hash]) == null)  
    tab[i] = newNode(hash, key, value, null);
```

### 2.1.3 기존 노드가 있는 경우에 대한 처리


### 2.1.4 기존 노드 처리 및 값 업데이트


### 2.1.5 해시맵 상태 업데이트


---
## 2.2 언제 호출되는가


---
## 2.3 put은 O(1) 정말?


---
# 3. remove


# 4. get



---
### References
1. https://d2.naver.com/helloworld/831311