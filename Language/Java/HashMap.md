> HashMap에 대해 깊게 알아본다.

# 1. put
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

## 1.1 put의 기본적인 동작 순서

## 21
# 2. remove


# 3. get

