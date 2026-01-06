---
tags:
    - C#
    - Threading
icon: fontawesome/solid/code
---

## 概述

从思路上来说 `CancellationToken` 是 .NET 提供的一种类型，这种类型的变量被用来从一个任务的外部终止任务。

之所以能够实现“终止”的功能，是因为在任务的实现中有类似：

```C#
while (true) {
    if (token.IsCancellationRequested) {
        // ...
        break;
    }
    // ...
}
```

这样的逻辑，使得在任务外部能够控制任务内部的停止。

`CancellationTokenSource` 是 `CancellationToken` 的管理类，可以通过它来执行取消操作。

## Token

```C#
/// <summary>Gets the <see cref="CancellationToken"/> associated with this <see cref="CancellationTokenSource"/>.</summary>
/// <value>The <see cref="CancellationToken"/> associated with this <see cref="CancellationTokenSource"/>.</value>
/// <exception cref="ObjectDisposedException">The token source has been disposed.</exception>
public CancellationToken Token
{
    get
    {
        ThrowIfDisposed();
        return new CancellationToken(this);
    }
}
```

`Token` 是一个计算属性，每次访问都会得到一个新的 `CancellationToken` 示例。

`CancellationToken` 是值类型，它不需要 `Dispose`。

## TransitionToCancellationRequested()

这个私有方法在 `Cancel()` 等方法内部调用：

```C#
/// <summary>Transitions from <see cref="States.NotCanceledState"/> to <see cref="States.NotifyingState"/>.</summary>
/// <returns>true if it successfully transitioned; otherwise, false.</returns>
/// <remarks>If it successfully transitions, it will also have disposed of <see cref="_timer"/> and set <see cref="_kernelEvent"/>.</remarks>
private bool TransitionToCancellationRequested()
{
    if (!IsCancellationRequested &&
        Interlocked.CompareExchange(ref _state, States.NotifyingState, States.NotCanceledState) == States.NotCanceledState)
    {
        // Dispose of the timer, if any.  Dispose may be running concurrently here, but ITimer.Dispose is thread-safe.
        ITimer? timer = _timer;
        if (timer != null)
        {
            _timer = null;
            timer.Dispose();
        }

        // Set the event if it's been lazily initialized and hasn't yet been disposed of.  Dispose may
        // be running concurrently, in which case either it'll have set m_kernelEvent back to null and
        // we won't see it here, or it'll see that we've transitioned to NOTIFYING and will skip disposing it,
        // leaving cleanup to finalization.
        _kernelEvent?.Set();

        return true;
    }

    return false;
}
```

- `_state` 是一个 `volatile` 变量 `volatile` 不保证原子性，因此在写 `_state` 时使用了原子操作作为锁；
- `CompareExchange` 的含义是，如果当前引用等于第三个变量的值，那么将其改为第二个值，并返回修改前的状态；即，如果当前是“NotCanceledState”状态，将其改为“NotifyingState”状态；
- 释放了内部持有的资源，timer 及 WaitHandle；

## Ref

[CancellationTokenSource Class](https://learn.microsoft.com/en-us/dotnet/api/system.threading.cancellationtokensource?view=net-10.0)
