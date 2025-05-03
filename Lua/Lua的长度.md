由于Lua的表并不是传统意义上的数组或者哈希表，因此Lua长度的意义其实并不明确。也是因此我们使用#符号的时候要很慎重

Lua的#符号的具体行为要看版本：

实际上Lua#返回的是：边界

什么是边界，Lua的定义是：对于一个表t，若t[i]~=nil and t[i+1] == nil 那么，这个i就是一个边界

任何一个版本的#，对于数组或者哈希表，找的都是：边界

1. 5.2

这是我们从Github上拉取的v5.2版本的代码：

```lua
int luaH_getn (Table *t) {
  unsigned int j = t->sizearray;
    -- 二分查找数组部分的nil
  if (j > 0 && ttisnil(&t->array[j - 1])) {
    /* there is a boundary in the array part: (binary) search for it */
    unsigned int i = 0;
    while (j - i > 1) {
      unsigned int m = (i+j)/2;
      if (ttisnil(&t->array[m - 1])) j = m;
      else i = m;
    }
    return i;
  }
  /* else must find a boundary in hash part */
  else if (isdummy(t->node))  /* hash part is empty? */
    return j;  /* that is easy... */
  else return unbound_search(t, j);
}
-- 这个函数的作用是计算 Lua 表 t 的长度（对应 #t 操作符的结果）。它的实现分为以下几个步骤：

-- 检查数组部分：
-- j = t->sizearray：获取表的数组部分长度。
-- 如果 j > 0 且最后一个元素 t->array[j - 1] 为 nil，说明数组部分内部有空洞（即存在 nil）。
-- 在这种情况下，使用二分查找在数组部分 [0, j) 中找到第一个 nil 出现的位置：
-- 初始化 i = 0，j = t->sizearray。
-- 每次取中间位置 m = (i + j) / 2。
-- 如果 t->array[m - 1] 为 nil，说明边界在左侧，更新 j = m。
-- 如果 t->array[m - 1] 非 nil，说明边界在右侧，更新 i = m。
-- 最终返回 i，即最后一个非 nil 元素的索引加 1（也就是连续非 nil 元素的个数）。
-- 处理哈希部分：
-- 如果数组部分的最后一个元素非 nil（!ttisnil(&t->array[j - 1])），或者数组部分为空（j == 0），则需要检查哈希部分。
-- 如果哈希部分为空（isdummy(t->node)），直接返回数组部分的长度 j。
-- 否则，调用 unbound_search(t, j) 在哈希部分中继续查找边界。
-- 行为：从 t[1] 开始计算连续非 nil 元素的个数，遇到第一个 nil 就停止。
            
static int unbound_search (Table *t, unsigned int j) {
  unsigned int i = j;  /* i is zero or a present index */
  j++;
  /* find `i' and `j' such that i is present and j is not */
  while (!ttisnil(luaH_getint(t, j))) {
    i = j;
    j *= 2;
    if (j > cast(unsigned int, MAX_INT)) {  /* overflow? */
      /* table was built with bad purposes: resort to linear search */
      i = 1;
      while (!ttisnil(luaH_getint(t, i))) i++;
      return i - 1;
    }
  }
  /* now do a binary search between them */
  while (j - i > 1) {
    unsigned int m = (i+j)/2;
    if (ttisnil(luaH_getint(t, m))) j = m;
    else i = m;
  }
  return i;
}
-- 数组的长度为 j，hash 部分从 j+1 开始遍历，j 每次扩大两倍，找到t[j] 不为空， t[2*j] 为空，然后通过二分法查找，找到最终的值。
```

2. 5.2+

> 1. 检查数组部分
>
>    - Lua 表有一个数组部分，用于存储从 1 开始的连续整数键，容量由 asize 表示。
>    - 如果 asize > 0，说明表有数组部分，函数会优先处理这部分。
>
> 2. 利用长度提示（hint）
>
>    - 表中有一个缓存的长度提示字段 lenhint，记录上次计算的长度。
>    - 初始时，limit = *lenhint(t)，如果 limit 为 0，则设为 1（因为 Lua 索引从 1 开始）。
>
> 3. 根据 t[limit] 的状态查找边界
>
>    - 情况 1：t[limit] 为空（nil）
>
>      - 说明边界在 limit 之前。
>      - **附近查找**：从 limit 向前最多检查 4 次（maxvicinity = 4），寻找一个位置 i，使得 t[i] 非 nil 且 t[i+1] 为 nil。若找到，返回 i 并更新提示。
>      - **二分查找**：如果附近没找到，在 [0, limit) 范围内进行二分查找，找到边界并更新提示。
>
>    - 情况 2：t[limit] 非空
>
>      - 说明边界在 limit 之后。
>
>      - **附近查找**：从 limit 向后最多检查 4 次，若 limit < asize，寻找 t[i] 非 nil 且 t[i+1] 为 nil 的位置。若找到，返回 i 并更新提示。
>
>      - 进一步检查
>
>        ：
>
>        - 如果数组部分的最后一个元素 t[asize] 为空，则在 [limit, asize) 范围内进行二分查找，找到边界并更新提示。
>        - 如果 t[asize] 非空，说明数组部分没有边界，设置提示为 asize，并进入哈希部分的处理。
>
> 4. 处理哈希部分
>
>    - 如果没有数组部分（asize == 0）或 t[asize] 非空，检查哈希部分。
>
>    - 情况分析
>
>      ：
>
>      - 如果表是“dummy”（空的或无哈希部分）或 t[asize + 1] 为空，返回 asize。
>      - 如果 t[asize + 1] 非空，调用 hash_search(t, asize) 在哈希部分中查找最大的 n > asize，使得 t[n] 非 nil 且 t[n+1] 为 nil，并返回该值。
>
> ------
>
> #### **关键优化**
>
> - **提示机制**：通过缓存长度提示，避免重复计算。
> - **附近查找**：在提示附近进行快速线性查找（最多 4 次），减少查找开销。
> - **二分查找**：当附近查找失败时，使用二分查找高效定位边界。
> - **数组与哈希分离**：优先处理数组部分，若必要再处理哈希部分，充分利用 Lua 表的结构特性。
>
> ------
>
> #### **总结**
>
> luaH_getn 函数通过结合长度提示、附近线性查找和二分查找，高效地计算 Lua 表的“长度”。它首先尝试利用数组部分的连续性快速找到边界，若无法满足则在哈希部分继续查找。这种设计在表有大量连续整数键时性能优异，同时也能正确处理稀疏表或混合表的情况。最终返回的 lua_Unsigned 值反映了表的有效长度，适用于 Lua 的 # 操作符。

https://www.jianshu.com/p/1e8ab8fe55e4

> 这段代码是 Lua 中用于获取表（Table）数组部分有效长度的函数 `luaH_getn` 的实现。以下是代码的逐段解释：
>
> ------
>
> ### **1. 初始条件检查**
>
> c
>
> 
>
> 复制
>
> 
>
> 下载
>
> ```
> unsigned int limit = t->alimit;
> if (limit > 0 && isempty(&t->array[limit - 1])) {
> ```
>
> - **目的**：检查数组部分的“逻辑边界”是否需要调整。
> - `alimit` 是表的预分配数组大小，但可能不反映实际的有效元素边界。
> - 如果 `alimit-1` 位置的元素为空（`isempty`），说明当前 `alimit` 可能不是有效边界，需要进一步处理。
>
> ------
>
> ### **2. 快速调整边界**
>
> c
>
> 
>
> 复制
>
> 
>
> 下载
>
> ```
> if (limit >= 2 && !isempty(&t->array[limit - 2])) {
>     if (ispow2realasize(t) && !ispow2(limit - 1)) {
>         t->alimit = limit - 1;
>         setnorealasize(t);
>     }
>     return limit - 1;
> }
> ```
>
> - **场景**：如果 `limit-2` 位置非空，且 `limit-1` 为空，说明 `limit-1` 是有效边界。
> - **优化**：如果表的真实数组大小是 2 的幂（`ispow2realasize`），但 `limit-1` 不是 2 的幂，则将 `alimit` 设为 `limit-1`，并标记真实大小不再等于 `alimit`（`setnorealasize`）。
> - **返回**：直接返回 `limit-1` 作为有效边界。
>
> ------
>
> ### **3. 二分查找边界**
>
> c
>
> 
>
> 复制
>
> 
>
> 下载
>
> ```
> else {
>     unsigned int boundary = binsearch(t->array, 0, limit);
>     if (ispow2realasize(t) && boundary > luaH_realasize(t) / 2) {
>         t->alimit = boundary;
>         setnorealasize(t);
>     }
>     return boundary;
> }
> ```
>
> - **场景**：当无法快速确定边界时，在 `[0, limit]` 范围内通过二分查找 (`binsearch`) 找到第一个空元素的位置作为边界。
> - **优化**：如果真实数组大小是 2 的幂，且找到的边界超过其一半，更新 `alimit` 为这个边界，以提高后续操作效率。
>
> ------
>
> ### **4. 处理未调整的 `alimit`**
>
> c
>
> 
>
> 复制
>
> 
>
> 下载
>
> ```
> if (!limitequalsasize(t)) {
>     if (isempty(&t->array[limit])) return limit;
>     limit = luaH_realasize(t);
>     if (isempty(&t->array[limit - 1])) {
>         unsigned int boundary = binsearch(t->array, t->alimit, limit);
>         t->alimit = boundary;
>         return boundary;
>     }
> }
> ```
>
> - **场景**：如果 `alimit` 不等于真实数组大小（`limitequalsasize`），说明数组有更多元素。
> - **检查空洞**：
>   - 如果 `array[limit]` 为空，直接返回 `limit` 作为边界。
>   - 否则，将 `limit` 设为真实数组大小，继续检查末尾是否为空。
>   - 如果末尾为空，通过二分查找调整 `alimit`。
>
> ------
>
> ### **5. 处理哈希部分的影响**
>
> c
>
> 
>
> 复制
>
> 
>
> 下载
>
> ```
> lua_assert(limit == luaH_realasize(t) &&
>            (limit == 0 || !isempty(&t->array[limit - 1])));
> if (isdummy(t) || isempty(luaH_getint(t, cast(lua_Integer, limit + 1))))
>     return limit;
> else
>     return hash_search(t, limit);
> ```
>
> - **断言**：确保 `limit` 是真实数组大小，且末尾元素非空。
> - **哈希检查**：
>   - 如果表是虚拟表（`isdummy`）或 `limit+1` 在哈希部分为空，返回 `limit`。
>   - 否则，调用 `hash_search` 在哈希部分中查找可能的连续键（如 `limit+1`），扩展有效长度。
>
> ------
>
> ### **总结**
>
> - **核心逻辑**：通过数组部分的预分配大小 (`alimit`)、二分查找和哈希部分的检查，高效确定表的有效长度。
> - **优化点**：维护 `alimit` 为 2 的幂以提高内存分配效率，动态调整逻辑边界。
> - **典型用例**：Lua 中 `#` 运算符获取数组长度时调用此函数，正确处理数组空洞和哈希部分。

```lua

/*
** Try to find a boundary in table 't'. (A 'boundary' is an integer index
** such that t[i] is present and t[i+1] is absent, or 0 if t[1] is absent
** and 'maxinteger' if t[maxinteger] is present.)
** (In the next explanation, we use Lua indices, that is, with base 1.
** The code itself uses base 0 when indexing the array part of the table.)
** The code starts with 'limit = t->alimit', a position in the array
** part that may be a boundary.
**
** (1) If 't[limit]' is empty, there must be a boundary before it.
** As a common case (e.g., after 't[#t]=nil'), check whether 'limit-1'
** is present. If so, it is a boundary. Otherwise, do a binary search
** between 0 and limit to find a boundary. In both cases, try to
** use this boundary as the new 'alimit', as a hint for the next call.
**
** (2) If 't[limit]' is not empty and the array has more elements
** after 'limit', try to find a boundary there. Again, try first
** the special case (which should be quite frequent) where 'limit+1'
** is empty, so that 'limit' is a boundary. Otherwise, check the
** last element of the array part. If it is empty, there must be a
** boundary between the old limit (present) and the last element
** (absent), which is found with a binary search. (This boundary always
** can be a new limit.)
**
** (3) The last case is when there are no elements in the array part
** (limit == 0) or its last element (the new limit) is present.
** In this case, must check the hash part. If there is no hash part
** or 'limit+1' is absent, 'limit' is a boundary.  Otherwise, call
** 'hash_search' to find a boundary in the hash part of the table.
** (In those cases, the boundary is not inside the array part, and
** therefore cannot be used as a new limit.)
*/
lua_Unsigned luaH_getn (Table *t) {
  unsigned int limit = t->alimit;
  if (limit > 0 && isempty(&t->array[limit - 1])) {  /* (1)? */
    /* there must be a boundary before 'limit' */
    if (limit >= 2 && !isempty(&t->array[limit - 2])) {
      /* 'limit - 1' is a boundary; can it be a new limit? */
      if (ispow2realasize(t) && !ispow2(limit - 1)) {
        t->alimit = limit - 1;
        setnorealasize(t);  /* now 'alimit' is not the real size */
      }
      return limit - 1;
    }
    else {  /* must search for a boundary in [0, limit] */
      unsigned int boundary = binsearch(t->array, 0, limit);
      /* can this boundary represent the real size of the array? */
      if (ispow2realasize(t) && boundary > luaH_realasize(t) / 2) {
        t->alimit = boundary;  /* use it as the new limit */
        setnorealasize(t);
      }
      return boundary;
    }
  }
  /* 'limit' is zero or present in table */
  if (!limitequalsasize(t)) {  /* (2)? */
    /* 'limit' > 0 and array has more elements after 'limit' */
    if (isempty(&t->array[limit]))  /* 'limit + 1' is empty? */
      return limit;  /* this is the boundary */
    /* else, try last element in the array */
    limit = luaH_realasize(t);
    if (isempty(&t->array[limit - 1])) {  /* empty? */
      /* there must be a boundary in the array after old limit,
         and it must be a valid new limit */
      unsigned int boundary = binsearch(t->array, t->alimit, limit);
      t->alimit = boundary;
      return boundary;
    }
    /* else, new limit is present in the table; check the hash part */
  }
  /* (3) 'limit' is the last element and either is zero or present in table */
  lua_assert(limit == luaH_realasize(t) &&
             (limit == 0 || !isempty(&t->array[limit - 1])));
  if (isdummy(t) || isempty(luaH_getint(t, cast(lua_Integer, limit + 1))))
    return limit;  /* 'limit + 1' is absent */
  else  /* 'limit + 1' is also present */
    return hash_search(t, limit);
}
                                                                
                                                                
/*
** Try to find a boundary in the hash part of table 't'. From the
** caller, we know that 'j' is zero or present and that 'j + 1' is
** present. We want to find a larger key that is absent from the
** table, so that we can do a binary search between the two keys to
** find a boundary. We keep doubling 'j' until we get an absent index.
** If the doubling would overflow, we try LUA_MAXINTEGER. If it is
** absent, we are ready for the binary search. ('j', being max integer,
** is larger or equal to 'i', but it cannot be equal because it is
** absent while 'i' is present; so 'j > i'.) Otherwise, 'j' is a
** boundary. ('j + 1' cannot be a present integer key because it is
** not a valid integer in Lua.)
*/
static lua_Unsigned hash_search (Table *t, lua_Unsigned j) {
  lua_Unsigned i;
  if (j == 0) j++;  /* the caller ensures 'j + 1' is present */
  do {
    i = j;  /* 'i' is a present index */
    if (j <= l_castS2U(LUA_MAXINTEGER) / 2)
      j *= 2;
    else {
      j = LUA_MAXINTEGER;
      if (isempty(luaH_getint(t, j)))  /* t[j] not present? */
        break;  /* 'j' now is an absent index */
      else  /* weird case */
        return j;  /* well, max integer is a boundary... */
    }
  } while (!isempty(luaH_getint(t, j)));  /* repeat until an absent t[j] */
  /* i < j  &&  t[i] present  &&  t[j] absent */
  while (j - i > 1u) {  /* do a binary search between them */
    lua_Unsigned m = (i + j) / 2;
    if (isempty(luaH_getint(t, m))) j = m;
    else i = m;
  }
  return i;
}

```

以下是 Lua 5.1 和 Lua 5.2+ 版本 luaH_getn 的主要区别：

1. 处理空洞（nil 值）的方式

   ：

   - Lua 5.1

     ：

     - 在数组部分，遇到第一个 nil 就停止，返回其之前的长度。
     - 例如，对于表 t = {1, nil, 3}，luaH_getn 返回 1，因为 t[2] = nil 中断了连续性。

   - Lua 5.2+

     ：

     - 可以跳过中间的 nil，寻找最后一个非 nil 的整数键。
     - 对于同样的表 t = {1, nil, 3}，返回 3，因为 t[3] 非 nil 且 t[4] 为 nil。

2. 查找策略

   ：

   - Lua 5.1

     ：

     - 使用简单的二分查找，在数组部分找到第一个 nil 的位置。
     - 没有长度提示，效率较低。

   - Lua 5.2+

     ：

     - 使用更复杂的查找策略，包括附近查找和二分查找，并利用长度提示优化性能。

3. 对哈希部分的处理

   ：

   - Lua 5.1

     ：

     - 如果数组部分没有 nil，调用 unbound_search 在哈希部分查找边界，逻辑较为简单。

   - Lua 5.2+

     ：

     - 类似地在哈希部分查找，但实现上可能更复杂，支持跳过 nil 的逻辑。