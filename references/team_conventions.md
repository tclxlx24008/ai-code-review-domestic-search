# 团队代码开发规范

> 规范约束力由强到弱依次是：**【强制】**、**【推荐】**、**【参考】**。

| 标记 | 含义 |
|------|------|
| 【强制】 | 必须要遵守 |
| 【推荐】 | 建议遵守，特殊情况可豁免 |
| 【参考】 | 可选参考 |
| 【说明】 | 对规范的补充说明 |
| 【正例】 | 正确的使用实例 |
| 【反例】 | 错误的使用实例 |

---

## 一、命名

### 1. 【推荐】`private` 与 `internal` 字段用下划线开头的驼峰命名法，属性用首字母大写的命名法。

```csharp
// 错误 - private 字段未使用下划线前缀，属性未首字母大写
public class DataService
{
    private IWorkerQueue workerQueue;
    public string dataName { get; set; }
}

// 正确 - private 字段下划线驼峰，属性首字母大写
public class DataService
{
    public string DataName { get; set; }
    private IWorkerQueue _workerQueue;
}
```

### 2. 【强制】变量命名要名副其实、简洁明确，要避免歧义、难懂或误导性的命名。

```csharp
// 错误 - 变量名无意义、缩写难懂、误导
var d = DateTime.Now;                    // 含义不明
var ms = ToMemoryStream(content);        // 缩写不直观
var userList = GetUserCount();           // 命名与实际行为不符（List vs Count）

// 正确 - 变量名名副其实
var currentTime = DateTime.Now;
var memoryStream = ToMemoryStream(content);
var userCount = GetUserCount();
```

### 3. 【强制】尽量不要使用魔法数字，要优先考虑使用枚举、常量。对于魔法数字要有对应的注释进行解释说明。

```csharp
// 错误 - 魔法数字含义不明
if (status == 3)
{
    order.Discount = price * 0.85;
}

// 正确 - 使用枚举和常量
if (status == OrderStatus.Paid)
{
    order.Discount = price * VipDiscountRate;
}

private const double VipDiscountRate = 0.85;
```

### 4. 【推荐】变量命名优先考虑使用常见的英文单词，命名方式要优先考虑符合中国程序员的阅读习惯。

```csharp
// 错误 - 使用拼音或生僻缩写
var jiage = GetPrice();
var yhxz = GetUserRestrictions();

// 正确 - 使用常见英文单词
var price = GetPrice();
var userRestrictions = GetUserRestrictions();
```

### 5. 【推荐】如果变量存在的作用域比较广时，不要因为名字过长而简写。即：名称的长短应该与其作用域的大小相对应。

```csharp
// 错误 - 作用域广的变量过度简写
public class FlightSearchService
{
    private ISearchCacheService _sc;   // 类级别字段，作用域广，缩写难懂
}

// 正确 - 作用域广的变量使用完整命名
public class FlightSearchService
{
    private ISearchCacheService _searchCacheService;
}
```

```csharp
// 正确 - 作用域极小的循环变量可以用短名
foreach (var f in flights)  // 作用域仅限循环体，f 可接受
{
    Console.WriteLine(f.FlightNo);
}
```

### 6. 【强制】类与对象的命名应该是名词，方法命名应该是动词。

```csharp
// 错误 - 类名用动词，方法名用名词
public class Search { }
public void Flight() { }

// 正确 - 类名用名词，方法名用动词
public class FlightSearcher { }
public void SearchFlight() { }
```

### 7. 【强制】每个专业名词、概念都应该对应唯一的一个词，并坚持使用这个词，同时也要坚持一个词对应唯一一个专有词。即：**一词一义，一义一词**，词与义是一对一的关系。

> 【说明】参考术语对照表。

```csharp
// 错误 - 同一概念使用不同词汇，造成混淆
public class UserService
{
    public User GetUserById(int id) { }
}

public class OrderService
{
    public User FindUserById(int id) { }  // 同一概念用了 Find 而非 Get
}

// 正确 - 同一概念统一使用 Get
public class UserService
{
    public User GetUserById(int id) { }
}

public class OrderService
{
    public User GetUserById(int id) { }
}
```

