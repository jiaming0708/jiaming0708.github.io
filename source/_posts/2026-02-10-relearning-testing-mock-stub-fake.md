---
title: 重新學習單元測試中的 Stub/Mock/Fake
date: 2026-02-10 10:42:55
updated: 2026-02-10 10:42:55
categories:
  - Testing
tags:
  - Testing
  - Dotnet
thumbnail:
---

趁著年前有點時間來讀一本放在桌上有段時間的書 **單元測試的藝術 - 以 JavaScript 為例**，裡面講到 Stub/Mock/Fake 差異的時候沒有很懂，因此重新學習了這三個的含義以及應用情境。

<!-- more -->

![cover](cover.png)

雖然書是用 JavaScript，但我最近主要碰的是 Dotnet，因此本文章的所有內容會以 Dotnet 為主

> Dotnet 10 + xUnit + Moq

## 情境一

假設有一個服務在算訂單總價，會去查會員折扣

``` csharp
public class OrderService
{
    private readonly IMemberService _memberService;

    public OrderService(IMemberService memberService)
    {
        _memberService = memberService;
    }

    public decimal CalculateTotal(string memberId, decimal amount)
    {
        var discount = _memberService.GetDiscount(memberId);
        return amount * (1 - discount);
    }
}
```

``` csharp
public interface IMemberService
{
    decimal GetDiscount(string memberId);
}
```

### Stub

這個測試手法的重點是 OrderService 的結果，GetDiscount 的呼叫只需要在意可以讓程式運作下去就好

因此建立一個 Stub 物件繼承 IMemberService 並且固定回傳數字

``` csharp
public class MemberServiceStub : IMemberService
{
    public decimal GetDiscount(string memberId)
    {
        return 0.1m; // 永遠給 10% 折扣
    }
}
```

``` csharp
[Fact]
public void CalculateTotal_With10PercentDiscount()
{
    var stub = new MemberServiceStub();
    var service = new OrderService(stub);

    var total = service.CalculateTotal("A123", 100);

    Assert.Equal(90, total);
}
```

### Mock

基於 Stub 的概念更往上增加確認方法有被呼叫，以及怎麼呼叫

``` csharp
[Fact]
public void Should_Call_MemberService_Once()
{
    var mock = new Mock<IMemberService>();
    mock.Setup(m => m.GetDiscount("A123"))
        .Returns(0.1m);

    var service = new OrderService(mock.Object);

    var total = service.CalculateTotal("A123", 100);

    Assert.Equal(90, total);
    mock.Verify(m => m.GetDiscount("A123"), Times.Once);
}
```

### Fake

有邏輯但不是完整的實作，通常用於當作假的資料庫類型

例如 MemberService 實際上有去拿資料庫的資料，因此我們用 Dictionary 存一些資料並且方法裡面可以拿到對應的設定，有一定程度的邏輯但又不是很完整

在這邊可以透過傳入參數的方式，一次驗證不同使用者並且有符合預期的計算

``` csharp
public class MemberServiceFake : IMemberService
{
    private readonly Dictionary<string, decimal> _discounts =
        new()
        {
            { "VIP", 0.2m },
            { "NORMAL", 0.05m }
        };

    public decimal GetDiscount(string memberId)
    {
        return _discounts.TryGetValue(memberId, out var d) ? d : 0;
    }
}
```

``` csharp
[Theory]
[InlineData("VIP", 80)]
[InlineData("NORMAL", 95)]
[InlineData("A123", 100)]
public void CalculateTotal_VipMember(string memberId, int expectedTotal)
{
    var fake = new MemberServiceFake();
    var service = new OrderService(fake);

    var total = service.CalculateTotal(memberId, 100);

    Assert.Equal(expectedTotal, total);
}
```

## 情境二

以情境一來說 stub 和 mock 的差異性沒有到很大，因此來調整一下程式，讓兩者有比較明顯的比較

在上線後發現有些使用者會多輸入到空白在前後，而找不到會員資料，因此要將傳進來的參數處理過

``` csharp
    public decimal CalculateTotal(string memberId, decimal amount)
    {
        // 過濾掉空白
        memberId = memberId.Trim();

        var discount = _memberService.GetDiscount(memberId);
        return amount * (1 - discount);
    }
```

就算加上了這個條件測試案例不用特別調整沒錯
但我們應該要新增測試資料，要來確保這樣的情境可以符合

為了要測試多組資料，使用 InlineData 的方式將會員資料傳進去，這樣測試案例都不特別動就可以達成目的

``` csharp
[Theory]
[InlineData("A123")]
[InlineData("A123 ")]
[InlineData(" A123")]
public void CalculateTotal_With10PercentDiscount(string memberId)
{
    var stub = new MemberServiceStub();
    var service = new OrderService(stub);

    var total = service.CalculateTotal(memberId, 100);

    Assert.Equal(90, total);
}
```

``` csharp
[Theory]
[InlineData("A123")]
[InlineData("A123 ")]
[InlineData(" A123")]
public void Should_Call_MemberService_Once(string memberId)
{
    var mock = new Mock<IMemberService>();
    mock.Setup(m => m.GetDiscount(memberId))
        .Returns(0.1m);

    var service = new OrderService(mock.Object);

    var total = service.CalculateTotal(memberId, 100);

    Assert.Equal(90, total);
    mock.Verify(m => m.GetDiscount(memberId), Times.Once);
}
```

調整以後，搭配修改前的程式沒問題，但 mock 的寫法搭配修改後的程式就掛了
因為 GetDiscount 預期拿到的是去除空白的資料也就是 `A123`，可以發現 stub 的寫法反而沒事

這邊就要來重新思考，在這個情境中 GetDiscount 被呼叫到是不是重要的事情
如果是重要並且就是要測試到會員資料應該是被 trim 過後的，那使用 mock 的寫法就沒有錯，那就應該要改測試的寫法，讓情境可以符合；若沒有需要，直接使用 stub 就可以了

> 若上面的測試要改正確，可以這樣調整

``` csharp
[Theory]
[InlineData("A123")]
[InlineData("A123 ")]
[InlineData(" A123")]
public void Should_Call_MemberService_Once(string memberId)
{
    var mock = new Mock<IMemberService>();
    mock.Setup(m => m.GetDiscount("A123"))
        .Returns(0.1m);

    var service = new OrderService(mock.Object);

    var total = service.CalculateTotal(memberId, 100);

    Assert.Equal(90, total);
    mock.Verify(m => m.GetDiscount("A123"), Times.Once);
}
```

## 總結

這三種測試替身各自有適合的使用場景：

| 類型       | 它的意義是什麼      | 你在測什麼                  | 典型特徵              | 常見誤用         |
| -------- | ------------ | ---------------------- | ----------------- | ------------ |
| **Stub** | 提供固定或可預期的回傳值 | **結果（Input → Output）** | 回傳資料，不驗證互動        | 為了方便卻改用 Mock |
| **Mock** | 驗證物件之間的互動    | **行為（有沒有被呼叫）**         | 可 Verify 次數、參數    | 驗證太多內部細節     |
| **Fake** | 簡化但可運作的真實實作  | **整體流程**               | 有邏輯，通常是 in-memory | 被拿來當 Stub 用  |

挑選的原則很簡單：如果只在意結果用 Stub，如果要驗證互動用 Mock，如果需要模擬複雜邏輯用 Fake
