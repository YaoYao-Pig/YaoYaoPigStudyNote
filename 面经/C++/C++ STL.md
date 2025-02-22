# map的查找效率是多少

在 C++ 标准库中，`std::map` 通常使用**红黑树**（或其他平衡二叉搜索树）作为底层数据结构，因此它的常见操作（例如 `find`、`insert`、`erase`）的**时间复杂度**都为 **O(log n)**。下面列出几个与查找相关的重点：

1. **查找（find）**
   - 复杂度：**O(log n)**。
   - 由于底层是平衡二叉搜索树，寻找键值的过程就是在树中做一次平衡二叉查找，每一步都能排除一半左右的可能节点，因此是对数级别。
2. **插入（insert）**
   - 复杂度：**O(log n)**。
   - 插入键时，需要先在树中查找正确的插入位置（对数级别），然后可能触发局部的旋转/变色操作，以保持平衡。
3. **删除（erase）**
   - 复杂度：**O(log n)**。
   - 类似插入，需要先查找目标键，再做平衡树的删除与局部旋转操作。
4. **与其他关联容器的比较**
   - `std::unordered_map` 是基于哈希表的，平均复杂度 **O(1)** 查找，但最坏情况下可能退化到 **O(n)**；
   - `std::map` 则是稳定的 **O(log n)**，并且它是有序的，支持有序遍历及各种基于排序的操作（如 `lower_bound`、`upper_bound`、`equal_range`），这些操作也都维持在对数复杂度；
   - `std::multimap` 与 `std::map` 类似，也是平衡树结构，支持相同键的多重映射，查找复杂度依旧是 **O(log n)**。

------

### 小结

- **std::map**：平衡 BST 实现，**查找复杂度 = O(log n)**
- **std::unordered_map**：哈希表实现，**查找平均复杂度 = O(1)**（最坏 **O(n)**）

如果你需要“**有序**”并且“查找/插入/删除都在对数时间”，就使用 `std::map`；如果主要需求是**快速查找**且对键的有序性没有要求，使用 `std::unordered_map` 通常会更好。