```csharp
// 错误 - 同一词汇表达不同含义
public void Apply(string code) { }   // Apply = 使用优惠码
public void Apply(int ruleId) { }    // Apply = 应用规则

// 正确 - 不同含义使用不同词汇
public void RedeemCoupon(string code) { }   // 使用优惠码
public void ApplyRule(int ruleId) { }        // 应用规则
```

### 8. 【强制】命名优先考虑计算机专业领域的词汇，其次是业务领域。

> 【说明】阅读你代码的人是程序员，尽量要使用专业领域的词汇。如果你使用了设计模式，比如工厂模式，那么使用了工厂设计模式的接口与类都要带有 `Factory` 后缀，比如 `XXXFactory`。

```csharp
// 错误 - 忽略设计模式命名约定，使用业务术语
public interface IMakeSearchService { }
public class GetSearchStuff { }

// 正确 - 使用计算机专业术语，体现设计模式
public interface ISearchServiceFactory { }
public class SearchServiceFactory : ISearchServiceFactory { }
```

```csharp
// 错误 - 使用业务行话代替专业术语
public class TicketBucket { }        // 业务行话：票桶

// 正确 - 使用计算机专业术语
public class TicketPool { }          // 专业术语：对象池/资源池
```

---

## 二、函数

### 1. 【推荐】一个函数的代码应该保持短小。

```csharp
// 错误 - 函数超过 80 行，做了太多事情
public async Task<SearchResult> SearchAsync(Request request)
{
    // 参数校验 10 行
    // 缓存查询 15 行
    // 数据库查询 20 行
    // 结果过滤 15 行
    // 日志记录 10 行
    // 结果组装 15 行
    // ...总计 85 行
}

// 正确 - 拆分为多个短小函数，每个函数职责单一
public async Task<SearchResult> SearchAsync(Request request)
{
    ValidateRequest(request);
    var cached = await TryGetCacheAsync(request);
    if (cached != null) return cached;

    var raw = await QueryDatabaseAsync(request);
    var filtered = ApplyFilters(raw, request);
    LogSearchMetrics(request, filtered);
    return BuildResult(filtered);
}
```

### 2. 【强制】一个函数应该只做一件事，并且要做好这件事。

> 【说明】这个原则跟第一个是相似的，如果一个函数只是专注做一件事的话，那么这个函数的代码就不会太长。

```csharp
// 错误 - 函数既查询用户又发送通知
public User GetUserAndNotify(int userId)
{
    var user = _userRepository.GetById(userId);
    _notificationService.SendWelcome(user.Email);
    return user;
}

// 正确 - 查询和通知分离
public User GetUser(int userId)
{
    return _userRepository.GetById(userId);
}

public void SendWelcomeNotification(string email)
{
    _notificationService.SendWelcome(email);
}
```

### 3. 【强制】函数命名要能正确表达该函数要做的事情。如果遇到函数命名难以表达的场景，需要添加注释辅以解释说明。

```csharp
// 错误 - 函数名无法表达实际行为
public List<Flight> Process(FlightRequest req) { }

// 正确 - 函数名正确表达意图
public List<Flight> SearchAvailableFlights(FlightRequest request) { }
```

```csharp
// 函数名难以完全表达时，补充注释说明
/// <summary>
/// 根据双价格策略筛选替补产品
/// </summary>
/// <remarks>
/// 当主产品被双价格策略移除时，从同舱等中选取最低价替补
/// </remarks>
public void ApplyDualPriceSubstitution(PlatDo plat, OnewayTripDo trip) { }
```

### 4. 【强制】一个函数内的语句都应该在同一个抽象层级。

