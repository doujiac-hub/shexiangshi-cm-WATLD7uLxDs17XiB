[合集 \- .NET响应式编程(2\)](https://github.com)[1\..NET 响应式编程 System.Reactive 系列文章（一）：基础概念01\-07](https://github.com/VAllen/p/18656600/system-reactive-intro):[wgetCloud机场](https://tabijibiyori.org)2\..NET 响应式编程 System.Reactive 系列文章（二）：深入理解 IObservable 和 IObserver01\-08收起
# **.NET 响应式编程 System.Reactive 系列文章（二）：深入理解 IObservable 和 IObserver**




---


## **引言：为什么我们调整了学习顺序？**


在上一篇文章的结尾，我原本计划在本篇介绍 **System.Reactive** 的基础操作符，比如如何创建、转换和过滤数据流。但在撰写内容时，我意识到，对于刚接触 **System.Reactive** 的读者来说，直接介绍操作符可能有些仓促，因为 **操作符的使用必须建立在对 `IObservable` 和 `IObserver` 这两个核心接口的深刻理解之上**。


正如在传统编程中，你需要先理解 **集合（Collection）** 和 **迭代器（Iterator）** 的本质，才能更好地使用 **LINQ** 操作符一样。而在 Rx 中，**`IObservable` 是数据流的生产者，`IObserver` 是数据流的消费者**，理解这两个接口是掌握 Rx 的基础。


因此，我决定调整顺序，在本篇文章中，**深入介绍 `IObservable` 和 `IObserver` 的核心概念、方法和使用方式**，为后续学习操作符打下坚实的基础。




---


## **IObservable 和 IObserver 的关系**


在 Rx 中，数据流的生产和消费是通过 **观察者模式（Observer Pattern）** 实现的。这种模式定义了两种角色：


* **`IObservable`（可观察对象/数据流的生产者）**
* **`IObserver`（观察者/数据流的消费者）**


二者的关系可以简单理解为：


* **`IObservable` 负责“推送”数据项**。
* **`IObserver` 负责“接收”数据项**。


**订阅（Subscribe）** 是连接这两者的桥梁。当 `IObserver` 订阅一个 `IObservable` 时，数据流开始传递。




---


### **1\. `IObservable` 的定义和职责**


#### **`IObservable` 接口定义**



```
public interface IObservable<out T>
{
    IDisposable Subscribe(IObserver observer);
}

```

**`IObservable` 的职责：**


* 代表一个**数据流**，它可以产生零个、一个或多个数据项。
* 当一个观察者（`IObserver`）订阅这个数据流时，它会调用 **`Subscribe`** 方法，并开始推送数据。
* 数据流可能会因为**正常完成**或**发生错误**而终止。




---


### **2\. `IObserver` 的定义和职责**


#### **`IObserver` 接口定义**



```
public interface IObserver<in T>
{
    void OnNext(T value);
    void OnError(Exception error);
    void OnCompleted();
}

```

**`IObserver` 的职责：**


* 代表一个**数据的消费者**，它对 `IObservable` 提供的数据流做出响应。
* `IObserver` 需要实现三个方法：
	+ **`OnNext(T value)`**：当有新的数据项时调用。
	+ **`OnError(Exception error)`**：当数据流发生错误时调用。
	+ **`OnCompleted()`**：当数据流正常结束时调用。




---


### **3\. `IObservable` 和 `IObserver` 的交互流程**


让我们通过一个实际的交互流程图来直观地理解 **`IObservable` 和 `IObserver`** 的关系：


1. **观察者（Observer）通过 `Subscribe` 方法订阅可观察对象（Observable）。**
2. **可观察对象（Observable）调用 Observer 的 `OnNext` 方法推送数据。**
3. **如果发生错误，可观察对象（Observable）调用 `OnError` 方法终止数据流。**
4. **如果数据流正常结束，可观察对象（Observable）调用 `OnCompleted` 方法终止数据流。**


IObserverIObservableIObserverIObservablealt\[数据流正常结束]\[发生错误]Subscribe()OnNext(T value)OnNext(T value)OnCompleted()OnError(Exception error)

---


### **4\. 示例代码：实现一个简单的 Observable 和 Observer**


为了更好地理解这两个接口，我们从零开始，手动实现一个简单的 `IObservable` 和 `IObserver`。


#### **实现自定义 Observable**



```
using System;
using System.Threading;
using System.Threading.Tasks;

public sealed class SimpleObservable : IObservable<int>
{
	IDisposable IObservable<int>.Subscribe(IObserver<int> observer)
	{
		SimpleDisposable disposable = new();

		Task.Run(() =>
		{
			// 模拟数据的生产，以及假设每次生产都需要时间，消费者可以随时调用Dispose方法取消订阅
			for (int i = 1; i <= 5; i++)
			{
				if (disposable.IsDisposed)
				{
					return;
				}
				observer.OnNext(i);
                // 模拟产生数据需要耗时50毫秒
				Thread.Sleep(50);
			}

			observer.OnCompleted();
		});

		return disposable;
	}

	private sealed class SimpleDisposable : IDisposable
	{
		internal bool IsDisposed { get; private set; }
		void IDisposable.Dispose()
		{
			IsDisposed = true;
			Console.WriteLine("Subscription disposed.");
		}
	}
}

```

#### **实现自定义 Observer**



```
using System;

public sealed class SimpleObserver : IObserver<int>
{
	void IObserver<int>.OnNext(int value) => Console.WriteLine($"Received: {value}");

	void IObserver<int>.OnError(Exception error) => Console.WriteLine($"Error: {error.Message}");

	void IObserver<int>.OnCompleted() => Console.WriteLine("Sequence Completed.");
}

```

#### **订阅和运行**



```
using System;
using System.Threading;

class Program
{
	static void Main(string[] args)
	{
		IObservable<int> observable = new SimpleObservable();
		IObserver<int> observer = new SimpleObserver();

		IDisposable subscription = observable.Subscribe(observer);

        // 模拟消费数据100毫秒后取消订阅
		Thread.Sleep(100);
		subscription.Dispose();
	}
}

```

**输出结果：**



```
Received: 1
Received: 2
Subscription disposed.

```



---


### **5\. 常见问题解答**


#### **Q1：为什么 `Subscribe` 方法返回 `IDisposable`？**


`Subscribe` 方法返回一个 **`IDisposable`** 对象，允许订阅者在不再需要数据流时**取消订阅**，以释放资源，避免内存泄漏。


#### **Q2：`OnError` 和 `OnCompleted` 可以同时调用吗？**


不能。**数据流要么以错误终止，要么正常结束**，二者是**互斥的**。


#### **Q3：`IObservable` 可以被多个 `IObserver` 订阅吗？**


可以。一个 **`IObservable`** 可以被**多个观察者**订阅，每个观察者都会接收到数据流的推送。




---


## **总结**


在本篇文章中，我们深入探讨了 **`IObservable` 和 `IObserver`** 这两个核心接口的定义和职责，并通过代码示例展示了它们如何交互。


### **核心要点：**


1. **`IObservable` 是数据流的生产者**，它负责推送数据。
2. **`IObserver` 是数据流的消费者**，它负责接收和处理数据。
3. **`Subscribe` 方法将生产者和消费者连接起来**，并返回一个 **`IDisposable`** 对象，用于取消订阅。




---


### **下一篇文章预告**



> **《.NET 响应式编程 System.Reactive 系列文章（三）：Subscribe 和 IDisposable 的深入理解》**
> 在下一篇文章中，我们将重点探讨 **`Subscribe` 方法的内部工作机制**、**`IDisposable` 的作用**，以及如何**优雅地管理订阅的生命周期**。敬请期待！


