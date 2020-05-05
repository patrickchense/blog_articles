## 健康接口检查模式(Health Endpoint Monitoring Pattern)

通过外部工具定期访问暴露的接口来对功能进行检查。能够帮助验证系统的功能和服务运行正常。

### 上下文和问题

这是一个很好的实践，也是商业需求，监控web应用和后台服务来保证他们的可用性和性能。然而，监控云端服务会比监控本地服务更困难。比如，你对机器没有完全控制权，服务也会依赖其他第三方。

有很多原因会影响云应用，比如网络延迟，底层计算机和存储的性能和可用性，还有网络带宽。服务可能因为这些原因完全不可用或部分不可用。因此，需要定期检查服务可用性来保证一定的标准，这也是SLA的一部分要求。

### 解决

通过发送请求到应用的接口来实现健康监控。系统应该提供必要检查，并返回相关数据。

一个健康监控检查一般包含2个因素：

* 需要应用提供检查报告响应
* 通过工具或框架来分析响应数据

响应返回表示了系统状态，也可以针对任何它用到的组件或服务。延迟或响应时间检查由监控工具或框架提供。下图，提供了大概的情景

其他可能包含在健康监控标准中的检查：

* 检查云存储或数据库可用性和响应时间
* 检查其他包含的服务或者用到的服务

一些服务和工具可用于监控web应用，需要发送请求到一组配置的接口，并对结果根据一组配置的规则进行验证。创建单纯功能测试的服务接口是比较简单的。

典型的监控工具提供的检查包括：

* 验证返回码。比如，HTTP(200)表示接口无错误。监控系统可能还需要检查其他返回码，从而给出更具体的结果
* 检查返回的内容来判断错误，即使返回码200.这只可以检查网页或者服务的一部分错误。比如，检查网页标题或查找一段固定的段落表示正确的网页被返回了
* 检查响应时间，可以综合看网络延迟和系统执行时间。如果时间增长了可能表示存在系统或网络问题。
* 检查系统外的资源或服务，比如CDN。
* 检查SSL的过期时间
* 检查DNS解析时间，确认DNS延迟或故障
* 验证DNS返回URL的正确性。防止针对DNS的恶意攻击

同样这些检查从不同的本地或云端口发起，对于计量和比较响应时间也很有效。理论上最好从最接近真实客户端的地方发起检查，从而得到正确的性能报告。另外更健壮的健康检查，可以帮助判断从哪里部署应用更好，是否需要部署到不止一个数据中心。

测试同时也要针对所有的接口服务，来保证所有用户都可以被覆盖。比如，存储可以针对多个账户，检查也需要覆盖所有。

### 问题和考虑

实现这个模式的时候考虑下面的问题：

怎么评估返回。比如，返回HTTP(200)是不是已经足够反应服务正常？这个提供最基本的应用可用性判断，是这个模式的最小实现，没有提供太多应用关于操作，趋势，和可能出现的问题。

> 确保返回200，只有在服务需要的资源存在且被处理的情况。有些情况，服务会返回200即使需要的web页不存在(404)。

应用暴露的接口数量。一个方法是至少暴露一个核心服务的接口，另一个低优先级的服务，允许不同级别的服务分配给不同的监控接口。也可以考虑暴露更多的接口，至少每个核心服务一个监控，提供额外的监控粒度。比如，一个监控监控检查，可能检查数据库，存储和外部的geo计算服务，每个监控都提供不同程度的正常运行时间和响应时间监控。系统可以还是正常的，即使geo服务，或者一些其他后台任务暂时不可用。

是否使用与一般访问相同的接口进行监视，但是使用为运行状况验证检查而设计的特定路径，例如，通用访问接口上的/ HealthCheck / {GUID} /。 这允许监视工具在应用程序中运行某些功能测试，例如添加新用户注册，登录和下达测试订单，同时还验证通用访问接口是否可用。

