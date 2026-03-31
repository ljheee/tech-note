

#### 自定义EqualsAndHashCode
HashSet类型，可以直接作为字段进行哈希计算；
HashSet#hashCode 会遍历集合每个元素；集合元素完全相同，但遍历顺序不同，h += obj.hashCode()计算出的哈希值相同；
HashMap同理。

List<Long>类型计算哈希值：两个List元素完全相同，但元素顺序不同，hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());计算出的哈希值不相同。
hashCode方法中对Ids进行了排序，这会改变原始对象的内部状态，违反了hashCode方法不应修改对象的原则；需要深拷贝后排序。

