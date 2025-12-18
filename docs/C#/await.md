---
tags:
    - C#
    - Threading
icon: fontawesome/solid/code
---

> https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await
> https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/
> https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1299-await-expressions

## 概念

- await 的作用是，将其修饰的表达式后面的流程作为回调加给 task；
- 有的描述中会说 “方法会从 await ‘暂停’ 的地方继续执行”，这实际上是因为 `await` 后的代码作为回调被执行了，与 “暂停” 这个概念没有关系；
- “暂停” 这个概念的产生或许是因为调试的时候，看起来 `await` 前后还是在同一个方法里，但前后的流程可能并没有在相同线程中执行（当然也有可能在相同线程中，取决于任务调度）；
- 如果直接调用一个 `Task` 而不 `await`，那么它会在线程池线程上执行，但是无法正常捕获异常；
- 当使用 `await` 被应用到 task 上时，失败的 task 会抛出异常；
- 如果下一步的开始依赖于上一步的结果，那么异步甚至比同步更慢，因为还额外有线程调度的时间；
- 如果几个任务之间是独立的，那么异步执行能够显著提高效率；
- 通过分离 `Task` 的执行和 `await` 来实现独立任务同时在不同线程上的执行；
- Task.Exception 属性是 System.AggregateException 类型的对象，因为异步任务执行的过程中可能会抛出多个异常；
- 如果 Task.Exception 是空的，那么会创建一个 AggregateException 异常，并抛出这集合中的第一个异常；
- `async/await` 在语义上与 `ContinueWith` 类似，但实际上编译器并没有将 `await` 表达式直接转换成 `ContinueWith`。编译器生成优化过的状态机代码，提供类似的运行逻辑。
- `await` 的方法并不是一定要有线程切换，`await` 只是标记了一个同步上下文切换点；

## 几个例子

```C#
public class AwaitOperator
{
    public static async Task Main()
    {
        Task<int> downloading = DownloadDocsMainPageAsync();
        Console.WriteLine($"{nameof(Main)}: Launched downloading.");

        int bytesLoaded = await downloading;
        Console.WriteLine($"{nameof(Main)}: Downloaded {bytesLoaded} bytes.");
    }

    private static async Task<int> DownloadDocsMainPageAsync()
    {
        Console.WriteLine($"{nameof(DownloadDocsMainPageAsync)}: About to start downloading.");

        var client = new HttpClient();
        byte[] content = await client.GetByteArrayAsync("https://learn.microsoft.com/en-us/");

        Console.WriteLine($"{nameof(DownloadDocsMainPageAsync)}: Finished downloading.");
        return content.Length;
    }
}
// Output similar to:
// DownloadDocsMainPageAsync: About to start downloading.
// Main: Launched downloading.
// DownloadDocsMainPageAsync: Finished downloading.
// Main: Downloaded 27700 bytes.
```

- Task 完成后会通知（实际是以回调的形式执行 await 后的操作）

```C#
Coffee cup = PourCoffee();
Console.WriteLine("Coffee is ready");

Task<Egg> eggsTask = FryEggsAsync(2);
Task<HashBrown> hashBrownTask = FryHashBrownsAsync(3);
Task<Toast> toastTask = ToastBreadAsync(2);

Toast toast = await toastTask;
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("Toast is ready");
Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

Egg eggs = await eggsTask;
Console.WriteLine("Eggs are ready");
HashBrown hashBrown = await hashBrownTask;
Console.WriteLine("Hash browns are ready");

Console.WriteLine("Breakfast is ready!");
```
