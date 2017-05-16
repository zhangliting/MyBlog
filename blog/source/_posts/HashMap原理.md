---
title: HashMap原理
tags: [HashMap]
categories: [HashMap]
photo:
  - /images/pose.jpg
date: 2017-04-18 00:58:37
---
# HashMap 工作原理
---
## 概念
* hash算法，将任何变量或对象，计算一个唯一的值。
* map, 将对象通过key,映射到值。
### example
`hashmap`使用内部类:Entry<K, V>存数据。
```java
static class Entry<K, V> implements Map.Entry<K,V>
{
final K key;
V value;
Entry<K, V> next;// liek linkedList
final int hash;// avoid computation everytime.
...//More code goes here
}
```
table默认大小16。为了优化，size=2^N.
举例：size=17, size-1=16。
16=0...010000, and h & (size-1) ，不管h是多少，结果只能是16 or 0.
```java
/**
 * The table, resized as necessary. Length MUST Always be a power of two.
 */
transient Entry[] table;
```
rehashes. avoid all data in the same index of array.
```java
// the "rehash" function in JAVA 7 that takes the hashcode of the key
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
// the "rehash" function in JAVA 8 that directly takes the key
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
// the function that returns the index from the rehashed hash
static int indexFor(int h, int length) {
    return h & (length-1);//取模运算的优化
}
```
<!--more-->

![hashmap](/images/internal_storage_java_hashmap.jpg)
## Put方法解析
```java
/**
* Associates the specified value with the specified key in this map. If the
* map previously contained a mapping for the key, the old value is
* replaced.
*
* @param key
*            key with which the specified value is to be associated
* @param value
*            value to be associated with the specified key
* @return the previous value associated with <tt>key</tt>, or <tt>null</tt>
*         if there was no mapping for <tt>key</tt>. (A <tt>null</tt> return
*         can also indicate that the map previously associated
*         <tt>null</tt> with <tt>key</tt>.)
*/
public V put(K key, V value) {
    if (key == null)
    return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K , V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
 
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
1. if (key == null) put table[0].// hash(null) == 0.
2. object.hashCode()计算对象的hash值，再使用HashMap的hash()避免所有数据都是一个的hash值。
3. indexFor(hash, table.length)计算具体的索引。
4. HashMap内部是一个数组，每个一个元素是一个Entry(LinkedList)。如果当前索引已经有值，遍历当前的Entry，判断 key.equals(k) ，如果是true,replace value. 否则就检查Entry的next, until Entry.next == null, store value

```flow
st=>start
key_is_null=>condition: key != null?
hashcode=>operation: hashCode(),hash();
indexFor=>operation: indexFor();
index=>operation: indexFor();
store=>operation: put talbe[0];
exist=>condition: iterator, key.equals(k)?
replace=>operation: replace value;
put=>operation: put value;
e=>end

st->key_is_null
key_is_null(no)->store(right)->e
key_is_null(yes)->hashcode->indexFor->exist
exist(yes)->replace->e
exist(no,left)->put(right)->e
```

## get()方法解析
```java
/**
* Returns the value to which the specified key is mapped, or {@code null}
* if this map contains no mapping for the key.
*
* <p>
* More formally, if this map contains a mapping from a key {@code k} to a
* value {@code v} such that {@code (key==null ? k==null :
* key.equals(k))}, then this method returns {@code v}; otherwise it returns
* {@code null}. (There can be at most one such mapping.)
*
* </p><p>
* A return value of {@code null} does not <i>necessarily</i> indicate that
* the map contains no mapping for the key; it's also possible that the map
* explicitly maps the key to {@code null}. The {@link #containsKey
* containsKey} operation may be used to distinguish these two cases.
*
* @see #put(Object, Object)
*/
public V get(Object key) {
    if (key == null)
    return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K , V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```
1. if key == null, 返回talbe[0]。
2. 根据hash(key.hashCode()),indexFor()找到entry。
3. 遍历entry，找到key.equals(k), or return null

## HashMap初始容量
HashMap初始容量是16，如果有16000条数据，平均每次get()，put()，remove()需要遍历1000次。
所以需要合理的初始化容量。
```java
public HashMap(int initialCapacity, float loadFactor)
```
`size`: the size of key-value store in hashmap.
`threshold`: threshold = capacity * loadFactor.

当put()时候，检查size> threshold，如果是true，自动扩大2倍容量。并且重新分配。
![auto_size](/images/resizing_of_java_hashmap.jpg)

## HashMap是线程不安全的
* 因为存在自动扩容，现存可能获得扩容前的索引，导致拿到的错误数据。
* 两个现存同时扩容，可能引起死循环。

**方案：**

- [ ] `HashTable`：CRUD使用synchronized，不是高效的。
- [X] `ConcurrentHashMap`：只synchronized buckets(Entry),如果hash不一样，就不会影响效率。

## Java 8 优化HashMap

当`TREEIFY_THRESHOLD > 8` 时，bucket就会从`likedList`转变成`balanced tree`（**红黑树**）。
当`UNTREEIFY_THRESHOLD 《 6`时，bucket就会从`balanced tree`转变成linkedList`。
性能：**O(n) -> O(logn)** (空间也会增加)
```java
static class Node<K,V> implements Map.Entry<K,V> {
     final int hash;
     final K key;
     V value;
     Node<K,V> next;//Node replace Entry.
     ...
```
java
```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    final int hash; // inherited from Node<K,V>
    final K key; // inherited from Node<K,V>
    V value; // inherited from Node<K,V>
    Node<K,V> next; // inherited from Node<K,V>
    Entry<K,V> before, after;// inherited from LinkedHashMap.Entry<K,V>
    TreeNode<K,V> parent;
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;
    boolean red;
    ...
```
因为TreeNode extends LinkedHashMap.Entry。所以可以使用原来的Entry.
![node](/images/internal_storage_java8_hashmap.jpg)


## 优化建议

- 使用合适的hashCode()减少冲突。
- 合理初始化HashMap容量，避免resize。默认16*0.75=12，当插入12th时，扩容。