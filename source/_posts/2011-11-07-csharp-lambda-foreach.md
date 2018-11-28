---
title: lambda的foreach如何使用break/continue
date: 2011-11-07 19:19:28
categories:
- Back-end
- C#
tags:
- C#
---
在一般情況下，使用foreach時，遇到特定條件要讓他continue/break
會這樣去寫

<!--more-->

``` csharp
List<string> lstSql;
foreach(var sql  in lstSql)
{
     if(string.IsNullOrEmpty(sql))
       continue;//break;
}
```

但是在如果我們使用lambda還這樣寫，就會error
``` csharp
List<string> lstSql;
lstSql.Foreach(p=> if(stirng.IsNullOrEmpty(p)) continue;);
```

但我們又想要這樣做，該怎麼辦呢
必須知道lambda是怎麼處理foreach的
``` csharp
public static void ForEach<t>(this IEnumerable<t> sequence, Action<t> action)
{
    foreach(var item in sequence) action(item);
}
```

由以上可以得知，我們下continue/break對function來說是看不懂的
我的作法是直接寫return，這樣就等於離開function

``` csharp
List<string> lstSql;
lstSql.Foreach(p=> if(stirng.IsNullOrEmpty(p)) return;);
```