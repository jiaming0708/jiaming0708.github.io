---
title: '[DotnetCore] 實作原則型授權'
date: 2024-09-21 17:19:36
updated: 2024-09-21 17:19:36
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
- Authorization
thumbnail: /images/dotnet-core.png
---

接到一個需求是希望能夠針對某些 API 檢查使用者的權限，使用者的權限維護於資料庫中，例如權限名稱叫做 `Published`，底下會介紹如何使用 `Policy-based authorization` 的機制來實現。

<!-- more -->

> 環境為 Dotnet core 8

## 定義 AuthorizationHandler

寫一個 `AuthorizationHandler` 來處理權限邏輯，預期會在裡面檢查資料庫，確認使用者是否有對應的權限。

```c#
public class PermissionRequirement : IAuthorizationRequirement
{
    public string Permission { get; }

    public PermissionRequirement(string permission)
    {
        Permission = permission;
    }
}

public class PermissionHandler : AuthorizationHandler<PermissionRequirement>
{
    private readonly IUserPermissionService _userPermissionService;

    public PermissionHandler(IUserPermissionService userPermissionService)
    {
        _userPermissionService = userPermissionService;
    }

    protected override async Task HandleRequirementAsync(AuthorizationHandlerContext context, PermissionRequirement requirement)
    {
        // 取得使用者 ID
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        if (userId == null)
        {
            return Task.CompletedTask;
        }

        // 從資料庫中檢查是否擁有對應權限
        var hasPermission = await _userPermissionService.UserHasPermissionAsync(userId, requirement.Permission);

        if (hasPermission)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

## 直接定義 Policy

在 `Program.cs` 設定 `Authorization Policy`，增加一個 `RequirePublishedPermission` 的規則。

```c#
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("PublishedPermission", policy =>
        policy.Requirements.Add(new PermissionRequirement("Published")));
});
// 註冊自定義的授權處理器
builder.Services.AddSingleton<IAuthorizationHandler, PermissionHandler>();
```

以上處理完畢就能夠在 API 使用 `Authorize` 這個屬性來做限制。

```c#
[ApiController]
[Route("api/[controller]")]
public class ArticlesController : ControllerBase
{
		[Authorize(Policy = "PublishedPermission")]
    [HttpGet]
    public IActionResult Get()
    {
        // 只有擁有 "Published" 權限的用戶才可以訪問此方法
        return Ok(new { Message = "This is a protected API." });
    }
}
```

## 動態產生 Policy

上面的作法比較簡單，也相對侷限，必須要在 `Program.cs` 定義好，另一種作法就是自定義 Policy Provider 來動態的產生 Policy。

```c#
public class PermissionPolicyProvider : IAuthorizationPolicyProvider
{
    public DefaultAuthorizationPolicyProvider FallbackPolicyProvider { get; }

    public PermissionPolicyProvider(IOptions<AuthorizationOptions> options)
    {
        FallbackPolicyProvider = new DefaultAuthorizationPolicyProvider(options);
    }

    public Task<AuthorizationPolicy> GetPolicyAsync(string policyName)
    {
        // 如果 policyName 以 'Permission_' 開頭，則動態生成 policy
        if (policyName.StartsWith("Permission_", StringComparison.OrdinalIgnoreCase))
        {
            var policy = new AuthorizationPolicyBuilder()
                .AddRequirements(new PermissionRequirement(policyName.Substring("Permission_".Length)))
                .Build();

            return Task.FromResult(policy);
        }

        // 使用預設的 PolicyProvider 處理其他的 policy
        return FallbackPolicyProvider.GetPolicyAsync(policyName);
    }

    public Task<AuthorizationPolicy> GetDefaultPolicyAsync() => FallbackPolicyProvider.GetDefaultPolicyAsync();

    public Task<AuthorizationPolicy?> GetFallbackPolicyAsync() => FallbackPolicyProvider.GetFallbackPolicyAsync();
}
```

在 `Program.cs` 註冊

```c#
builder.Services.AddSingleton<IAuthorizationPolicyProvider, PermissionPolicyProvider>();
builder.Services.AddSingleton<IAuthorizationHandler, PermissionHandler>();
```

在 API 就能夠透過 Policy Provider 的機制去動態產生前一種作法的 Policy 並且檢查是否有權限。

```c#
[ApiController]
[Route("api/[controller]")]
public class ArticlesController : ControllerBase
{
		[Authorize(Policy = "Permission_Published")]
    [HttpGet]
    public IActionResult Get()
    {
        // 只有擁有 "Published" 權限的用戶才可以訪問此方法
        return Ok(new { Message = "This is a protected API." });
    }
}
```

更好的作法是建立一個 Attribute 用來取代 `[Authorize(Policy = "Permission_Published")]` 這個寫法。

```c#
public class PermissionAuthorizeAttribute : AuthorizeAttribute
{
    private const string POLICY_PREFIX = "Permission_";

    public PermissionAuthorizeAttribute(string rightName)
    {
        RightName = rightName;
    }

    public string RightName
    {
        get => Policy[POLICY_PREFIX.Length..];
        set => Policy = $"{POLICY_PREFIX}{value}";
    }
}
```

實際使用就可以改成這樣

```c#
[PermissionAuthorize("Published")]
```

## Reference

- [Policy-based authorization in ASP.NET Core - Learn Microsoft](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-7.0)
- [ASP.NET Core 6 使用 Policy Authorization 保護端點 - 余小章 @ 大內殿堂](https://www.dotblogs.com.tw/yc421206/2022/06/21/aspnet_core_6_use_policy_authorization)
