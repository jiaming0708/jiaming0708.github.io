---
title: '[IIS] Gitea 搭配 reverse proxy 的問題排解'
date: 2021-04-19 21:20:54
categories:
- IIS
tags:
- IIS
---

繼前一篇 {% post_link iis-reverse-proxy-header %} 後，又持續的跟 IIS 奮鬥了一陣子，才讓 Gitea 搭配 reverse proxy 的運作是正確的！

<!-- more -->

# 無法建立PR

## 問題描述

一進去 repo 的畫面，直接點選 `建立合併請求` 會提示 **404** 的錯誤，這時候的 URL 如下

> git.com/jimmy/test/compare/master...master

但這時候如果切換到其他 branch 再來建立 PR 的話，就可以正常顯示。

## 解法

IIS 會針對一些附檔名做一些處理，這邊要將 IIS 設定成忽略所有的副檔名，全部交給 Gitea server 做處理。在 web.config 的設定中加上以下的設定

```xml
<configuration>
    <system.webServer>
			<security>
        <!-- 將所有的副檔名都排除 -->
				<fileExtensions allowUnlisted="true">
					<clear />
				</fileExtensions>
			</security>
    </system.webServer>
</configuration>
```

# Push 認證失敗

## 問題描述

可以使用 domain 做 clone/fetch/pull 等行為，但 push 的時候會一直跳錯誤 **401 未授權** 。但只要不使用 domain 馬上可以正常的使用，IIS 就是你的鍋阿！！

## 解法

> 記得設定要將 `HTTP_X_ORIGINAL_ACCEPT_ENCODING` 及 `HTTP_ACCEPT_ENCODING` 加到**伺服器變數**中。

```xml
<rules>
  <rule name="ReverseProxyInboundRule1" stopProcessing="true">
      <match url="(.*)" />
      <conditions logicalGrouping="MatchAll" trackAllCaptures="false" />
      <serverVariables>
          <set name="HTTP_X_ORIGINAL_ACCEPT_ENCODING" value="HTTP_ACCEPT_ENCODING" />
          <set name="HTTP_ACCEPT_ENCODING" value="" />
      </serverVariables>
      <action type="Rewrite" url="http://localhost:3000{UNENCODED_URL}" />
  </rule>
</rules>
<outboundRules>
    <rule name="RestoreAcceptEncoding" preCondition="NeedsRestoringAcceptEncoding">
        <match serverVariable="HTTP_ACCEPT_ENCODING" pattern="^(.*)" />
        <action type="Rewrite" value="{HTTP_X_ORIGINAL_ACCEPT_ENCODING}" />
    </rule>
    <preConditions>
        <preCondition name="NeedsRestoringAcceptEncoding">
            <add input="{HTTP_X_ORIGINAL_ACCEPT_ENCODING}" pattern=".+" />
        </preCondition>
    </preConditions>
</outboundRules>
```
# 結論

IIS 的 reverse proxy 設定相對於 nginx 複雜，不過他們的功能定義上本身就不一樣，所以也不能單純的這樣比。要使用的話就有許多地方要先設定好來，不然會比較多的小問題。

另外我有找到一些網路上推薦的設定，但可能我沒有注意到那些問題，這邊附上完整的 web.config。

{% gist bfd0837a9cd224426e334c793cf9ec53 web.config %}

# 參考

* [[IIS\][GitLab] 利用 IIS Reverse Proxy 將 GitLab 加上 Https](https://exfast.me/2018/09/iis-gitlab-using-iis-reverse-proxy-to-add-gitlab-to-https/)

* [[IIS\][GitLab] Reverse Proxy 後的小問題](https://exfast.me/2018/12/iis-gitlab-reverse-proxy-small-problem/)

* [Gitea document - reverse proxies](https://docs.gitea.io/en-us/reverse-proxies/#iis)

