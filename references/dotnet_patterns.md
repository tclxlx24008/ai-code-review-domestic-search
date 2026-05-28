# .NET/C# 代码审查模式

.NET 和 C# 代码审查中常见的模式、反模式和陷阱。

## Async/Await 陷阱

### Async Void
```csharp
// 错误 - async void 隐藏异常、无法等待、可能导致进程崩溃
private async void HandleClick()
{
    await _service.DoWorkAsync();
}

// 正确 - async Task，异常可被捕获
private async Task HandleClickAsync()
{
    await _service.DoWorkAsync();
}
```
`async void` 唯一可接受的用途是 UI 框架中的事件处理器。

### 阻塞异步代码
```csharp
// 错误 - 在 ASP.NET（Core 之前）或 UI 上下文中可能死锁
var result = _service.GetDataAsync().Result;
var result = _service.GetDataAsync().GetAwaiter().GetResult();

// 正确 - 全链路异步
var result = await _service.GetDataAsync();
```

### CancellationToken 未传递
```csharp
// 错误 - CancellationToken 未转发
public async Task<Data> FetchAsync(CancellationToken ct)
{
    return await _http.GetAsync(url); // ct 未传递
}

// 正确 - 传播取消
public async Task<Data> FetchAsync(CancellationToken ct)
{
    return await _http.GetAsync(url, ct);
}
```

### ConfigureAwait
- 在库代码中：使用 `ConfigureAwait(false)` 避免捕获 SynchronizationContext
- 在 ASP.NET Core 中：无 SynchronizationContext，ConfigureAwait(false) 无效但无害
- 在 UI 代码中：必须使用 `ConfigureAwait(true)`（默认值）以返回 UI 线程

## 依赖注入反模式

### 服务定位器
```csharp
// 错误 - 服务定位器反模式
public class MyService
{
    public void DoWork()
    {
        var repo = _serviceProvider.GetService<IRepository>();
    }
}

// 正确 - 构造函数注入
public class MyService
{
    private readonly IRepository _repo;
    public MyService(IRepository repo) => _repo = repo;
}
```

### 生命周期俘获依赖
```csharp
// 错误 - 生命周期俘获（单例依赖了 Scoped 服务）
public class SingletonService
{
    private readonly ScopedService _scoped; // Scoped 被注入到 Singleton！
}

// 正确 - 依赖正确的生命周期，或谨慎使用 IServiceProvider
public class SingletonService
{
    private readonly IServiceScopeFactory _scopeFactory;
    public void DoWork()
    {
        using var scope = _scopeFactory.CreateScope();
        var scoped = scope.ServiceProvider.GetRequiredService<IScopedService>();
    }
}
```

### 缺少 DI 注册
- 使用 `[MapTo]` 特性约定的项目：验证新服务是否有该特性
- 使用程序集扫描时：检查各层的 `ServiceRegister.cs`
- 使用工厂模式选择服务时：验证 `IsSupport()` 实现

## 内存与资源管理

### IDisposable 模式
```csharp
// 错误 - 资源未释放
public byte[] ReadFile(string path)
{
    var stream = new FileStream(path, FileMode.Open);
    return ReadAllBytes(stream); // stream 从未释放
}

// 正确 - using 声明或语句
public byte[] ReadFile(string path)
{
    using var stream = new FileStream(path, FileMode.Open);
    return ReadAllBytes(stream);
}
```

### 大对象堆 (LOH)
- 大于 85KB 的对象会进入 LOH（数组、大字符串）
- 旧版 .NET 中 LOH 不会压缩 → 内存碎片化
- 大型临时数组使用 `ArrayPool<T>`
- 零分配切片考虑使用 `Memory<T>` / `Span<T>`

### 字符串分配
```csharp
// 错误 - 循环中字符串拼接
var result = "";
foreach (var item in items)
    result += item.ToString(); // 每次迭代分配新字符串

// 正确 - StringBuilder 或 string.Join
var result = new StringBuilder();
foreach (var item in items)
    result.Append(item);
// 或：var result = string.Join("", items);
```

## LINQ 反模式

### 多次枚举
```csharp
// 错误 - IEnumerable 被枚举两次
public void Process(IEnumerable<int> items)
{
    if (items.Any())          // 第一次枚举
        foreach (var item in items) { } // 第二次枚举（可能重新执行查询）
}

// 正确 - 必要时物化
public void Process(IEnumerable<int> items)
{
    var list = items.ToList();
    if (list.Any())
        foreach (var item in list) { }
}
```

### LINQ 中的 N+1 问题
```csharp
// 错误 - N+1 查询（循环中延迟加载）
var orders = _context.Orders.ToList();
foreach (var order in orders)
{
    var items = order.Items; // 每个 order 触发独立查询
}

// 正确 - 预加载
var orders = _context.Orders.Include(o => o.Items).ToList();
```