```csharp
// 错误 - "迁移强制抢票的逻辑"与其他代码不是同一个抽象层级
public async Task<OnewaySearchResponseMessage> SearchCabinsAsync(
    PlatModel plat,
    OnewayRequestMessage oneway,
    CancellationToken cancellationToken)
{
    if (oneway.IsForceGrab && (flight.cabins?.Count > 0)) // 迁移强制抢票的逻辑
    {
        var groupCabins = flight.cabins
            .OrderBy(c => c.SalePrice)
            .GroupBy(o => o.realRoomCode);
        flight.cabins = new List<CabinPriceMessage>();
        foreach (var item in groupCabins)
        {
            if (Constants.GrabCabins.Contains(item.Key))
            {
                flight.cabins.Add(item.FirstOrDefault());
            }
        }
    }

    await _buddhaTagFilter.IndexCountAsync(oneway.User, flight, cancellationToken);

    return response;
}

// 正确 - 低层细节抽离为独立函数，保持同一抽象层级
public async Task<OnewaySearchResponseMessage> SearchCabinsAsync(
    PlatModel plat,
    OnewayRequestMessage oneway,
    CancellationToken cancellationToken)
{
    // 强制抢票场景下筛选指定舱等
    if (oneway.IsForceGrab && flight.cabins?.Count > 0)
    {
        FilterGrabCabins(flight);
    }

    await _buddhaTagFilter.IndexCountAsync(oneway.User, flight, cancellationToken);

    return response;
}

/// <summary>
/// 按舱等分组，仅保留抢票舱等的最低价
/// </summary>
private void FilterGrabCabins(FlightMessage flight)
{
    var groupCabins = flight.cabins
        .OrderBy(c => c.SalePrice)
        .GroupBy(o => o.realRoomCode);
    flight.cabins = groupCabins
        .Where(g => Constants.GrabCabins.Contains(g.Key))
        .Select(g => g.First())
        .ToList();
}
```

### 5. 【推荐】异步函数的命名要带有 `Async` 后缀。

```csharp
// 错误 - 异步函数缺少 Async 后缀
public async Task<SearchResult> Search(SearchRequest request) { }

// 正确 - 异步函数带 Async 后缀
public async Task<SearchResult> SearchAsync(SearchRequest request) { }
```

### 6. 【强制】函数的参数不宜过多，建议四个以上的参数用结构体或者类进行封装，即：参数对象。构造函数除外。

```csharp
// 错误 - 参数过多，难以理解和维护
public async Task<SearchResult> SearchAsync(
    string departure,
    string arrival,
    DateTime date,
    string airline,
    string cabinClass,
    int adultCount,
    int childCount)
{
}

// 正确 - 使用参数对象封装
public async Task<SearchResult> SearchAsync(SearchCriteria criteria)
{
}

public class SearchCriteria
{
    public string Departure { get; set; }
    public string Arrival { get; set; }
    public DateTime Date { get; set; }
    public string Airline { get; set; }
    public string CabinClass { get; set; }
    public int AdultCount { get; set; }
    public int ChildCount { get; set; }
}
```

### 7. 【推荐】指令操作类语句与查询操作类语句不应该放到同一个函数中。

```csharp
// 错误 - 查询函数中执行了修改操作
public bool IsExist(int userId)
{
    var user = _repository.FindById(userId);
    user.Name = "test";  // 查询函数中不应修改数据
    return user != null;
}

// 正确 - 查询与修改分离
public bool IsExist(int userId)
{
    return _repository.FindById(userId) != null;
}

public void UpdateUserName(int userId, string name)
{
    var user = _repository.FindById(userId);
    user.Name = name;
    _repository.Save(user);
}
```

### 8. 【强制】不要重复代码，即：**DRY 原则**。

```csharp
// 错误 - 相同的过滤逻辑在多处重复
public List<Flight> GetDomesticFlights(SearchRequest request)
{
    var flights = _flightDao.GetByRoute(request.Departure, request.Arrival);
    var filtered = flights.Where(f => f.Status == FlightStatus.Active && f.Price > 0).ToList();
    return filtered;
}

public List<Flight> GetInternationalFlights(SearchRequest request)
{
    var flights = _flightDao.GetByRoute(request.Departure, request.Arrival);
    var filtered = flights.Where(f => f.Status == FlightStatus.Active && f.Price > 0).ToList();
    return filtered;
}

// 正确 - 提取公共过滤逻辑
public List<Flight> GetDomesticFlights(SearchRequest request)
{
    var flights = _flightDao.GetByRoute(request.Departure, request.Arrival);
    return FilterActiveFlights(flights);
}

public List<Flight> GetInternationalFlights(SearchRequest request)
{
    var flights = _flightDao.GetByRoute(request.Departure, request.Arrival);
    return FilterActiveFlights(flights);
}

private List<Flight> FilterActiveFlights(IEnumerable<Flight> flights)
{
    return flights.Where(f => f.Status == FlightStatus.Active && f.Price > 0).ToList();
}
```

