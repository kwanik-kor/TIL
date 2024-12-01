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

#### 2.1.1.2 resize
> HashMap이 확장될 때 실행되는 핵심 함수로, HashMap의 크기 변경은 저장된 데이터의 충돌 최소화와 성능 유지를 위해 설계된 알고리즘으로 이루어짐



### 2.1.2  해시 값 기반의 인덱스 계산 및 첫 번째 노드 확인
> 

```Java
if ((p = tab[i = (n - 1) & hash]) == null)  
    tab[i] = newNode(hash, key, value, null);
```

### 2.1.3 기존 노드가 있는 경우에 대한 처리


### 2.1.4 기존 노드 처리 및 값 업데이트


### 2.1.5 해시맵 상태 업데이트


## 2.2 언제 호출되는가

## 2.3 


---
# 3. remove


# 4. get

