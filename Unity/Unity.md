# UnityAPI

## Mathf

### Lerp和Inverse Lerp

Inverse Lerp求的是一个值在min和max之间的比例

```
t = (value - min) / (max - min)
```

Lerp求的是一个插值的数值

```
result = a + (b - a) * t
```

### **🔹 `InvokeRepeating` 作用**

`InvokeRepeating` 是 Unity 提供的方法，用于 **在指定时间后开始重复调用某个方法，并以固定间隔重复执行**。

------

### **🔹 语法**

```
csharp


复制编辑
InvokeRepeating(string methodName, float time, float repeatRate);
```

| 参数         | 作用                        |
| ------------ | --------------------------- |
| `methodName` | 要调用的方法名称（字符串）  |
| `time`       | **延迟多少秒后** 开始执行   |
| `repeatRate` | **每隔多少秒** 重新执行一次 |

------

### **🔹 示例**

#### **🌟 让一个物体每 2 秒执行一次 `PrintMessage()`**

```
csharp复制编辑void Start()
{
    InvokeRepeating("PrintMessage", 1f, 2f);
}

void PrintMessage()
{
    Debug.Log("每 2 秒触发一次！");
}
```

✅ **运行结果：**

- **1 秒后**，`PrintMessage()` 执行 **（第一次调用）**。
- **之后每 2 秒**，`PrintMessage()` 再次执行。

------

## **🔹 取消 `InvokeRepeating`**

如果想 **停止调用**，可以使用：

```
csharp


复制编辑
CancelInvoke("方法名");
```

#### **🌟 停止 `InvokeRepeating`**

```
csharp复制编辑void Start()
{
    InvokeRepeating("PrintMessage", 1f, 2f);
    Invoke("StopRepeating", 10f); // 10 秒后停止
}

void PrintMessage()
{
    Debug.Log("每 2 秒触发一次！");
}

void StopRepeating()
{
    CancelInvoke("PrintMessage");
    Debug.Log("停止重复调用！");
}
```

✅ **运行结果：**

- **1 秒后**，开始循环执行 `PrintMessage()`。
- **每 2 秒触发一次**。
- **10 秒后**，`StopRepeating()` 调用 `CancelInvoke()`，停止重复调用。

------

## **🔹 `InvokeRepeating` vs. `Coroutine`**

| **功能**         | **InvokeRepeating** | **Coroutine（协程）**            |
| ---------------- | ------------------- | -------------------------------- |
| **定时重复执行** | ✅ 可以              | ✅ 可以                           |
| **控制执行次数** | ❌ 不能（无限循环）  | ✅ `for` / `while` 轻松控制       |
| **暂停 & 继续**  | ❌ 不能暂停          | ✅ `yield return` 控制            |
| **提前终止**     | ✅ `CancelInvoke()`  | ✅ `StopCoroutine()`              |
| **代码复杂度**   | ✅ 简单              | ⚠ 稍微复杂                       |
| **等待其他条件** | ❌ 只能固定间隔      | ✅ `WaitUntil` / `WaitForSeconds` |

------

## **🔹 用协程替代 `InvokeRepeating`**

如果你想要**更灵活的控制（如暂停、提前终止、动态调整时间间隔）**，可以使用 **协程** 代替 `InvokeRepeating`。

#### **🌟 用协程实现 `InvokeRepeating`**

```
csharp复制编辑IEnumerator RepeatMessage(float interval)
{
    while (true) // 无限循环
    {
        Debug.Log("每 " + interval + " 秒触发一次！");
        yield return new WaitForSeconds(interval); // 等待 interval 秒
    }
}

void Start()
{
    StartCoroutine(RepeatMessage(2f));
}
```

✅ **优点：**

- 可以用 **`yield return`** 让函数**暂停执行**，等待一定时间后继续。
- 可以 **随时 `StopCoroutine` 终止**，更加灵活！

------

## **🔹 适用场景**

✅ **使用 `InvokeRepeating`**

- 简单的 **无限重复** 调用（如**自动攻击、背景音乐循环**）。
- 没有 **暂停、动态修改间隔、提前终止** 的需求。

✅ **使用 `Coroutine`**

- **可以暂停**、恢复、控制执行次数（如**倒计时、动画控制**）。
- 需要 **等待特定条件** 才执行下一次循环（如**敌人巡逻行为**）。

------

## **🔹 结论**

1. **`InvokeRepeating`** 用于**固定间隔重复调用**，简单易用。
2. **可以用 `CancelInvoke("方法名")` 停止 `InvokeRepeating`**。
3. **协程（Coroutine）更灵活**，可以控制暂停、恢复、条件等待等。
4. **如果只是简单的定时调用，`InvokeRepeating` 更方便**。
5. **如果需要动态调整执行间隔或暂停，推荐 `Coroutine`**。

🚀 **如果你的任务需要无限循环执行，`InvokeRepeating` 很方便！如果需要更强的控制，`Coroutine` 更适合！** 😃



## FixedUpdate

https://zhuanlan.zhihu.com/p/30335370
