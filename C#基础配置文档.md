

## 系统中优先使用异步方法

## Startup默认格式

```c#
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    // 该方法由运行时调用，使用该方法向DI容器添加服务
    public void ConfigureServices(IServiceCollection services)
    {
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    // 该方法由运行时调用，使用该方法配置HTTP请求管道
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env,IHostApplicationLifetime appLifetime)
    {
    }
}

```

```C#
using BMS_Base.Config;
using Consul;

namespace BMS;

public class Startup
{

    private readonly IConfiguration _configuration;

    public Startup(IConfiguration configuration)
    {
        this._configuration = configuration;
    }
    // This method gets called by the runtime. Use this method to add services to the container.
    // 该方法由运行时调用，使用该方法向DI容器添加服务
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
        services.AddControllers();
        services.AddEndpointsApiExplorer();
        services.AddSwaggerGen();
        services.AddCors(options =>
        {
            options.AddPolicy("AllowAllOrigin", builder =>
            {
                builder
                    .SetIsOriginAllowed(_ => true)
                    .AllowAnyMethod()
                    .AllowAnyHeader()
                    .AllowCredentials();
            });
        });

        services.Configure<ConsulConfig>(_configuration.GetSection("ConsulConfig"));
        _configuration.Bind("ConsulConfig", ConsulConfig.Instance);
        Console.WriteLine( ConsulConfig.Instance);
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    // 该方法由运行时调用，使用该方法配置HTTP请求管道
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IHostApplicationLifetime appLifetime)
    {
        if (env.IsDevelopment())
        {
            app.UseSwagger();
            app.UseSwaggerUI();
        }

        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(x =>
        {
            x.MapControllers();
            x.MapGet("/", async context =>
            {
                await context.Response.WriteAsync("Hello World!");
            });
        });
        app.UseCors("AllowAllOrigin");
        RegisterConsul(appLifetime);
    }

    private void RegisterConsul(IHostApplicationLifetime appLifetime)
    {
        using var client = new ConsulClient(x => x.Address = new Uri("http://127.0.0.1:8500"));
        var check = new AgentServiceCheck()
        {
            DeregisterCriticalServiceAfter = TimeSpan.FromSeconds(5),//服务停止后，5s开始接触注册
            HTTP = ConsulConfig.Instance.CheckApi,//健康检查
            Interval = TimeSpan.FromSeconds(10),//每10s轮询一次健康检查
            Timeout = TimeSpan.FromSeconds(5),
        };
        var service = new AgentServiceRegistration()
        {
            Checks = new[] { check },
            ID = Guid.NewGuid().ToString(),
            Name = ConsulConfig.Instance.ServiceName,
            Port = ConsulConfig.Instance.Port,
            Address = ConsulConfig.Instance.Address
        };
        client.Agent.ServiceRegister(service).Wait();
        appLifetime.ApplicationStopped.Register(() =>
        {
            Console.WriteLine("服务停止中");
            using var consulClient = new ConsulClient(x => x.Address = new Uri("http://127.0.0.1:8500"));
            consulClient.Agent.ServiceDeregister(service.ID).Wait();
        });
    }
}
```





```C#
CreateHostBuilder(args).Build().Run();
static IHostBuilder CreateHostBuilder(string[] args) =>
   Host.CreateDefaultBuilder(args)
       .ConfigureWebHostDefaults(webBuilder =>
       {
           webBuilder.UseStartup<Startup>();
       });
```

## Ocelot

```json
{
    "ReRoutes": [],
    "GlobalConfiguration": {}
}
```