### 9. 【推荐】`public` 级别函数参数不要接受 `null` 值，而且还要考虑在函数体的开头部分对参数进行判 `null`。

```csharp
// 错误 - 未校验参数，NullReferenceException 延后爆发
public async Task<SearchResult> SearchAsync(SearchRequest request)
{
    var flights = _service.Search(request.Departure);  // request 为 null 时此处才报错
    return BuildResult(flights);
}

// 正确 - 入口处判 null，快速失败并给出明确信息
public async Task<SearchResult> SearchAsync(SearchRequest request)
{
    ArgumentNullException.ThrowIfNull(request);
    var flights = _service.Search(request.Departure);
    return BuildResult(flights);
}
```

### 10. 【推荐】及时 Clean Code，及时删除没有被引用的临时变量、字段、函数。

```csharp
// 错误 - 残留未使用的变量和字段
public class FlightService
{
    private readonly ILogger _oldLogger;       // 已废弃，不再使用
    private readonly ISearchService _searchService;

    public async Task<Flight> SearchAsync(string id)
    {
        var tempResult = await _searchService.GetAsync(id);  // tempResult 未被使用
        return await _searchService.GetAsync(id);
    }
}

// 正确 - 清理未使用的变量和字段
public class FlightService
{
    private readonly ISearchService _searchService;

    public async Task<Flight> SearchAsync(string id)
    {
        return await _searchService.GetAsync(id);
    }
}
```

### 11. 【强制】函数中的 `if` 语句判断条件不允许有两层及以上的嵌套层级，如果有多个判断条件需要写到单独的函数中。

```csharp
// 错误 - 条件嵌套过深，难以理解
var ticket = availables.Where(it =>
    (!it.PolicyTypes.Any() || it.PolicyTypes.Contains(cabin.pt.ToString()))
    && it.MinTicketPrice <= cabin.SalePrice
    && (string.IsNullOrEmpty(it.GoodType)
        || !it.GoodType.Equals("AIRLINEVOUCHER", StringComparison.CurrentCultureIgnoreCase)
        || (cabin.OfficialDirectAmount <= 0 && offreduceamount <= 0))
    && (it.PhoenixRuleIds == null || !it.PhoenixRuleIds.Any()
        || it.PhoenixRuleIds.Contains(cabin.ruleId))
);

// 正确 - 复杂条件抽离为命名明确的函数
var ticket = availables.Where(it => IsTicketMatchPolicy(it, cabin, offreduceamount));

/// <summary>
/// 判断票务是否符合策略条件
/// </summary>
private bool IsTicketMatchPolicy(AvailableTicket ticket, Cabin cabin, decimal offReduceAmount)
{
    if (!IsPolicyTypeMatch(ticket, cabin)) return false;
    if (ticket.MinTicketPrice > cabin.SalePrice) return false;
    if (!IsGoodTypeMatch(ticket, cabin, offReduceAmount)) return false;
    if (!IsPhoenixRuleMatch(ticket, cabin)) return false;
    return true;
}
```

### 12. 【推荐】异步函数要考虑支持 `CancellationToken` 参数。

```csharp
// 错误 - 长时间异步操作不支持取消，浪费资源
public async Task<List<Flight>> SearchFlightsAsync(SearchCriteria criteria)
{
    var result = await _remoteService.QueryAsync(criteria);
    return result;
}

// 正确 - 支持 CancellationToken，允许调用方取消操作
public async Task<List<Flight>> SearchFlightsAsync(SearchCriteria criteria, CancellationToken cancellationToken)
{
    var result = await _remoteService.QueryAsync(criteria, cancellationToken);
    return result;
}
```

### 13. 【参考】`if` 语句要考虑是否需要处理 `else` 的场景。

```csharp
// 错误 - 遗漏了 else 分支，导致静默丢失数据
public void ApplyDiscount(Order order, Voucher voucher)
{
    if (voucher != null)
    {
        order.Discount = voucher.Amount;
    }
    // voucher 为 null 时没有任何处理，调用方不知道折扣未生效
}

// 正确 - 明确处理 else 场景
public void ApplyDiscount(Order order, Voucher voucher)
{
    if (voucher != null)
    {
        order.Discount = voucher.Amount;
    }
    else
    {
        _logger.Warn($"Order {order.Id} has no valid voucher, discount not applied");
        order.Discount = 0;
    }
}
```

