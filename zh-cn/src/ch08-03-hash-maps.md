## 使用哈希表存储键及其关联值

最后一种常见集合是哈希表。`HashMap<K, V>` 使用<em>哈希函数(hashing function)</em>
决定键值在内存中的位置，存储从类型 `K` 的键到类型 `V` 的值的映射。许多语言都
支持这种数据结构，名称可能是 hash、map、object、hash table、dictionary 或
associative array 等。

如果要用任意类型的键而非向量索引查找数据，哈希表很有用。例如游戏可以用队名
作为键、得分作为值，记录每支队伍的分数；给出队名即可取得分数。

本节介绍基本 API，标准库还为 `HashMap<K, V>` 定义了更多功能，请查阅其文档。

### 创建新哈希表

可以用 `new` 创建空表，再用 `insert` 添加元素。示例 8-20 记录 Blue 和 Yellow
两队的分数，初始分别为 10 和 50。

<Listing number="8-20" caption="创建新哈希表并插入一些键和值">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

首先需要从标准库 collections 部分 `use HashMap`。它是三种集合中使用最少的，
所以未由预导入模块自动引入；标准库对它的支持也较少，例如没有内置构造宏。

与向量一样，哈希表把数据存储在堆上。这里的键为 `String`，值为 `i32`。哈希表
也是同质的：所有键必须同类型，所有值也必须同类型。

<a id="accessing-values-in-a-hash-map"></a>

### 访问哈希表中的值

把键传给 `get` 即可取得值，如示例 8-21。

<Listing number="8-21" caption="访问哈希表中 Blue 队的分数">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

`score` 的结果为 `10`。`get` 返回 `Option<&V>`；不存在该键时返回 `None`。程序
调用 `copied` 把 `Option<&i32>` 变为 `Option<i32>`，再用 `unwrap_or` 在键不存在
时把 `score` 设为零。

可以像遍历向量一样，用 `for` 遍历每个键值对：

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

代码以任意顺序打印：

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### 管理哈希表中的所有权

对于 `i32` 这类实现 `Copy` 的类型，值会复制进哈希表；对于 `String` 这类拥有
所有权的值，值会移动，由哈希表成为所有者，如示例 8-22。

<Listing number="8-22" caption="展示插入后键和值归哈希表所有">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

调用 `insert` 把 `field_name` 和 `field_value` 移入哈希表后，不能再使用它们。

如果插入值的引用，值不会移动，但引用指向的值必须至少与哈希表一样长时间有效。
第 10 章的[“使用生命周期验证引用”][validating-references-with-lifetimes]
<!-- ignore -->一节会深入讨论。

### 更新哈希表

键值对数量可以增长，但每个唯一键同一时间只能关联一个值（反过来不成立，例如
Blue 和 Yellow 都可以得 10 分）。修改数据时，需要决定键已有值时如何处理：用
新值替换旧值；保留旧值，只有键不存在时才添加；或组合新旧值。

#### 覆盖值

插入某键值后再为同一键插入不同值，关联值会被替换。示例 8-23 虽调用两次
`insert`，最终只有一个键值对。

<Listing number="8-23" caption="替换与特定键关联的值">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

输出 `{"Blue": 25}`，原值 `10` 被覆盖。

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### 仅在键不存在时添加键值

经常需要检查键是否已有值：存在则保留，不存在才插入键和值。

哈希表为此提供 `entry` API，接收要检查的键，返回表示值可能存在或不存在的
`Entry` 枚举。示例 8-24 检查 Yellow 和 Blue 的键，不存在时分别插入值。

<Listing number="8-24" caption="使用 `entry`，只在键尚无关联值时插入">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

键存在时，`Entry::or_insert` 返回对应值的可变引用；否则把形参作为新值插入，
再返回新值的可变引用。这比自行编写逻辑简洁，也更符合借用检查器要求。

运行后输出 `{"Yellow": 50, "Blue": 10}`。第一次 `entry` 插入不存在的 Yellow；
第二次不改变已有值为 `10` 的 Blue。

#### 根据旧值更新值

另一个常见用法是查找键值，再根据旧值更新。示例 8-25 统计文本中每个单词出现
次数：单词作键，计数作值；首次见到时先插入 `0`。

<Listing number="8-25" caption="用保存单词和计数的哈希表统计单词出现次数">

```rust
{{#rustdoc_include ../../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

输出 `{"world": 2, "hello": 1, "wonderful": 1}`，顺序可能不同；回想[“访问哈希表中的值”][access]<!-- ignore -->一节，哈希表
遍历顺序任意。

`split_whitespace` 返回按空白分隔的 `text` 子切片迭代器。`or_insert` 返回指定
键对应值的可变引用 `&mut V`，存入 `count` 后，必须先用 `*` 解引用才能赋值。
可变引用在每次 `for` 迭代结束时离开作用域，因此修改符合借用规则。

### 哈希函数

`HashMap` 默认使用名为 <em>SipHash</em> 的哈希函数，可以抵御针对哈希表的拒绝服务
攻击[^siphash]<!-- ignore -->。它不是最快算法，但性能下降换来的安全性值得。如果
性能分析发现默认函数过慢，可以指定其他<em>哈希器(hasher)</em>。哈希器是实现
`BuildHasher` 特征的类型；[第 10 章][traits]<!-- ignore -->会介绍特征。无须从头
实现，[crates.io](https://crates.io/)<!-- ignore -->有许多提供常见算法哈希器的库。

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## 小结

向量、字符串和哈希表为程序存储、访问和修改数据提供了大量功能。现在可以尝试：

1. 给定整数列表，用向量返回中位数（排序后居中的值）和众数（出现最多的值；哈希
   表在这里很有帮助）。
2. 把字符串转换为 Pig Latin：把每个单词的第一个辅音移到词尾并添加 <em>ay</em>，
   如 <em>first</em> 变为 <em>irst-fay</em>；元音开头的词在末尾添加 <em>hay</em>，如
   <em>apple</em> 变为 <em>apple-hay</em>。注意 UTF-8 编码细节！
3. 使用哈希表和向量创建文本界面，让用户把员工姓名加入公司部门，例如“Add Sally
   to Engineering”或“Add Amir to Sales”；再让用户按字母顺序取得某部门全部人员，
   或按部门列出公司全部人员。

标准库 API 文档介绍了向量、字符串和哈希表中可帮助完成练习的方法！

程序正变得更复杂，操作也可能失败，现在正适合讨论错误处理。下一章继续！

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