```josn
{
    "Routes": [
        {
            "DownstreamPathTemplate": "/api/{url}",
            "DownstreamScheme": "http",
            "DownStreamHostAndPorts": [
                {
                    "Host": "localhost",
                    "Port": "5086"
                }
            ],
            "UpstreamPathTemplate": "/ServiceA/{url}",
            "UpstreamHttpMethod": [ "Get", "Post" ],
            "ServiceName": "ServiceA",
            "UseServiceDiscovery": true
            //"LoadBalanceOptions": {
            //    "Type": "RoundRobin"
            //}
        },
        {
            "DownstreamPathTemplate": "/api/{url}",
            "DownstreamScheme": "http",
            "DownStreamHostAndPorts": [
                {
                    "Host": "localhost",
                    "Port": "5087"
                }
            ],
            "UpstreamPathTemplate": "/ServiceB/{url}",
            "UpstreamHttpMethod": [ "Get", "Post" ],
            "ServiceName": "ServiceB",
            "UseServiceDiscovery": true
            //"LoadBalanceOptions": {
            //    "Type": "RoundRobin"
            //}
        }
    ],
    "GlobalConfiguration": {
        "BaseUrl": "http://localhost:5131",
        "ServiceDiscoveryProvider": {
            "Host": "localhost",
            "Port": 8500,
            "Type": "Consul"
        }
    }
}

```

**LoadBalanceOptions**

LeastConnection - 最少连接，跟踪哪些服务正在处理请求，并把新请求发送到现有请求最少的服务上。该算法状态不在整个Ocelot集群中分布。

RoundRobin - 轮询可用的服务并发送请求。 该算法状态不在整个Ocelot集群中分布。

NoLoadBalancer - 不负载均衡，从配置或服务发现提供程序中取第一个可用的下游服务。

CookieStickySessions - 使用cookie关联所有相关的请求到制定的服务。下面有更多信息。

```josn
 {
            "DownstreamPathTemplate": "/Light/api/{all}",
            "UpstreamPathTemplate": "/api/manhui_wms_light/{all}",
            "UpstreamHttpMethod": ["Get", "Post"],
            "DownstreamHttpMethod": null,
            "AddHeadersToRequest": {},
            "UpstreamHeaderTransform": {},
            "DownstreamHeaderTransform": {},
            "AddClaimsToRequest": {},
            "RouteClaimsRequirement": {},
            "AddQueriesToRequest": {},
            "ChangeDownstreamPathTemplate": {},
            "RequestIdKey": null,
            "FileCacheOptions": { "TtlSeconds": 0, "Region": null },
            "RouteIsCaseSensitive": false,
            "ServiceName": "manhui_wms_light",
            "ServiceNamespace": null,
            "DownstreamScheme": "http",
            "QoSOptions": { "ExceptionsAllowedBeforeBreaking": 0, "DurationOfBreak": 0, "TimeoutValue": 0 },
            "LoadBalancerOptions": { "Type": "LeastConnection", "Key": null, "Expiry": 0 },
            "RateLimitOptions": { "ClientWhitelist": [], "EnableRateLimiting": false, "Period": null, "PeriodTimespan": 0.0, "Limit": 0 },
            "AuthenticationOptions": { "AuthenticationProviderKey": null, "AllowedScopes": [] },
            "HttpHandlerOptions": { "AllowAutoRedirect": false, "UseCookieContainer": false, "UseTracing": false, "UseProxy": true, "MaxConnectionsPerServer": 2147483647 },
            "DownstreamHostAndPorts": [],
            "UpstreamHost": null,
            "Key": null,
            "DelegatingHandlers": [],
            "Priority": 1,
            "Timeout": 0,
            "DangerousAcceptAnyServerCertificateValidator": false,
            "SecurityOptions": { "IPAllowedList": [], "IPBlockedList": [] },
            "DownstreamHttpVersion": null
        },
```

**可使用的网关配置**

```C#
{
    "Routes": [
        {
            "DownstreamPathTemplate": "/api/{url}",
            "DownstreamScheme": "http",
            "UpstreamPathTemplate": "/ServiceA/{url}",
            "UpstreamHttpMethod": [ "Get", "Post" ],
            "ServiceName": "ServiceA",
            "UseServiceDiscovery": true,
            "LoadBalancerOptions": {
                "Type": "LeastConnection"
            }
        },
        {
            "DownstreamPathTemplate": "/api/{url}",
            "DownstreamScheme": "http",
            "UpstreamPathTemplate": "/ServiceB/{url}",
            "UpstreamHttpMethod": [ "Get", "Post" ],
            "ServiceName": "ServiceB",
            "UseServiceDiscovery": true,
            "LoadBalancerOptions": {
                "Type": "LeastConnection"
            }
        }
    ],
    "GlobalConfiguration": {
        "ServiceDiscoveryProvider": {
            "Host": "localhost",
            "Port": 8500,
            "Type": "Consul"
        }
    }
}
```