### Select 中含副作用
```csharp
// 错误 - LINQ 中的副作用
var results = items.Select(x =>
{
    _logger.Log($"Processing {x}"); // 副作用
    return Process(x);
}).ToList();

// 正确 - 分离关注点
foreach (var item in items)
    _logger.Log($"Processing {item}");
var results = items.Select(Process).ToList();
```

## ASP.NET Core 专项

### 中间件顺序
- 中间件按添加顺序执行
- 错误处理应放在前面
- 认证在授权之前
- CORS 在可能拒绝的其他中间件之前

### IHttpClientFactory
```csharp
// 错误 - 直接创建 HttpClient
public class MyService
{
    public async Task CallApi()
    {
        using var client = new HttpClient(); // 套接字耗尽风险
    }
}

// 正确 - 使用 IHttpClientFactory
public class MyService
{
    private readonly HttpClient _client;
    public MyService(HttpClient client) => _client = client; // 通过工厂注入
}
```

### Action Filter 与 Middleware 的选择
- 中间件用于横切关注点（日志、CORS、错误处理）
- Action Filter 用于 MVC 特定关注点（模型验证、授权）

## 异常处理模式

### 异常重抛
```csharp
// 错误 - catch 后重新抛出丢失堆栈跟踪
catch (Exception ex)
{
    _logger.Error(ex);
    throw ex; // 重置堆栈跟踪
}

// 正确 - 重抛保留堆栈跟踪
catch (Exception ex)
{
    _logger.Error(ex);
    throw; // 保留堆栈跟踪
}
```

### AggregateException 处理
```csharp
// 错误 - 吞掉内部异常
catch (AggregateException ae)
{
    _logger.Error(ae.Message); // 丢失内部异常
}

// 正确 - 正确处理或展平
catch (AggregateException ae)
{
    ae.Handle(ex =>
    {
        _logger.Error(ex);
        return true; // 标记为已处理
    });
}
```

## 线程安全模式

### 延迟初始化
```csharp
// 错误 - 非线程安全
private ExpensiveObject _cache;
public ExpensiveObject Cache => _cache ??= CreateExpensiveObject();

// 正确 - 线程安全延迟
private readonly Lazy<ExpensiveObject> _cache = new(CreateExpensiveObject);
public ExpensiveObject Cache => _cache.Value;

// 正确 - 可配置 LazyThreadSafetyMode
private readonly Lazy<ExpensiveObject> _cache = new(
    CreateExpensiveObject, LazyThreadSafetyMode.PublicationOnly);
```

### 并发集合
```csharp
// 错误 - 先检查后操作的竞态条件
if (!_dict.ContainsKey(key))
    _dict[key] = value; // 另一个线程可能已经添加

// 正确 - 使用并发集合
_dict.TryAdd(key, value);
```

## 配置模式

### Options 模式
```csharp
// 错误 - 直接读取配置
var value = _configuration["MySection:MyKey"];

// 正确 - 强类型选项
public class MyOptions
{
    public string MyKey { get; set; }
}
// 注册：services.Configure<MyOptions>(configuration.GetSection("MySection"));
// 注入：IOptions<MyOptions>、IOptionsMonitor<MyOptions>、IOptionsSnapshot<MyOptions>
```

### IOptions 与 IOptionsMonitor 与 IOptionsSnapshot
- `IOptions<T>` - 单例，只读一次，无变更检测
- `IOptionsMonitor<T>` - 单例，支持变更通知（ConfigCenter 重载）
- `IOptionsSnapshot<T>` - Scoped，每次请求重新读取

## .NET 常见安全问题

### SQL 注入
```csharp
// 错误
var sql = $"SELECT * FROM Users WHERE Name = '{name}'";

// 正确 - 参数化查询
var sql = "SELECT * FROM Users WHERE Name = @Name";
command.Parameters.AddWithValue("@Name", name);
```

### 不安全的反序列化
```csharp
// 错误 - BinaryFormatter 有安全风险
var obj = (MyType)new BinaryFormatter().Deserialize(stream);

// 正确 - 使用 System.Text.Json 或自定义转换器
var obj = JsonSerializer.Deserialize<MyType>(json);
```

### 加密算法
```csharp
// 错误 - 弱算法
using var md5 = MD5.Create();
using var sha1 = SHA1.Create();

// 正确 - 现代算法
using var sha256 = SHA256.Create();
// 加密使用 AES（Aes.Create()），而非 DES 或 RC2
// 密码哈希使用 Rfc2898DeriveBytes（PBKDF2），迭代次数应足够高
```
