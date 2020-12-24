# .NET 5 | Furion 資料庫連接

> [**.NET 5 | Furion 資料庫連接**](#net-5-|-furion-資料庫連接)
>
>   * [架設 Furion 環境](#架設-furion-環境)
>     * [重建解決方案](#重建解決方案)
>   * [連接資料庫](#連接資料庫)
>     * [配置連接字串](#配置連接字串)
>     * [AppStartup 配置](#appstartup-配置)
>     * [資料庫上下文定位器](#資料庫上下文定位器)
>     * [資料庫生成模型](#資料庫生成模型)
>     * [數據庫操作基本設定](#數據庫操作基本設定)
>     * [Swagger 版面設計](#swagger-版面設計)
>     * [Swagger 輸入優化](#swagger-輸入優化)
>     * [Logging 寫入](#logging-寫入)
>     * [簡易日誌寫入](#簡易日誌寫入)

## 架設 Furion 環境

以 Visual Studio 建立一空白方案。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/001.png)

新增五個類別庫，分別為：
* `Furion.Application`
* `Furion.Core`
* `Furion.Database.Migrations`
* `Furion.EntityFramework.Core`
* `Furion.Web.Core`

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/002.png)

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/003.png)

建立後刪除附加的 `.cs` 檔。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/004.png)

再新增一 `ASP.NET Core Web應用程式` 專案。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/005.png)

命名為 `Furion.Web.Entry`，選擇 `MVC架構`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/006.png)

點擊各個專案並將版本改為 `net5.0`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/007.png)

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/008.png)
重建方案後，右鍵 `Furion.Application` 加入專案參考，增加 `Furion.Core`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/009.png)

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/010.png)

其餘專案參考如下：

- `Furion.Application`：添加 `Furion.Core` 參考。
- `Furion.Database.Migrations`：添加 `Furion.EntityFramework.Core` 參考。
- `Furion.EntityFramework.Core`：添加 ` Furion.Core` 參考。
- `Furion.Web.Core`：添加 ` Furion.Application`、`Furion.Database.Migrations` 參考。
- `Furion.Web.Entry`：添加 `Furion.Web.Core` 參考。

右鍵 `Furion.Core`→`管理NuGet套件`→安裝 `Furion`。

右鍵 `Furion.Web.Entry`→`管理NuGet套件`→安裝 `Microsoft.EntityFrameworkCore.Tools`。

### 重建解決方案

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/011.png)

將 `Furion.Web.Entry` 設為啟動專案。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/012.png)

* Furion.Web.Entry\Program.cs，添加 `Inject()`。

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.Inject().UseStartup<Startup>(); // 新增於此
                });
```

* Furion.Web.Entry\Startup.cs，在 `ConfigureServices` 中添加 `AddInject()`。

```c#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews().AddInject(); // 新增於此
        }
```

* Furion.Web.Entry\Startup.cs，在 `Configure` 中添加 `app.UseInject()`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/013.png)

於 `Furion.Web.Core` 新增一類別, 命名為 `FurionWebCoreStartup.cs`，並依下方指示修改 `Startup.cs` 與 `FurionWebCoreStartup.cs` ：

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

* Furion.Web.Core\FurionWebCoreStartup.cs

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

在 `Furion.EntityFramework.Core` 新建檔案 `FurDbcontext.cs` 和 `dbsetting.json`。

`dbsetting.json` 結構描述設定為 `https://json.schemastore.org/appsetting`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/014.jpg)

右鍵 `dbsetting.json`→`屬性`→`複製到輸出目錄`→`更新時複製`。

* Furion.EntityFramework.Core\dbsetting.json

```c#
{
  "ConnectionStrings": {
    "DbConnectionString": ";Data Source=user;Initial Catalog=test;Integrated Security=True"
  }
}
```

* Furion.EntityFramework.Core\FurDbcontext.cs

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
確認無誤後重建方案。

### AppStartup 配置

`Furion.EntityFramework.Core` 建新檔 `FurionEntityFrameworkCoreStartup.cs`。

* Furion.EntityFramework.Core\FurionEntityFrameworkCoreStartup

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

在 `Furion.EntituFramework.Core` 安裝套件 `Microsoft.EntityFrameworkCore.SqlServer`（依據連接的資料庫做更改）。
![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/015.png)

### 資料庫上下文定位器

* Furion.Core\FurDbContextLocator.cs

```csharp
using Furion.DatabaseAccessor;

namespace Furion.Core
{
    public sealed class FurDbContextLocator : IDbContextLocator
    {
    }
}
```

確認無誤後重建方案。

### 資料庫生成模型
1. `Furion.Core` 新增一資料夾`Entity`。