## EFContext基础模板

```C#
public class BmsV1DbContext : DbContext
{
    public BmsV1DbContext(DbContextOptions<BmsV1DbContext> options) : base(options) { }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        modelBuilder.ApplyConfigurationsFromAssembly(this.GetType().Assembly);
    }
}
```

需要使用 Microsoft.EntityFrameworkCore包





## 获取程序集的方法

```html
AppDomain.CurrentDomain.GetAssemblies()
这个方法获取的是当前应用程序域已经加载的程序集，未加载的是获取不到的(尽管引用了该项目),所以在配置依赖注入时，可能会出现有些程序集拿不到的情况，导致没有注入所有需要的服务。

Assembly.GetEntryAssembly().GetReferencedAssemblies().Select(Assembly.Load)
获取该应用所有引用的程序集

DependencyContext.Default.CompileLibraries
获取CLR中的所有库(我也是一知半解)
```







## 数据库链接字符串

### PostgreSQL

```SQl
Host=118.178.241.151;Username=postgres;Database=version;Port=54321
```

### SQL Server

```SQl
server=192.168.31.126;database=EFLearn;user id=sa;password=123456;TrustServerCertificate=true
```

sql server 新建Select锁

```sql
select * from T_House  WITH (HOLDLOCK) 
```

### MySQL

```sql
server=192.168.31.126;database=webapp;user id=root;password=123456;
```

## jwt基础配置

> 需要包Microsoft.AspNetCore.Authentication.JwtBearer Microsoft.AspNetCore.Authorization

```C#
// 注入Jwt
services.AddAuthentication(option =>
{
    //认证middleware配置
    option.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    option.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(config =>
{
    config.TokenValidationParameters = new TokenValidationParameters
    {
        //Token颁发机构
        ValidIssuer = TokenConfig.Instance.IsUser,
        //颁发给谁
        ValidAudience = TokenConfig.Instance.Audience,
        //这里的key要进行加密
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(TokenConfig.Instance.Key)),
        //是否验证Token有效期，使用当前时间与Token的Claims中的NotBefore和Expires对比
        ValidateLifetime = true,
        ClockSkew = TimeSpan.Zero,
    };
    config.SaveToken = true;
});
```

```C#
//UseAuthentication必须在UseAuthorization的上面
app.UseAuthentication();
app.UseAuthorization();
```

```C#
public virtual CurrentUser? CurrentUser
{
    get
    {
        try
        {
            var claims = HttpContext.User.Claims.ToList();
            var user = new CurrentUser
            {
                Id = claims.First(p => p.Type == ClaimTypes.NameIdentifier).Value,
                Name = claims.First(p => p.Type == ClaimTypes.Name).Value,
                Code = claims.First(p => p.Type == "UserCode").Value,
                JwtVersion = claims.First(p => p.Type == ClaimTypes.Version).Value,
                ExpireTime = claims.First(p => p.Type == "ExpireTime").Value,
                Claims = claims,
            };
            return user;
        }
        catch (System.Exception e)
        {
            Console.WriteLine(e.ToString());
            return null;
        }
    }
}
```

```C#
public record CurrentUser
{
    /// <summary>
    /// 用户ID
    /// </summary>
    public string Id { get; set; } = String.Empty;

    /// <summary>
    /// 用户姓名
    /// </summary>
    public string Name { get; set; } = String.Empty;

    /// <summary>
    /// 用户编号
    /// </summary>
    public string Code { get; set; } = String.Empty;
    /// <summary>
    /// token版本
    /// </summary>
    public string JwtVersion { get; set; } = String.Empty;
    public string ExpireTime { get; set; } = String.Empty;

    /// <summary>
    /// 用户信息
    /// </summary>
    public object Data { get; set; } = String.Empty;

    public List<Claim> Claims { get; set; } = new List<Claim>();
}
```

```C#
public override CurrentUser CurrentUser {
    get
    {

        if (base.CurrentUser == null)
        {
            throw new MessageException("获取用户失败");
        }

        return base.CurrentUser;
    }
}
```

