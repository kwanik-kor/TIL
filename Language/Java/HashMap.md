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
	- 뭔솔?

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
            // Separate Chaining
            // 다음 노드가 설정되어 있지 않다면 바로 새 위치를 계산하여 새 테이블에 복사
            if (e.next == null)  
	            // 용량이 2의 제곱수 이므로 1을 빼서 1의 자리 수로 만든 다음 해시 비트와 연산
	            // 즉, 해시 비트 위치에 값을 넣음
                newTab[e.hash & (newCap - 1)] = e;  
			// 트리 노드를 분리하여 새 테이블에 적절히 재배치
            else if (e instanceof TreeNode)  
                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);  
            else { // preserve order  
            // 연결 리스트로 구성된 버킷의 데이터를 새 테이블로 옮김
                Node<K,V> loHead = null, loTail = null;  
                Node<K,V> hiHead = null, hiTail = null;  
                Node<K,V> next;  
                do {  
                    next = e.next;  
                    // 기존 임계치의 최상위 비트를 기준으로 그룹을 나눔
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
                
	            // Low Group은 기존 위치에 두고
                if (loTail != null) {  
                    loTail.next = null;  
                    newTab[j] = loHead;  
                }  
                // High Group은 기존 해시값에 oldCap을 더한 위치로 이동
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

- 싱글 노드에 대해서는 해당 노드의 해시값으로 바로 위치 시켰는데, 그룹으로 분할할 경우, High Group은 Old cap이 더해진 값으로 이동시킴? 그럼 어떻게 찾죠? 해시값으로?

### 2.1.2  해시 값 기반의 인덱스 계산 및 첫 번째 노드 확인

```Java
// 노드의 해시값에 데이터가 없다면, 테이블에 새로운 노드를 생성하여 배치함
// 나머지 연산 테이블 버킷의 길이로 나눠서 균등 배치를 하고자 함
if ((p = tab[i = (n - 1) & hash]) == null)  
    tab[i] = newNode(hash, key, value, null);

// 해시위치에 기존 노드가 있는 경우
else {  
    Node<K,V> e; 
    K k;  
    
    // 해시값이 동일하고, 키가 동일하다면
    if (p.hash == hash &&  
        ((k = p.key) == key || (key != null && key.equals(k))))  
        // 기존 노드를 e로 설정
        e = p;  
	// 트리구조라면 트리 노드에 삽입을 위임
    else if (p instanceof TreeNode)  
        e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);  
	// 해시값이 다르거나, 키가 같지 않은 경우
    else {  
        for (int binCount = 0; ; ++binCount) {  
			// 루트 노드의 다음 노드가 없을 경우 해당 노드에 새로운 노드로 생성/배치
            if ((e = p.next) == null) {  
                p.next = newNode(hash, key, value, null);  
                // 트리 구성 임계치(8)를 넘어설 경우 트리 재구성
                if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
                    treeifyBin(tab, hash);  
                break;  
            }  
            // p를 다음 노드 값으로 설정
            if (e.hash == hash &&  
                ((k = e.key) == key || (key != null && key.equals(k))))  
                break;  
            p = e;  
        }  
    }  
    // 기존 노드가 발견된 경우에는 값을 대체
    // 추가 등록된 노드에 따른 사후처리 진행
    if (e != null) { // existing mapping for key  
        V oldValue = e.value;  
        if (!onlyIfAbsent || oldValue == null)  
            e.value = value;  
		// LinkedHashMap에서만 처리
        afterNodeAccess(e);  
        return oldValue;  
    }  
}
```


### 2.1.3 해시맵 상태 업데이트
```Java
++modCount;  
if (++size > threshold)  
    resize();  
// LinkedHashMap 에서만 처리
afterNodeInsertion(evict);  
return null;
```


---
## 2.3 put은 O(1) 정말?
- 트리의 다음 노드를 순회하는 과정이 포함되는데 이는 상수 시간이라 할 수 있는가?
	- 트리 노드의 임계치가 8로 설정되어 있어 최대 연산 횟수가 고정값이므로 상수 시간이라고 볼 수 있음
	- 즉, 해시값 기반으로 배열에 접근하는 시간 상수시간만 발생함
	- 하지만 해시 테이블의 확장이 일어날수록 접근해야 하는 건수가 늘어나는 것은 사실임

---
# 3. remove
## 3.1 remove의 기본 동작 순서
```Java
/**
 * @param hash - hash for key
 * @param key - the key
 * @param value - the value to match if matchValue, else ignored
 * @param matchValue - if true only remove if value is equal
 * @param movable - if false do not move other nodes while removing
 */
final Node<K,V> removeNode(int hash, Object key, Object value,  
                           boolean matchValue, boolean movable) {  
    Node<K,V>[] tab; 
    Node<K,V> p; 
    int n, index;  
    
    // table이 존재하고, 테이블의 데이터가 존재하고, 해시값에 해당하는 노드가 있을 경우만 진행
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (p = tab[index = (n - 1) & hash]) != null) {  
        
        Node<K,V> node = null, e; 
        K k; 
        V v;  
        
        // 해시값과 키가 동일하다면 node는 해시로 탐색한 노드
        if (p.hash == hash &&  
            ((k = p.key) == key || (key != null && key.equals(k))))  
            node = p;  
		// 노드의 다음 값이 존재한다면
        else if ((e = p.next) != null) {  
	        // 트리인 경우 트리 로직
            if (p instanceof TreeNode)  
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);  
			// 노드의 다음 노드가 존재하는 동안
            else {  
                do {
	                // 해시값과 키가 동일한 노드를 찾는 다면 탐색 종료
                    if (e.hash == hash &&  
                        ((k = e.key) == key ||  
                         (key != null && key.equals(k)))) {  
                        node = e;  
                        break;  
                    }  
                    p = e;  
                } while ((e = e.next) != null);  
            }  
        }

		// 1. 노드를 찾았고 2. 값 일치 여부를 따지지 않거나 3. 따진다면 값이 같을 경우
        if (node != null && (!matchValue || (v = node.value) == value ||  
                             (value != null && value.equals(v)))) {  
			// 트리면 트리 로직
            if (node instanceof TreeNode)  
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);  
			// 탐색한 노드가 해시값으로 탐색한 헤드 노드였다면, 한 칸 씩 당김
            else if (node == p)  
                tab[index] = node.next;  
			// 그렇지 않다면 탐색된 해시 노드에 이어붙임
            else  
                p.next = node.next;  
            ++modCount;  
            --size;  
            // LinkedHashMap에서만 처리
            afterNodeRemoval(node);  
            return node;  
        }  
    }  
    return null;  
}
```


# 4. get
## 4.1 get의 기본 동작 순서
```Java
/**
 * @param hash - hash for key
 * @param key - the key
 */
final Node<K,V> getNode(int hash, Object key) {  
    Node<K,V>[] tab; 
    Node<K,V> first, e; 
    int n; 
    K k;  

	// 테이블과 테이블의 데이터가 존재하고, 해시 기반의 해드 노드가 존재할 경우 탐색
    if ((tab = table) != null && (n = tab.length) > 0 &&  
        (first = tab[(n - 1) & hash]) != null) {  

		// 해시값과 키가 동일하다면 반환함
        if (first.hash == hash && // always check first node  
            ((k = first.key) == key || (key != null && key.equals(k))))  
            return first;  

		// 다음 노드가 존재한다면
        if ((e = first.next) != null) {  
            if (first instanceof TreeNode)  
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);  
            do {  
		        // 찾을 때까지 수행
                if (e.hash == hash &&  
                    ((k = e.key) == key || (key != null && key.equals(k))))  
                    return e;  
            } while ((e = e.next) != null);  
        }  
    }  
    return null;  
}
```

# 5. hash
> 아니 그래서 LinkedList로 관리되는 이 hash의 첫 번째 노드는 무엇을 기준으로 세팅하는거지?

## 5.1 그 전에 잠시 해싱이 근본적으로 뭔지 정의할 수 있나요?
> 임의의 길이의 데이터를 고정된 길이의 데이터로 매핑하는 함수. 빠른 데이터 검색을 위해 사용. 해시 함수의 질은 입력 영역에서의 해시 충돌 확률로 결정

## 5.2 hash(Object key)
```Java
static final int hash(Object key) {  
    int h;  
    // 상위 16비트 값을 XOR 처리
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  
}
```

**이 보조 해시 함수가 필요한 이유**
- 해시테이블에 데이터를 넣을 때 버킷의 길이(2^a)로 나머지 연산을 함에 따라 해시코드의 하위 a개의 비트만 사용하게 됨. 

---
### References
1. https://d2.naver.com/helloworld/831311