### 14. 【强制】禁止使用 LINQ 的 `ForEach`。

```csharp
// 错误 - 使用 LINQ ForEach，难以调试和异常处理
flights.Where(f => f.IsAvailable)
    .ToList()
    .ForEach(f => f.Status = FlightStatus.Sold);

// 正确 - 使用标准 foreach 循环，清晰、可调试
foreach (var flight in flights.Where(f => f.IsAvailable))
{
    flight.Status = FlightStatus.Sold;
}
```

---

## 三、注释

### 1. 【强制】注释要说明代码的意图，不要写无用的注释。

```csharp
// 错误 - 注释只是重复代码，没有说明意图
// 给 i 加 1
i++;

// 检查 count 是否大于 0
if (count > 0) { }

// 正确 - 注释说明意图和原因
// 重试计数器递增，为下一次请求准备
i++;

// 仅在有可用数据时才执行批量导入
if (count > 0) { }
```

```csharp
// 正例 - 完整的 XML 文档注释说明意图
/// <summary>
/// 设置双价格替补资源
/// </summary>
/// <param name="plat">渠道</param>
/// <param name="user">用户</param>
/// <param name="onewayTripDo">行程</param>
/// <remarks>
/// <para>一、根据双价格开启状态进行分组，双价格产品一组、其他产品一组</para>
/// <para>二、在其他产品分组内进行筛选可用于双价格替补的产品，按照每个舱等取一个最低</para>
/// <para>可作为替补产品的要求：</para>
/// <para>1、非打包、非抢票、非精准营销、非航司旗舰店、非打包立减、非打包返券、非精选特价、非超值特价、非小众</para>
/// <para>2、无限制类标签或者有且只有一个限制类标签并且是供应商延迟出票</para>
/// <para>3、如果是官网双价格，替补资源的邮寄类型必须是行程单</para>
/// <para>4、替补资源的价格最低为客户实际支付价即用销售价减直减金额后的价格参与替补筛选</para>
/// <para>三、设置替补资源或者移除双价格资源</para>
/// <para>根据双价格资源的舱等获取对应的替补资源，如果没有获取到对应的替补资源需要将当前双价格资源移除</para>
/// <para>特殊流程：如果如来返回用户是企业双价格用户或者航司双城卡也需要进行替补资源设置</para>
/// <para>设置双价格的产品标签(btpt)、在替补资源上标记适用双价格资源的产品码(aobtpt)</para>
/// <para>如果双价格资源为企业双价格产品、需要在当前双价格产品上设置企业代码和企业名称</para>
/// <para>在行程上设置双价格产品码对应的book1标签，此逻辑需要移至行程属性上统一设置</para>
/// </remarks>
public void SetAlternateProduct(PlatDo plat, UserDo user, OnewayTripDo onewayTripDo)
```

### 2. 【强制】如果参数、返回值的含义比较晦涩难懂应该考虑用注释来说明。

```csharp
// 错误 - 参数和返回值含义不明确
public Task<byte[]> ProcessAsync(Memory<byte> data, bool flag)
{
}

// 正确 - 用注释说明参数和返回值的含义
/// <summary>
/// 压缩并处理原始数据
/// </summary>
/// <param name="data">GZip 压缩的原始 Protobuf 数据</param>
/// <param name="flag">是否启用缓存旁路模式（true 表示跳过缓存直接查询）</param>
/// <returns>GZip 压缩后的处理结果字节，可直接写入缓存</returns>
public Task<byte[]> ProcessAsync(Memory<byte> data, bool flag)
{
}
```

### 3. 【强制】代码与注释要同步更新，要避免无用的、过时的以及具有误导性的注释。

```csharp
// 错误 - 注释与代码不一致，具有误导性
#region ==============
/// <summary>
/// *********郑重提示*********
/// 仅用于火车票推荐时候区分微信和小程序渠道，其他地方请勿使用该字段，否则后果自负
/// 小程序 852 微信h5 501
/// </summary>
public int Plat { get; set; }
#endregion

// 正确 - 注释与代码保持一致
/// <summary>
/// 渠道标识，用于区分不同查询来源
/// </summary>
/// <remarks>
/// 小程序渠道值为 852，微信 H5 渠道值为 501
/// </remarks>
public int Plat { get; set; }
```

