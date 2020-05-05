## Circuit Breaker Pattern

**Circuit Breaker** 用来处理那些可能需要未知时间恢复的远程服务或资源的失败。这个模式可以提高系统的稳定性和弹性。

### Content and Problem

在分布式环境中，调用远程服务和资源会因为一些原因短暂失败，比如缓慢的网络连接，超时，或者资源被过渡使用或暂时不可用。这些失败都能在短时间自动恢复，而且一个健壮的系统应该能够通过一些策略比如retry来处理他们。

然而，有些情况下错误会花更长的时间恢复，比如碰到了没有预见的事件。这些错误的范围也可以从部分失败到整个服务不可用。在这种情况下，用户不应该继续retry，而是接受服务不可用，采取其他方法应对。

而且，如果服务过于繁忙，部分失败可以引起整个服务的级联失败。比如，调用一个服务可能配置了超时，然后直接返回错误信息。然而，超时操作会导致并发情况下很多请求被阻塞整个超时时间过去。而这些被阻塞的请求可能会暂用系统内存，线程，数据库连接等。最后这些资源的耗尽，会导致其他共享这些资源的无关服务失败。这个情况下，最好让服务直接失败，只有在可能成功的情况下去尝试。当前用更短的超时可能会对这种情况有帮助，但是明显超时不能设置太小，否则会影响正常的服务。

### Solution

**Circuit Breaker**模式，可以防止应用去尝试一个大概率会失败的请求。避免应用等待失败或者失败恢复或者浪费CPU资源当应用知道这个失败会较长时间内存在。这个模式还能使应用去查看失败是否恢复。恢复了，应用就可以正常调用。

> Circuit Breaker和retry的目的不同。Retry的目的是觉得失败将会恢复，所以应用要尝试。Circuit Breaker是防止应用去执行大概率失败的请求。一个应用可以包含这两种模式，可以通过Circuit Breaker去使用Retry。当然Retry必须能对Circuit Breaker的异常进行处理，及时放弃Retry如果返回异常是一个不可短暂恢复的错误。

一个Circuit Breaker 是作为可能失败服务的代理，可以监控服务近期的失败数量，通过这个信息来判断服务是否继续，还是直接返回。

这个代理可以通过状态机来实现，会有以下这些状态（模仿真实电子断路器）：

* **关闭** 应用请求可以被允许接入。通过记录近期失败次数，当次数突破某个限值，进入**开启**状态，然后倒计时器开启，倒计时结束，进入**半开启**状态。

  > 倒计时给予系统恢复时间

* **开启** 请求直接失败。

* **半开启** 一定数量的请求可以被执行。如果这些请求成功了，就认为之前的错误恢复了，就转为**关闭**状态（错误计数器重置）。一旦有失败，就认为还没有完全恢复，转为**开启**状态，在打开倒计时。

  > **半开启**状态有利于避免系统一打开，直接被大量堆积请求冲击。当系统恢复时，应该可以支持一定数量的请求，直到完全恢复，但是如果直接全量，可能导致服务再次失败。

**Circuit Breaker**模式提高了系统的稳定性，当系统从故障恢复，减少了对性能的影响。可以保证系统响应时间，通过快速失败，而不是超时等待。如果**Circuit Breaker**在每次状态转换的时候能够发出事件通知，那么可以帮助系统监控由它保护的服务的状态，或者通知管理员当它被开启。

这个模式是可以根据失败的类型而定制的。比如，可以配置一个倒计时器不断增加的**Circuit Breaker**，就是第一次**开启** ，先倒计时几秒，如果还没恢复，就倒计时几分钟。一些情况下，**开启**状态，可以返回默认值，而不是异常。

### Issues and Considerations

使用这个模式考虑这些方面：

