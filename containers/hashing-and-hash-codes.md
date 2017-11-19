# 散列和散列码

使用散列的集合（包括 `HashSet`、`HashMap`、`LinkedHashSet` 或 `LinkedHashMap`）采用了一种称为“hash 算法”的方式来决定元素在数组中的存储位置，所以集合中的元素或者键，必须同时重载 `equals()` 和 `hashCode()`。

*   `equals()` 是根类 `Object` 中的方法，它用于严格地判断两个对象是否相同。因此所有 Java 对象都能够进行比较。

*   `hashCode()` 也是根类 `Object` 中的方法，它返回一个 “相对唯一”的、用于代表对象的 `int` 值。因此所有 Java 对象都能产生散列码。使用散列的数据结构就是使用对象的 `hashCode()` 进行快速查询的，此方法能够显著提高性能。

## 为速度而散列

性能是 `Set` 和 `Map` 中的一个重要问题。性能瓶颈在于元素的查询速度：当使用线性搜索时，执行速度会相当地幔。解决方案之一就是保持元素的排序状态，然后使用 `Collections.binarySearch()` 进行查询。

散列使用数组（也称为*散列表*）来存储数据，以便能够快速找到。因为效率最高的存储和随机访问对象序列的数据结构就是是数组。但是数组结构存在两个问题：

*   数组不能调整容量，那么如何能够保存数量不确定的值呢？答案就是：数组通过元素生成一个数字，将其经过一定的处理后作为数组的下标。这个数字就是*散列码*，由类的 `hashCode()` 生成。这样任何元素总能在数组中找到自己的位置。

*   不同的元素可以产生相同的数组下标，可能会有冲突。通常，冲突由外部链接处理：数组并不保存值，而是保存值的序列（最简单的就是 `List`，`HashMap` 使用了一个单向链表），然后对序列中的值使用 `equals()` 进行线性的查询。这部分的查询自然比较慢。但是，如果散列函数好的话，数组的每个位置就只有较少的值。

这样查询一个值的过程就是计算散列码，然后使用散列码查询数组。如果能够保证没有冲突，那可就有了一个*完美的散列函数*。否则，在数组的当前位置上，只需对序列中很少的值进行比较即可。这便是散列如此快的原因。

设计 `hashCode()` 的首要因素是：对同一个对象调用 `hashCode()` 必须生成相同的值。为了创建一个好的散列函数，对不相等的对象调用 `hashCode()` 还应该尽量产生不同的散列码。换句话说，好的 `hashCode()` 应该产生均匀分布的散列码。如果散列码都集中在一块，那么散列表在某些区域的负载会很重。

关于如何覆盖 `hashCode()`，请参考[hashCode() 方法](../object/common-methods-to-all-objects/hash-code-method.md)。另外，Apache 的 Jacarta Commons 项目 lang 类库可以帮助你编写正确的 `equals()` 和 `hashCode()`。

## HashMap

对 `HashMap` 而言，它将键值对中的值当作键的附属，当决定了键的存储位置之后，随之将值也保存在那里即可。

具体地说，`HashMap` 使用一个 `Entry[]` 来存储键值对数据，而 `Entry` 是单向链表的结构；换句话说，数组中的某个位置保存的是一个 `Entry` 链。当添加一个键值对时，它会使用键的散列码来确定该 `Entry` 在数组中的存储位置。如果数组中的某个 `Entry` 的键也具有相同散列码，说明这两个 `Entry` 应该保存在数组中相同位置上，然后再使用 `equals()` 比较两个 `Entry` 的键：

*   如果 `equals()` 返回 `true`，则使用新 `Entry` 的值去覆盖原 `Entry` 的值；

*   如果 `equals()` 返回 `false`，新 `Entry` 将与原 `Entry` 形成 `Entry` 链，并且新 `Entry` 位于 `Entry` 链的头部。

## HashSet

`HashSet` 本质上就是一个 `HashMap`。`HashSet` 封装了一个 `HashMap` 对象来存储所有的元素。所有放入 `HashSet` 中的元素实际上会保存为底层的 `HashMap` 的键，而底层的 `HashMap` 的值都是同一个静态的 `Object` 对象。`HashSet` 中的绝大部分方法都是通过调用 `HashMap` 的方法来实现的。

显然易见，`HashSet` 和 `HashMap` 的 hash 算法是完全相同的。

## LinkedHashMap

源代码摘自 Java 1.8.0_131。

### Entry 元素

`Map.Entry` 接口表示一个键值对对象。`HashMap` 定义了一个 `Node` 类，它实现了 `Map.Entry` 接口，同时还保存了下一个 `Node` 对象的引用。这是一种单向链表的结构，这里也称之为“哈希表”。`HashMap` 使用一个 `Node` 数组来保存所有的键值对对象。

`LinkedHashMap` 重新定义了 一个 `Entry` 类，它继承自 `HashMap.Node`，同时还保存了上一个 `Entry` 对象和下一个 `Entry` 对象的引用。这样就从哈希表的基础上构成了双向链表。源码如下：

```java
/**
 * 双向链表的表头（最旧的）元素。
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * 双向链表的表尾（最新的）元素。
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 * HashMap.Node 的子类，用于普通的 LinkedHashMap 的 Entry。
 */
static class Entry<K,V> extends HashMap.Node<K,V> {
	Entry<K,V> before, after;
	Entry(int hash, K key, V value, Node<K,V> next) {
		super(hash, key, value, next);
	}
}
```

### 存储

`LinkedHashMap` 并未覆盖父类 `HashMap` 的 `put()` 方法，而是覆盖了父类 `HashMap` 的 `put()` 方法调用的子方法 `afterNodeAccess()` 和 `afterNodeInsertion()`，提供了自己特有的双向链表的实现。

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
	LinkedHashMap.Entry<K,V> last;
	if (accessOrder && (last = tail) != e) {
		LinkedHashMap.Entry<K,V> p =
			(LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
		p.after = null;
		if (b == null)
			head = a;
		else
			b.after = a;
		if (a != null)
			a.before = b;
		else
			last = b;
		if (last == null)
			head = p;
		else {
			p.before = last;
			last.after = p;
		}
		tail = p;
		++modCount;
	}
}
```

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
	LinkedHashMap.Entry<K,V> first;
	if (evict && (first = head) != null && removeEldestEntry(first)) {
		K key = first.key;
		removeNode(hash(key), key, null, false, true);
	}
}
```

### 读取

`LinkedHashMap` 覆盖了父类 `HashMap` 的 `get()` 方法，实际在调用父类 `HashMap` 的 `getNode()` 方法取得要查找的 `Node` 后，再判断当排序模式 `accessOrder` 为 `true` 时，记录访问顺序，将最新访问的元素添加到双向链表的表头，并从原来的位置删除。由于的链表的增加、删除操作是常量级的，故并不会带来性能的损失。

```java
public V get(Object key) {
	Node<K,V> e;
	if ((e = getNode(hash(key), key)) == null)
		return null;
	if (accessOrder)
		afterNodeAccess(e);
	return e.value;
}
```