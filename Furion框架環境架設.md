# 【.NET 5 | Furion】資料庫連接

## 架設Furion環境

以 Visual Studio 建立一空白方案。

![001.png (1025×630) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/001.png)

新增五個類別庫，分別為：
* `Furion.Application`
* `Furion.Core`
* `Furion.Database.Migrations`
* `Furion.EntityFramework.Core`
* `Furion.Web.Core`

![002.png (1366×768) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/002.png)

![003.png (1025×630) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/003.png)

建立後刪除附加的 `.cs` 檔。

![004.png (261×292) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/004.png)

再新增一 `ASP.NET Core Web應用程式` 專案。

![005.png (1025×630) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/005.png)

命名為 `Furion.Web.Entry`，選擇 `MVC架構`。

![006.png (926×555) (raw.githubusercontent.com)](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/006.png)

點擊各個專案並將版本改為 `net5.0`

![007]()

![008]()

解決方案&rarr;右鍵&rarr;**重建方案**
右鍵 `Furion.Application `加入專案參考，增加 `Furion.Core`

![009]()

其餘專案參考如下：

- `Furion.Application`：添加 `Furion.Core` 參考
- `Furion.Database.Migrations`：添加 `Furion.EntityFramework.Core `參考
- `Furion.EntityFramework.Core`：添加` Furion.Core` 參考
- `Furion.Web.Core`：添加` Furion.Application`、`Furion.Database.Migrations` 參考
- `Furion.Web.Entry`：添加 `Furion.Web.Core` 參考

在Furion.Core&rarr;右鍵&rarr;管理NuGet套件&rarr;安裝Furion
在Furion.Web.Entry&rarr;右鍵&rarr;管理NuGet套件&rarr;安裝Microsoft.EntityFrameworkCore.Tools

**重建解決方案**

![010]()

將Furion.Web.Entry設為啟動專案

![011]()

於*Furion.Web.Entry/Program.cs*中添加`Inject()`

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.Inject().UseStartup<Startup>(); // 新增於此
                });
```

於*Furion.Web.Entry/Startup.cs*中
​    在`ConfigureServices`中添加`AddInject()`

```c#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews().AddInject(); // 新增於此
        }
```

​	在Configure中添加`app.UseInject()`

![012]()

於Furion.Web.Core新增一類別, 命名為`FurionWebCoreStartup.cs`

改變`Startup.cs`與`FurionWebCoreStartup.cs`的程式碼如下

* Furion.Web.Entry\Startp.cs

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace Furion.Web.Entry
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }
        public IConfiguration Configuration { get; }
        public void ConfigureServices(IServiceCollection services)
        {
        }
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
        }
    }
}
```

*Furion.Web.Core/FurionWebCoreStartup.cs*

```c#
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace Furion.Web.Core
{
    public class FurionWebCoreStartup:AppStartup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews().AddInject();
        }
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                app.UseHsts();
            }
            app.UseHttpsRedirection();
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseInject();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
```

## 連接資料庫

### 配置連接字串

在Furion.EntityFramework.Core新建檔案`FurDbcontext.cs`和.josn檔`dbsetting.json`

`dbsetting.json`結構描述設定為https://json.schemastore.org/appsetting

![013]()

*Furion.EntityFramework.Core/dbsetting.json*

```c#
{
  "ConnectionStrings": {
    "DbConnectionString": ";Data Source=user;Initial Catalog=test;Integrated Security=True"
  }
}
```

*Furion.EntityFramework.Core/FurDbcontext*

```c#
using Furion.DatabaseAccessor;
using Microsoft.EntityFrameworkCore;

namespace Furion.EntityFramework.Core
{
    [AppDbContext("DbConnectionString")]   // 支持 appsetting.json名或連接字串
    public class FurDbContext : AppDbContext<FurDbContext>
    {
        public FurDbContext(DbContextOptions<FurDbContext> options) : base(options)
        {
        }
    }
}
```
**重建方案**

### AppStartup配置

Furion.EntityFramework.Core建新檔`FurionEntityFrameworkCoreStartup.cs`

*Furion.EntityFramework.Core/FurionEntityFrameworkCoreStartup*

```c#
using Furion.DatabaseAccessor;
using Microsoft.Extensions.DependencyInjection;

namespace Furion.EntityFramework.Core
{
    [AppStartup(600)] //數值越大則越先執行
    public sealed class FurEntityFrameworkCoreStartup : AppStartup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDatabaseAccessor(options =>
            {
                options.AddDbPool<FurDbContext>(DbProvider.SqlServer);
                //連甚麼資料庫就選甚麼
            }, migrationAssemblyName:"Furion.Database.Migration");
        }
    }
}
```

在Furion.EntituFramework.Core安裝套件Microsoft.EntityFrameworkCore.<u>SqlServer</u>(依據連接的資料庫做更改)
![014]()

### 資料庫上下文定位器

*Furion.Core/FurDbContextLocator.cs*

```
using Furion.DatabaseAccessor;

namespace Furion.Core
{
    public sealed class FurDbContextLocator : IDbContextLocator
    {
    }
}
```

**重建方案**

### 資料庫生成模型
1. Furion.Core新增一資料夾`Entity`

2. 於Furion的Github下載tools資料夾並放在專案的Furion.Core的資料夾

3. 工具&rarr;NuGet套件管理員&rarr;套件管理器主控台&rarr;預設專案設定為Furion.Core

4. 輸入 `&"C:\Users\user\source\repos\Furion_demo\Furion.Core\tools\cli.ps1"` (cli.ps1之完整路徑位置)

5. 輸入G&rarr;選擇連接字串&rarr;加載數據庫和視圖&rarr;選擇資料庫&rarr;立即生成&rarr;選擇Entity資料夾
![015]()
![016]()

  生成後腳本會自動在Entity資料夾內生成`.cs`檔

**重建方案**

### 數據庫操作基本設定

Furion.Application新增`AppService.cs`

*Furion.Application/AppService.cs*

```c#
using Furion.Core;
using Furion.DatabaseAccessor;
using Furion.DynamicApiController;
using System.Collections.Generic;

namespace Furion.Application
{
    /// <summary>
    /// DB資料庫
    /// </summary>
    public class cardAppService : IDynamicApiController
    {
        private readonly IRepository<Card> _repository;

        /// <summary>
        /// 
        /// </summary>
        /// <param name="repository"></param>
        public cardAppService(IRepository<Card> repository)
        {
            _repository = repository;
        }

        /// <summary>
        /// 新增
        /// </summary>
        /// <param name="card"></param>

        public void insert(Card card)
        {
            _repository.Insert(card);
        }
        /// <summary>
        /// 更新
        /// </summary>
        /// <param name="card"></param>
        public void Update(Card card)
        {
            _repository.Update(card);
        }
        /// <summary>
        /// 刪除
        /// </summary>
        /// <param name="card"></param>
        public void Delete(Card card)
        {
            _repository.Delete(card);
        }
        /// <summary>
        /// 查詢
        /// </summary>
        /// <returns></returns>
        public List<Card> GetAll()
        {
            return _repository.AsEnumerable();
        }
    }
}
```

Furion.Application右鍵 &rarr;屬性&rarr;建置&rarr;打勾XML文件檔案&rarr;名為Furion.Application.xml

**重建方案**

**執行**

![017]()

在網址後輸入`/api`

![018]()

### Swagger版面設計

*Furion.Application*新增`app.json`

![019]()

![020]()