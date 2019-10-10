---
title: '[C#] 使用Autofac'
date: 2014-09-17 09:30:27
categories:
- 
tags:
- MVC
- C#
---
最近接觸MVC...(好像也接觸一兩年了＠＠...
所看到的範例程式，都是使用Interface來呼叫method

<!--more-->

去了解後才知道這樣做有一個好處
以後若要進行客製，可以直接替換成另一個class來寫該method的內容，而不用改寫呼叫該method的程式
這個有一個詞叫做DI或是IoC，想要知道這塊的朋友，請搜尋一下囉

使用此作法，其實有很多的方法可以做到
今天要來說的是Autofac，這類的教學其實已經很多了
那我今天的重點會是在如何讓Autofac幫我們自動載入Interface到Controller的建構子中

那我們就先來準備一下Interface及實作的class

``` csharp
public interface ILotService
{
    LotData GetLotByLot(string lot); 
}

public class LotService : ILotService
{
    public LotData GetLotByLot(string lot)
    {
          var data = new LotData();
          return data;
    }
}
```

那接著我們在Controller加上建構子，並用ILotService當做參數
``` csharp
public class HomeController : Controller
{
    public HomeController(ILotService lotService)
    {
    }
 
    public ActionResult Index()
    {
        return View();
    }
}
```

以上都好了以後，直接去跑的話，系統一定死給你看
接下來我們要把Autofac加入，新增一個AutofacConfig的class到App_Start
在AutofacConfig裡面加入Bootstrapper的method

```csharp
public class AutofaceConfig
{
    public static void Bootstrapper()
    {
        var builder = new ContainerBuilder();
 
        //直接將Services專案中的所有Interface加入，可免除後來新增的忘記加入
        builder.RegisterAssemblyTypes(typeof(ILotService).Assembly).AsImplementedInterfaces();
        //可直接在Controller的建構子，使用Interface的參數
        builder.RegisterControllers(Assembly.GetExecutingAssembly());
        //交由MVC的Dependency處理
        var container = builder.Build();
        DependencyResolver.SetResolver(new AutofacDependencyResolver(container));
    }
}
```

想要做到Controller的建構子能夠自動載入Interface，重點是在使用builder.RegisterControllers
最後再Global的Application_Start裡面加上AutofaceConfig.Bootstrapper();

``` csharp
protected void Application_Start()
{
    AreaRegistration.RegisterAllAreas();
    FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    RouteConfig.RegisterRoutes(RouteTable.Routes);
    BundleConfig.RegisterBundles(BundleTable.Bundles);
    //載入autofac
    AutofaceConfig.Bootstrapper();
}
```