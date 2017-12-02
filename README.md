# Java

## 对象

*   [协变、逆变和不变](object/covariant-contravariant-invariant.md)

### 对于所有对象都通用的方法

`Object` 类所有的非 `final` 方法都有明确的通用约定（general contract），因为它们被设计成是要被覆盖（override）的。任何一个类，在覆盖这些方法的时候，都有责任遵守这些通用约定；否则，其他依赖于这些约定的类（例如 `HashMap` 和 `HashSet`）就无法结合该类一起正常运作。这些方法包括：

*   [equals() 和 hashCode() 方法](object/common-methods-to-all-objects/equals-and-hashcode-method.md) 覆盖 `equals()` 时请遵守通用约定；覆盖 `equals()` 时总是覆盖 `hashCode()`。

	如果在覆盖 `equals()` 后不覆盖 `hashCode()`，就会违反 hashCode 的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作。这样的集合包括 `HashMap`、`HashSet`、`LinkedHashMap` 和 `LinkedHashSet`。

*   [toString() 方法](object/common-methods-to-all-objects/tostring-method.md) 始终要覆盖 `toString()`。

*   [clone() 方法](object/common-methods-to-all-objects/clone-method.md) 谨慎地覆盖 `clone()`。

*   finalize() 方法

*   [Comparable#compareTo() 方法](object/common-methods-to-all-objects/compareto-method.md) 虽然不是 `Object` 方法，但是它具有类似的特征。

	如果违反了 compareTo 的通用约定，就会导致该类不能与依赖于比较关系的类一起正常运作。依赖于比较关系的类包括有序集合 `TreeSet` 和 `TreeMap`，以及工具类 `Collections` 和 `Arrays`，它们内部包含有搜索和排序算法。

## 语法基础

*   [访问权限控制](grammer/access-control.md)  

*   [final 关键字](grammer/final-keyword.md)

## 容器

*   [散列和散列码](containers/hashing-and-hash-codes.md)

## 附录-ASCII

*   [ASCII码对照表](http://tool.oschina.net/commons?type=4)

*   [ASCII码表——在线ASCII字符转换](ascii/ASCII码表——在线ASCII字符转换.mht)

*   [ASCII Table and Description](ascii/Ascii Table - ASCII character codes and html, octal, hex and decimal chart conversion.mht)