### 4. 【推荐】如果代码有潜在的风险并且没有办法通过命名方式来表达的时候，要用注释来警示同事。

```csharp
// 错误 - 零拷贝风险未警示，后续维护者可能误用
private static MemoryStream ToMemoryStream(Memory<byte> memory)
{
    if (MemoryMarshal.TryGetArray(memory, out var segment))
    {
        return new MemoryStream(segment.Array, segment.Offset, segment.Count, writable: false);
    }
    return new MemoryStream(memory.ToArray());
}

// 正确 - 用注释警示潜在风险
/// <summary>
/// 将 Memory&lt;byte&gt; 包装为只读 MemoryStream，优先零拷贝获取底层数组段
/// </summary>
/// <remarks>
/// ⚠️ 零拷贝风险：返回的 MemoryStream 与源 memory 共享底层数组。
/// 调用方必须确保 memory 在 Stream 被 ParseFrom 完整消费之前保持有效且不被修改，
/// 否则可能导致数据损坏。若无法保证生命周期，应改用 ToArray() 拷贝方式。
/// </remarks>
private static MemoryStream ToMemoryStream(Memory<byte> memory)
```

### 5. 【推荐】如果一段代码没有用了，应该直接删除不应该注释该代码。

```csharp
// 错误 - 注释掉废弃代码，增加阅读负担
public async Task<SearchResult> SearchAsync(SearchRequest request)
{
    // var oldResult = _legacyService.Search(request.Departure, request.Arrival);
    // if (oldResult.HasData)
    // {
    //     return TransformResult(oldResult);
    // }
    return await _newService.SearchAsync(request);
}

// 正确 - 直接删除废弃代码，版本控制已保存历史
public async Task<SearchResult> SearchAsync(SearchRequest request)
{
    return await _newService.SearchAsync(request);
}
```

### 6. 【强制】代码中的多条件 `if` 语句需要有注释进行解释说明。

```csharp
// 错误 - 多条件if 条件意图不明
if (response.StatusCode == 302 && !request.IsRedirectAllowed)
{
    throw new BusinessException("Redirect not allowed");
}

// 正确 - 多条件if 语句有注释说明判断意图
// 302 重定向且调用方未授权重定向时，视为异常
if (response.StatusCode == 302 && !request.IsRedirectAllowed)
{
    throw new BusinessException("Redirect not allowed");
}
```

### 7. 【强制】公开的函数、枚举、公开的常量等要有注释说明。

```csharp
// 错误 - 公开成员缺少注释
public class SearchOptions
{
    public static readonly int MaxRetries = 3;

    public enum CacheMode
    {
        None,
        ReadOnly,
        ReadWrite
    }

    public async Task<SearchResult> SearchAsync(SearchRequest request) { }
}

// 正确 - 公开成员都有注释说明
public class SearchOptions
{
    /// <summary>
    /// 查询最大重试次数
    /// </summary>
    public static readonly int MaxRetries = 3;

    /// <summary>
    /// 缓存使用模式
    /// </summary>
    public enum CacheMode
    {
        /// <summary>
        /// 不使用缓存
        /// </summary>
        None,

        /// <summary>
        /// 仅读取缓存
        /// </summary>
        ReadOnly,

        /// <summary>
        /// 读写缓存
        /// </summary>
        ReadWrite
    }

    /// <summary>
    /// 执行航班查询
    /// </summary>
    /// <param name="request">查询请求参数</param>
    /// <returns>查询结果</returns>
    public async Task<SearchResult> SearchAsync(SearchRequest request) { }
}
```

---

## 四、接口与类

### 1. 【强制】接口与类都应该保持短小，单一职责。

```csharp
// 错误 - 类承担了搜索、缓存、通知多种职责
public class FlightService
{
    public List<Flight> Search(SearchRequest request) { }
    public void SaveToCache(string key, List<Flight> flights) { }
    public List<Flight> GetFromCache(string key) { }
    public void NotifyUser(int userId, string message) { }
    public void SendEmail(string to, string subject) { }
}

// 正确 - 拆分为单一职责的类
public class FlightSearchService
{
    public List<Flight> Search(SearchRequest request) { }
}

public class FlightCacheService
{
    public void Save(string key, List<Flight> flights) { }
    public List<Flight> Get(string key) { }
}

public class NotificationService
{
    public void NotifyUser(int userId, string message) { }
    public void SendEmail(string to, string subject) { }
}
```

