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

## 2.1 put의 기본적인 동작 순서

### 2.1.1 초기화
```Java
Node<K,V>[] tab; 
Node<K,V> p; 
int n, i;

if ((tab = table) == null || (n = tab.length) == 0) {
	n = (tab = resize()).length;
}
```


## 2.2 언제 호출되는가


---
# 3. remove


# 4. get

