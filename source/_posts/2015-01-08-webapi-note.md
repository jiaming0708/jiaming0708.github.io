---
title: WebAPI使用
date: 2015-01-08 17:35:46
categories:
- Back-end
- C#
tags:
- MVC
- C#
---

以前在寫API時，都寫在Controller，而不是ApiController
這次的專案，總覺得應該要標準化改走ApiController
沒想到遇到了不少問題

1.預設的Route下，每個ApiController只允許一個Get/Delete/Post
若要同一個ApiController底下支援多個，可以修改WebApiConfig

``` csharp
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        // Web API 設定和服務
        config.EnableCors();
        // Web API 路由
        config.MapHttpAttributeRoutes();
 
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{action}"
        );
 
        //config.Routes.MapHttpRoute(
        //   name: "DefaultApiWithAction",
        //   routeTemplate: "api/{controller}/{action}",
        //   defaults: new { id = RouteParameter.Optional }
        //);
    }
}
```

2.ApiController的Route會認前面的字樣，呼叫的方法有限制
例如，開頭為Get就不支援Get以外的呼叫
若開頭都不在標準方法內，則需使用Post

``` csharp
//使用Get
public ReturnDataModel GetUserData()
{
}

//使用Post
public ReturnDataModel SignIn()
{
}
```

3.Post的傳遞參數，若為物件，則只能接受一個物件

``` csharp
//可以多個string
public ReturnDataModel SignIn(string user, string password)
{
}

//將user和password組合成一個物件
public class UserData
{
    public string UserName{get;set;}
    public string Password{get;set;}
}

//可以一個物件
public ReturnDataModel SignIn(UserData data)
{
}

//不可以一個物件以上，必須將這兩個參數合成一個物件
public ReturnDataModel SignIn(UserData data, string device)
{
}
```