2. 於 `Furion` 的 Github 下載 tools 資料夾，並放在專案的 `Furion.Core` 的資料夾。

3. `工具`→`NuGet套件管理員`→`套件管理器主控台`→預設專案設定為`Furion.Core`。

4. 輸入 `&"C:\Users\user\source\repos\Furion_demo\Furion.Core\tools\cli.ps1"`（`cli.ps1` 之完整路徑位置）。

5. 輸入 G→`選擇連接字串`→`加載數據庫和視圖`→`選擇資料庫`→`立即生成`→選擇 `Entity` 資料夾。
![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/016.png)
![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/017.png)

生成後腳本會自動在 `Entity` 資料夾內生成 `.cs` 檔。

確認無誤後重建方案。

### 數據庫操作基本設定

`Furion.Application` 新增 `AppService.cs`

* Furion.Application\AppService.cs

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

右鍵 `Furion.Application`→`屬性`→`建置`→打勾`XML文件檔案`→名為`Furion.Application.xml`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/018.png)

確認無誤後重建方案並執行。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/019.png)

在網域名後加上 `/api`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/020.png)

### Swagger 版面設計

* `Furion.Application` 新增 `app.json`

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/021.png)

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/022.png)

其餘基本資訊的標註：

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/023.png)

### Swagger 輸入優化

`新增`：

因 `Furion` 預設之新增資料的界面輸入不順暢，因此修改新增功能的程式碼。

```csharp
        /// <summary>
        /// 增加一條
        /// </summary>
        /// <param name="Id"></param>
        /// <param name="name"></param>
        /// <param name="amount"></param>
        /// <param name="detail"></param>
        /// <param name="price"></param>
        /// <returns></returns>
        public void ADD([Required][FromHeader] int Id, [FromHeader] string name, [FromHeader] string amount, [FromHeader] string detail, [FromHeader] string price)
        {
            var AAAAA = new Aaaaa();
            AAAAA.CId = Id;
            AAAAA.Name = name;
            AAAAA.Amount = amount;
            AAAAA.Detail = detail;
            AAAAA.Price = price;
            _repository.InsertNow(AAAAA);
        }
```

`更新`：

```csharp
        /// <summary>
        /// 更新一條
        /// </summary>
        public async Task Update([Required][FromHeader] int CId, [FromHeader] string Name, [FromHeader] string Amount, [FromHeader] string Detail, [FromHeader] string Price)
        {
            await _repository.SqlNonQueryAsync("UPDATE Aaaaa SET Name = '" + Name + "', Amount= '" + Amount + "', Detail= '" + Detail + "', Price ='" + Price + "' WHERE C_Id = '" + CId + "';");} //SQL語法

```

`以 ID 查詢`：

```csharp
        /// <summary>
        /// ID查詢
        /// </summary>
        /// <param name="id"></param>
        public async Task<Aaaaa> Find(int id)
        {
            var person = await _repository.FindAsync(id);
            return person.Adapt<Aaaaa>();
        }
```

`條件查詢`：

```csharp
        /// <summary>
        /// 條件查詢
        /// </summary>
        /// <param name="name"></param>
        /// <param name="detail"></param>
        /// <returns></returns>
        public async Task<List<Aaaaa>> Search([FromQuery] string name, [FromQuery] string detail) //可更改查詢的條件
        {
            var aaaaas = _repository.Where(u => u.Name.Contains(name), u => u.Detail.Contains(detail));
            return await aaaaas.ToListAsync();
        }
```

### Logging 寫入

於 `Furion.Core` 層安裝 `Furion.Extras.Logging.Serilog` 套件。

`Furion.Web.Entry\Program.cs` 中插入 `UseSerilogDefault()`。

![](https://raw.githubusercontent.com/DynasafeAiTemp/DevDocs/main/Images/Furion資料庫連接操作手冊/024.png)

>  如需客製化寫入可參考 `Serilog` 相關配置設定。

替換 `Furion.Web.Entry*\*appsetting.json` 預設 Log 内容：

找出以下程式碼區塊

```csharp
"Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  }
```

替換為

```csharp
"Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "System": "Warning",
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information",
        "Microsoft.EntityFrameworkCore": "Information"
      }
    }
  }
```

在 `FurionWebCoreStartup.cs` 加入 `app.UseSerilogRequestLogging();`。

> 必須在 `UseStaticFiles` 和 `UseRouting` 之間。

之後便會自動記錄日誌及下載日誌檔於 `.\Furion.Web.Entry\logs` 內。

### 簡易日誌寫入

依照以下範例，在任意處插入：

```csharp
"百小僧 新增了一条记录".LogInformation<HomeController>();

"程序出现异常啦".LogError<HomeController>();

"这是自定义类别日志".LogInformation("类别");
```

