# Delegate和Event

`delegate` 和 `event` 是 C# 中的两个相关概念，但它们各自有不同的用途和特点。理解它们的区别和相互关系对于编写可扩展和高效的代码非常重要。

### 1. 什么是 `delegate`？
`delegate` 是一种类型，可以像函数指针一样保存对方法的引用。它可以让你定义一个可以持有某种方法签名的变量，随后通过这个变量来调用相关的方法。它是一种类型安全的函数指针，常用于回调机制和事件处理。

**声明 `delegate`**：
```csharp
public delegate void MyDelegate(string message);
```

这里 `MyDelegate` 是一个委托，声明了它可以引用任何返回类型为 `void`，并且接受一个 `string` 参数的方法。

**使用 `delegate` 的示例**：
```csharp
public class DelegateExample
{
    public delegate void MyDelegate(string message);

    public void Execute()
    {
        MyDelegate del = PrintMessage;
        del("Hello from delegate!");
    }

    private void PrintMessage(string msg)
    {
        Console.WriteLine(msg);
    }
}
```

在这个例子中：
- `MyDelegate` 是一个委托类型，指向一个接受 `string` 参数的方法。
- `PrintMessage` 方法与 `MyDelegate` 匹配。
- 你可以通过 `del("Hello from delegate!")` 来调用 `PrintMessage`。

### 2. 什么是 `event`？
`event` 是对 `delegate` 的一种封装，用于实现观察者模式，主要用于**发布-订阅**场景。它使得类能够提供一个可以被外部代码订阅的事件，同时确保只有订阅者能够触发这些事件，而不能随意调用。这是为了实现更高的安全性和封装。

**声明 `event`**：
```csharp
public event MyDelegate OnSomethingHappened;
```
在这里，`OnSomethingHappened` 是一个事件，它基于 `MyDelegate` 委托类型。

**事件的使用示例**：
```csharp
public class EventExample
{
    // 声明一个委托类型
    public delegate void MyDelegate(string message);

    // 声明一个事件
    public event MyDelegate OnSomethingHappened;

    public void TriggerEvent()
    {
        if (OnSomethingHappened != null)
        {
            OnSomethingHappened("Event triggered!");
        }
    }
}

public class Subscriber
{
    public void Subscribe(EventExample eventExample)
    {
        eventExample.OnSomethingHappened += HandleEvent;
    }

    private void HandleEvent(string msg)
    {
        Console.WriteLine(msg);
    }
}
```

在这个例子中：
- `OnSomethingHappened` 是一个事件，它基于 `MyDelegate`。
- `TriggerEvent` 方法会检查是否有任何订阅者，然后通过事件通知它们。
- 在 `Subscriber` 类中，`Subscribe` 方法用于订阅事件，每当事件被触发时，`HandleEvent` 方法都会被调用。

### 3. `delegate` 与 `event` 的区别
1. **可见性和封装性**：
   - `delegate` 可以直接在类外部被调用。
   - `event` 只能在声明它的类内部触发，外部代码只能订阅或取消订阅，但不能触发事件。这提高了封装性和安全性，防止其他类随意地触发事件。

2. **典型使用场景**：
   - **`delegate`**：用于实现回调函数或将逻辑通过函数传递。
   - **`event`**：用于发布和订阅模式，比如按钮点击事件、游戏中玩家行为触发等。

3. **声明与使用**：
   - `delegate` 是一种函数类型，可以在代码中自由使用。
   - `event` 是对 `delegate` 的一种约束，用于实现订阅和发布机制，只能在类内部触发。

### 4. `event` 的底层机制
实际上，`event` 是基于 `delegate` 的，定义事件时，编译器会在幕后生成一些代码，使得 `event` 可以更安全地使用：
- 生成私有的委托字段，用于存储所有订阅的方法。
- 提供 `add` 和 `remove` 访问器来订阅或取消订阅事件。
- 防止直接通过 `event` 调用委托的方法。

### 5. 一个实际示例来展示区别
假设你有一个按钮，按下这个按钮时需要触发一些响应：

- 使用 `delegate`：
  - 任何代码都可以随意调用你的 `delegate`，这意味着可以在不符合预期的情况下触发方法。
- 使用 `event`：
  - 外部代码只能订阅按钮的点击事件，而不能随意触发按钮的点击。这样你的按钮逻辑就得到了保护，只有按钮的代码可以控制什么时候调用事件。

**示例：安全的事件触发（只能类内部调用）**

```csharp
public class Button
{
    public delegate void ButtonClickHandler(string buttonName);
    public event ButtonClickHandler OnClick;

    public void Click()
    {
        Console.WriteLine("Button clicked");
        OnClick?.Invoke("SubmitButton");
    }
}

public class Example
{
    public static void Main(string[] args)
    {
        Button button = new Button();
        button.OnClick += (buttonName) => Console.WriteLine($"Button {buttonName} was clicked");

        // 点击按钮
        button.Click();

        // 错误示例：外部代码无法调用 OnClick，因为它是 event
        // button.OnClick.Invoke("TestButton"); // 编译错误
    }
}
```
这里 `OnClick` 是事件，外部代码不能直接调用它的 `Invoke()` 方法，这样确保了按钮的逻辑只有在按钮被点击时触发，而不会被其他类意外调用。

