##Mysql索引为什么会失效？

首先mysql 索引的存在是为了提高查询的速度。当 Mysql 查询优化器判断，某些情况下不使用索引，反而可能更快拿到结果时，从而不使用索引。也就是产生了索引失效的情况。

##Mysql 索引在什么情况下会失效？

在以下几种情况下，Mysql的 select 语句中，哪怕使用了索引，依然会失效：

1. 对查询的字段（**select后的字段或 where 条件中的字段**）进行运算（**函数或隐式类型转换**）时，索引失效。
2. 对字符串类型的字段，不使用`"”`包起来时，会导致系统的隐式转换，从而索引失效。
3. `or运算`时。左右两边出现不是索引字段时，会导致失效。
4. `复合索引`，`左前缀规则`被打断时，会失效。左前缀规则。不是指 where 后使用字段的顺序，而是有没有用到复合索引中的字段，用到时，有没有按最左边的字段开始。中间有没有跳过。假如有三个字段，使用了第一个和第三方，这时，索引是生效的，只是第三方字段没有使用索引。因为用了第一个字段的索引。
5. `like` 查询时，左模糊 即`%abc`时，会失效。
6. `is null 和 is not null `时，有时会失效。失效的情况要根据字段的值中 null 所占比来看。如果大部分都是 null 时，is null 会失效。而 is not null 不会失效。反之亦然。原因是，大部分都是 null时，那不用索引反而更快。所以这种情况下，再以 is null 来查询。自然索引是帮不上什么忙的。所以不走索引。
7. ` not in `和`!=`查询时。即排除小部分的查询。索引帮不上忙。会失效。

## Mysql 索引出现失效时如何避免？

有些情况下的查询，哪怕触发了索引的失效。可以通过覆盖索引的使用来避免失效。

覆盖索引的意思时，在 select 字段时，字段只选择使用 索引字段。从而强制使用索引。

为什么 select 索引字段时，会生效呢？原因是 select 的执行顺序要优于 where。所以 select 的字段判断是索引字段时，会在索引的基础上去进行查询。

也就是我们常说的，优化方案中提到的尽量不要 `seelect *`。