1. **异常处理**：使用**Circuit Breaker**的应用肯定要处理由他返回的异常，而且处理方式是定制化的。比如，应用可以降级，使用另一个服务，返回相同的数据等，然后通知用户，过一段时间在尝试。
2. **异常类型**：一个请求失败可能是多种原因导致的，有的失败会更严重。比如，请求可能因为远程服务挂了，需要几分钟才能恢复，或者服务因为请求量大而超时。**Circuit Breaker**需要根据不同类型，采取不同的策略。比如超时异常可能需要更大的次数才能激活到**开启**状态，而有些严重异常次数要求低。
3. **日志**：**Circuit Breaker**需要能够记录失败次数（成功次数），然后让管理员能够监控。
4. **可恢复性**：可配置的恢复模式。比如，如果保持**开启**时间太长，可以导致其他异常，即使之前触发**开启**的异常已经恢复了。同时，从**开启**到**半开启**的改动，可能影响服务的响应时间。
5. **测试失败操作**：在**开启**状态下，除了倒计时器，还可以用ping远程服务或资源的方式来确定是否恢复。Ping可以是远程服务提供的一个health接口，这个可以阅读 Health EndPoint Monitoring Pattern.
6. **手动覆盖**：如果一个服务的恢复时间非常不可控，最好提供一个人口重置，让管理员能手动关闭**Circuit Breaker**。当然最好也能让管理员直接**开启**这个**Circuit Breaker**。
7. **并发**：**Circuit Breaker**应该能够支持大并发，而不会额外增加开销
8. **资源差异化**：注意当使用时增对的服务有多个供应商。比如，增对分布式数据库访问，某个shard可能有问题会导致错误次数增加，一旦**开启**会导致其他健康shard也不可用。
9. **加速短路**：有些情况下，一些错误表示应该马上**短路**，而不是在积累失败次数，比如数据库访问不可用，直接到**开启**，然后等待一定时间在重试。
10. **重现失败请求**：在**开启**时，除了直接**短路**，**Circuit Breaker**还能记录失败的请求，然后在服务恢复时，重试这些失败请求。
11. **不合适的超时**：超长的超时时间并不能完全保护系统。因为太长的超时时间，会出现太多的访问被积压。

### When to use this pattern

使用：

* 系统使用外部服务或者共享资源，而且有可能失败的情况下。

不适用：

* 系统内部数据访问，比如local cache
* 作为处理业务异常的代替品

### 例子

在一个web服务中，少数页面是热门的，数据来自外部服务。如果服务的缓存较少，那么大部分请求都会落到外部服务调用上。web和服务的连接有超时，如果超过了而没有响应，就会抛出异常。

然而，如果服务失败而系统又十分繁忙，用户会被迫等待整个超时时间段。最终会导致整体性能下降，其他用户也不可用。

扩展整个系统，增加更多的web server和LB，可以延缓问题，但是不能解决，最终还是会耗尽资源。

用**Circuit Breaker**包装整个服务调用和数据获取，可以更优雅的处理这个问题。用户请求还是会失败，更快的失败，也不会耗尽系统资源。

```c#
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

`State` 当前**Circuit Breaker**状态，**Open**,**Half-Open**,**Closed** `CircuitBreakerStateEnum`中定义。`Trip` -> Open, Reset -> Closed。

`CircuitBreaker` 类创建了`ICircuitBreakerStateStore`来保存信息, `ExectueAction`包装了action，类似代理。如果action失败，就会调用`TrackException`方法来**开启**。

```c#
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```



下面代码更具体的逻辑，先判断是否**开启**，开启了要判断开启多长时间，`OpenToHalfOpenWaitTime` ，到了就会转到**半开启**，然后在调用对应的action。

如果成功，更新到**关闭**，如果失败，就会回到**开启**，并且更新lastexception等信息。

另外，在**半开启**的时候使用了lock来操作action和回到**关闭**。

```c#
...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken);
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
        }
        catch (Exception ex)
        {
          // If there's still an exception, trip the breaker again immediately.
          this.stateStore.Trip(ex);

          // Throw the exception so that the caller knows which exception occurred.
          throw;
        }
        finally
        {
          if (lockTaken)
          {
            Monitor.Exit(halfOpenSyncObject);
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```



使用**CircuitBreaker**，类似**proxy**，需要执行的真正操作，封装到`ExecutionAction`中，而且要处理可能出现的`CircuitBreakerOpenException`，保证**CircuitBreaker**开启的时候不影响系统正常运行。

```c#
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```



### Related Pattern

[BulkHead Pattern](#https://patrickchen.cn/?p=157)

Retry Pattern

Health EndPoint Monitoring Pattern