### 总结
- **`delegate`** 是一种类型，可以用于保存方法引用，并且在任何时候可以自由调用。
- **`event`** 是基于 `delegate` 的一种封装，只能在类内部触发，用于实现安全的发布-订阅模式，适用于事件处理场景。
- 在设计中，`delegate` 更加灵活，但 `event` 提供了额外的封装性以确保事件只能按预期的方式被触发。

`event` 通常用于需要发布和订阅场景的地方，而 `delegate` 则用于更灵活的回调机制。

本质上，delegate和C++的std::funtional很类似。类似于函数指针，而event是一个修饰符，用来约束delegate对象的行为。

## Invoke和直接调用

`Invoke` 和直接调用委托之间的区别在于，`Invoke` 是调用委托实例的标准方法，且通常带有空检查，以确保委托不是 `null` 时再进行调用。而直接调用委托则可能引发空引用异常。以下是详细说明两者之间的不同：

### 1. 什么是 `Invoke`？

- **`Invoke()`** 是用于执行委托引用的方法，相当于调用委托内部存储的方法列表。
- 当你有一个委托实例时，使用 `Invoke()` 可以安全地调用它。
- 通常在调用委托前，使用 `?.`（空条件运算符）或进行空检查，确保委托引用不为 `null`，从而避免引发空引用异常。

### 2. `Invoke()` 和直接调用的区别

#### 2.1 安全性检查

- 当你调用委托时，通常需要先确保该委托不为 `null`。这可以通过显式的 `null` 检查或使用空条件运算符 `?.` 来实现。
- 例如，使用 `OnMove?.Invoke(this, EventArgs.Empty)` 的写法，可以在委托不为 `null` 时调用它。
- 如果你直接调用 `OnMove(this, EventArgs.Empty)` 而不做空检查，则当没有任何订阅者时，`OnMove` 的值为 `null`，这将导致**空引用异常** (`NullReferenceException`)，进而导致程序崩溃。

示例：

```
csharp复制代码public event EventHandler OnMove;

public void TriggerMove()
{
    // 使用空条件运算符和 Invoke 调用委托，确保不为 null 时才调用
    OnMove?.Invoke(this, EventArgs.Empty);  // 线程安全的调用方式
}
```

在这里：

- `OnMove?.Invoke(...)` 仅在 `OnMove` 不为 `null` 时执行，避免了潜在的异常。
- 如果没有订阅者，`OnMove` 为 `null`，`Invoke` 不会执行，且程序不会出错。

#### 2.2 线程安全性

- `Invoke` 可以配合空条件运算符或者使用其他的空检查逻辑，来确保调用的安全性和避免竞态条件的出现。
- 委托调用会存在一个时间窗口：检查是否为 `null` 到调用方法之间，可能有其他线程对该委托进行了修改。
- 如果手动使用 `if (OnMove != null)` 来检查，并紧接着调用 `OnMove(...)`，在多线程的环境中可能会产生竞态条件。在这段时间内，其他线程可能将 `OnMove` 的引用设置为 `null`，导致后续的直接调用引发空引用异常。

示例：

```
csharp复制代码public void TriggerMoveThreadSafe()
{
    // 创建一个局部副本以确保线程安全
    EventHandler handler = OnMove;
    handler?.Invoke(this, EventArgs.Empty);
}
```

- 在多线程场景中，创建局部副本（`handler`）可以避免在检查是否为 `null` 到实际调用之间的时间窗口内委托的变化，确保委托调用是安全的。

### 3. `Invoke` 和直接调用的实际区别

- **安全性**：使用 `Invoke()` 的时候通常会有安全性的检查，例如空条件运算符 `?.`，这有助于避免空引用异常，而直接调用 `OnMove(...)` 会在委托为空时导致程序崩溃。
- **阅读性**：使用 `Invoke()` 是一种明确的方式，表示这是对一个委托的调用，这样代码更易读，特别是在存在多个委托的情况下，方便理解代码行为。
- **语法糖**：在 C# 中，当你写 `OnMove(this, EventArgs.Empty)` 时，实际上编译器会将它编译为对 `Invoke()` 的调用。因此，这两者在功能上是一致的，区别只在于是否进行了安全性检查以及代码的可读性。

### 4. 代码示例比较

```
csharp复制代码public class Example
{
    // 定义一个事件
    public event EventHandler OnMove;

    public void Move()
    {
        // 方式 1：直接调用（如果没有订阅者，直接调用会引发异常）
        // OnMove(this, EventArgs.Empty); // 可能会导致 NullReferenceException

        // 方式 2：使用 Invoke，并且带有空条件运算符进行安全性检查
        OnMove?.Invoke(this, EventArgs.Empty);  // 安全的调用方式，避免空引用异常
    }
}
```

### 5. 总结

- **直接调用委托**（如 `OnMove(...)`）在委托为 `null` 时会导致**空引用异常**，所以通常要避免这种调用方式。
- **`Invoke()` 方法** 可以用于调用委托，通常与空条件运算符 `?.` 配合使用以确保安全性。例如，`OnMove?.Invoke(...)` 这种方式不仅避免了空引用异常，而且提高了代码的线程安全性和可读性。
- 在多线程环境中，直接调用委托（即使使用空条件运算符）也可能存在竞态条件，因此通常采用局部变量副本的方法来保证线程安全。

使用 `Invoke()` 和空条件运算符是调用委托的安全和推荐方式，可以确保代码健壮性，并避免一些潜在的崩溃风险。