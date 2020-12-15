##架設Furion環境

Visual Studio建立一空白方案

![image-20201214171338998](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214171338998.png)

新增五個類別庫
分別為:
Furion.Application 
Furion.Core     
Furion.Database.Migrations
Furion.EntityFramework.Core     
Furion.Web.Core

![image-20201214172024802](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214172024802.png)

![image-20201214172039775](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214172039775.png)

​	建立後刪除附加的.cs檔
![image-20201214172136037](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214172136037.png)

再新增一專案 ASP.NET Core Web應用程式

![image-20201214172353657](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214172353657.png)

命名為Furion.Web.Entry
選擇MVC架構

![image-20201214172407054](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214172407054.png)

點擊各個專案並將版本改為 ==net5.0==

![image-20201214173002989](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214173002989.png)

![image-20201214173020168](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214173020168.png)

解決方案&rarr;右鍵&rarr;**重建方案**
右鍵Furion.Application加入專案參考，增加`Furion.Core`

![image-20201214173118797](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214173118797.png)

其餘專案參考如下

- Furion.Application：添加 `Furion.Core` 參考
- Furion.Database.Migrations：添加 `Furion.EntityFramework.Core `參考
- Furion.EntityFramework.Core：添加` Furion.Core` 參考
- Furion.Web.Core：添加` Furion.Application`, `Furion.Database.Migrations` 參考
- Furion.Web.Entry：添加 `Furion.Web.Core` 參考

在Furion.Core&rarr;右鍵&rarr;管理NuGet套件&rarr;安裝Furion
在Furion.Web.Entry&rarr;右鍵&rarr;管理NuGet套件&rarr;安裝Microsoft.EntityFrameworkCore.Tools

**重建解決方案**

![image-20201214175740182](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214175740182.png)

於Furion.Web.Entry/Program.cs中添加`Inject()`

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.Inject().UseStartup<Startup>(); //加在這裡
                });
```

於Furion.Web.Entry/Startup.cs中
​    在`ConfigureServices`中添加`AddInject()`

```c#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews().AddInject(); //here
        }
```

在Configure中添加app.UseInject()

![image-20201214180211153](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20201214180211153.png)