响应监视请求而在服务中收集的信息类型，以及如何返回此信息。 大多数现有工具和框架仅查看接口返回的HTTP状态代码。 要返回并验证其他信息，您可能必须创建一个自定义监视实用程序或服务。

收集多少信息。 在检查过程中执行过多的处理可能会使应用程序过载并影响其他用户。 花费的时间可能超过监视系统的超时，因此将应用程序标记为不可用。 大多数应用程序包括诸如错误处理程序和性能计数器之类的工具，它们记录性能和详细的错误信息，这可能足以代替从运行状况验证检查中返回其他信息。

缓存接口状态。 过于频繁地运行健康检查可能会很昂贵。 例如，如果运行状况是通过仪表板报告的，则您不希望仪表板发出的每个请求都触发运行状况检查。 而是定期检查系统运行状况并缓存状态。 公开返回缓存状态的接口。

如何为监视接口配置安全性以保护它们免受公共访问，这可能会使应用程序遭受恶意攻击，冒暴露敏感信息的风险或吸引拒绝服务（DoS）攻击。 通常，应在应用程序配置中完成此操作，以便可以轻松更新它而无需重新启动应用程序。 考虑使用以下一种或多种技术：

* 通过要求身份验证来保护接口。 如果监视服务或工具支持身份验证，则可以通过在请求标头中使用身份验证安全密钥或通过在请求中传递凭据来执行此操作。

  * 使用晦涩或隐藏的接口。 例如，在与默认应用程序URL使用的IP地址不同的IP地址上公开终结点，在非标准的HTTP端口上配置终结点，和/或使用指向测试页面的复杂路径。 通常，您可以在应用程序配置中指定其他终结点地址和端口，并在需要时将这些终结点的条目添加到DNS服务器，以避免必须直接指定IP地址

  * 在接口上公开一种方法，该方法接受诸如键值或操作模式值之类的参数。 根据为此参数提供的值，当收到请求时，代码可以执行特定的测试或一组测试，或者如果无法识别参数值，则返回404（未找到）错误。 可以在应用程序配置中设置识别的参数值。

  * > DoS攻击对执行基本功能测试而不影响应用程序运行的单独接口的影响可能较小。 理想情况下，避免使用可能暴露敏感信息的测试。 如果必须返回可能对攻击者有用的信息，请考虑如何保护接口和数据免遭未经授权的访问。 在这种情况下，仅仅依靠模糊是不够的。 您还应该考虑使用HTTPS连接并对任何敏感数据进行加密，尽管这会增加服务器的负载。

* 如何访问使用身份验证保护的接口。 并非所有工具和框架都可以配置为在运行状况验证请求中包括凭据。可以用些第三方工具 [Pingdom](https://www.pingdom.com/), [Panopta](https://www.panopta.com/), [NewRelic](https://newrelic.com/), 和 [Statuscake](https://www.statuscake.com/)

* 如何确保监视代理程序正常运行。 一种方法是公开一个接口，该接口仅从应用程序配置中返回一个值或一个可用于测试代理的随机值。

* > 另外，还要确保监视系统对自身进行检查，例如自检和内置测试，以避免发出虚假结果。

### 什么时候使用

这个模式适用于：

* 监控web站点和web应用可用性
* 监控web站点和web应用的正确性
* 监控中间层或者共享服务，检测和隔离一个失败的实例或服务，避免影响其他
* 补充应用程序中的现有工具，例如性能计数器和错误处理程序。 运行状况验证检查不会替代应用程序中的日志记录和审核要求。 仪表可以为现有框架提供有价值的信息，该框架可以监视计数器和错误日志以检测故障或其他问题。 但是，如果应用程序不可用，它将无法提供信息。

### 例子

`HealthCheckController` 

`CronServices` 方法，检测一组服务。

```c#
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```

`ObscurePath` 显示从配置读取路径，用来测试接口。

```c#
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

`TestResponseFromConfig` 显示如何暴露一个检测接口

```c#
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```