### 2. 【推荐】保持内聚。

```csharp
// 错误 - 低内聚：不相关的方法放在同一个类
public class UserHelper
{
    public bool ValidateEmail(string email) { }
    public decimal CalculateFlightTax(Flight flight) { }
    public string FormatPhoneNumber(string phone) { }
}

// 正确 - 高内聚：相关方法放在同一个类
public class EmailValidator
{
    public bool Validate(string email) { }
    public bool ValidateFormat(string email) { }
    public bool ValidateDomain(string email) { }
}
```

### 3. 【推荐】隔离修改。

```csharp
// 错误 - 直接依赖具体实现，修改时影响范围大
public class FlightService
{
    private readonly RedisCacheProvider _cache = new RedisCacheProvider();

    public List<Flight> Search(SearchRequest request)
    {
        var cached = _cache.Get<List<Flight>>(request.Key);
        if (cached != null) return cached;
        // ...
    }
}

// 正确 - 依赖抽象，修改缓存实现不影响业务代码
public class FlightService
{
    private readonly ICacheService _cacheService;

    public FlightService(ICacheService cacheService)
    {
        _cacheService = cacheService;
    }

    public List<Flight> Search(SearchRequest request)
    {
        var cached = _cacheService.GetAsync<List<Flight>>(request.Key);
        if (cached != null) return cached;
        // ...
    }
}
```

---

## 五、性能

## 六、网关站与产品站entrance枚举

### entrance 枚举

1. 【强制】定义 `entrance` 枚举时要维护 entrance 定义文档：
   https://toca.17u.cn/wiki?fid=3b63a9e9d9d648caae14612626cd7c32

## 七、函数调用

### 1. 【强制】被调用函数存在 `out` 入参时，要用临时声明的变量去传参。

```csharp
// 错误 - out 参数直接赋值给业务变量，TryGetValue 失败时 productPriority 被重置为 0
int productPriority = -1;

if (_dosAccessor.TryGetValue((int)tripType, out var products))
{
    products.TryGetValue(productCode, out productPriority);  // 失败时 productPriority 变为 0
}

// 正确 - 用临时变量传 out 参数，成功后再赋值
int productPriority = -1;

if (_dosAccessor.TryGetValue((int)tripType, out var products))
{
    if (products.TryGetValue(productCode, out var productPriorityConfig))
    {
        productPriority = productPriorityConfig;
    }
}
```

> 用临时声明的变量传值 `out` 入参（比如上面代码用 `productPriorityConfig`），`TryGetValue` 函数返回成功后再对需要使用的变量进行赋值。

---

## 附录：强制规范速查表

审查时重点关注以下【强制】规则，违反即为 HIGH 及以上发现：

| 类别 | 编号 | 规则摘要 |
|------|------|----------|
| 命名 | 2 | 变量命名名副其实、简洁明确 |
| 命名 | 3 | 避免魔法数字，优先枚举/常量 |
| 命名 | 6 | 类/对象命名用名词，方法命名用动词 |
| 命名 | 7 | 一词一义，一义一词 |
| 命名 | 8 | 命名优先计算机专业词汇，其次业务领域 |
| 函数 | 2 | 一个函数只做一件事 |
| 函数 | 3 | 函数命名正确表达意图 |
| 函数 | 4 | 函数内语句同一抽象层级 |
| 函数 | 6 | 参数不超过4个（构造函数除外） |
| 函数 | 8 | DRY 原则，不重复代码 |
| 函数 | 11 | if 条件不超过两层嵌套 |
| 函数 | 14 | 禁止 LINQ ForEach |
| 注释 | 1 | 注释说明代码意图 |
| 注释 | 3 | 代码与注释同步更新 |
| 注释 | 6 | if 语句需要注释说明 |
| 注释 | 7 | 公开函数/枚举/常量要有注释 |
| 接口类 | 1 | 接口与类保持短小，单一职责 |
| 性能 | 1 | out 入参用临时变量传参 |
