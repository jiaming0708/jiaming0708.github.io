---
title: WCF AJAX 服務建置於 https
date: 2022-03-23 14:01:46
updated: 2022-03-23 14:01:46
categories:
- Backend
tags:
- Backend
- WCF
- https
---

今天接到同事來的一個求救，有一個產品基於 WCF 服務建置 https 時無法正常運行，原本以為簡單，但卡了一陣子，特別來筆記一下。

<!-- more -->

# WCF 架站於 https

WCF 建置於 https 的資料超多超好查，微軟官方就有提供作法 [How to: Configure an IIS-hosted WCF service with SSL - WCF | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/framework/wcf/feature-details/how-to-configure-an-iis-hosted-wcf-service-with-ssl)，看到這樣的資料立馬動手試試看。

重點是在 binding，要增加一組 `basicHttpBinding` ，並且設定到 endpoint 上。

```xml
<bindings>
      <basicHttpBinding>
        <binding name="secureHttpBinding">
          <security mode="Transport">
            <transport clientCredentialType="None"/>
          </security>
        </binding>
      </basicHttpBinding>
</bindings>

<services>
      <service name="MySecureWCFService.Service1">
        <endpoint address=""
                  binding="basicHttpBinding"
                  bindingConfiguration="secureHttpBinding"
                  contract="MySecureWCFService.IService1"/>
      </service>
</services>
```

設定完很開心的就打開系統，測試下去，還是一樣收到 404 的錯誤...

為了要確認問題，決定先建立一個新的 WCF 服務，並且加上同樣的設定，新的專案的確是可以順利運行。
馬上回過頭檢查系統的寫法，突然發現一個不一樣的地方，服務的 endpoint 有加上 `behaviorConfiguration` 並且裡面的內容有 Ajax 的字樣，以及 binding `webHttpBinding`。

```xml
<endpoint address="" behaviorConfiguration="MySecureWCFService.Service1AspNetAjaxBehavior" binding="webHttpBinding" contract="MySecureWCFService.Service1"/>
```

趕快拿著剛建立的專案，再新增一個檔案 **WCF Service (Ajax enabled)** ，馬上看到設定出現在 web.config 中。先來查詢看看官方怎麼定義 binding ([System-provided bindings - WCF | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/framework/wcf/system-provided-bindings))。

| Binding                                                      | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|<div style="word-break: keep-all;">[BasicHttpBinding](https://docs.microsoft.com/en-us/dotnet/api/system.servicemodel.basichttpbinding)</div>| A binding that is suitable for communicating with WS-Basic Profile-conformant Web services, for example, ASP.NET Web services (ASMX)-based services. This binding uses HTTP as the transport and text/XML as the default message encoding. |
| [WebHttpBinding](https://docs.microsoft.com/en-us/dotnet/api/system.servicemodel.webhttpbinding) | A binding used to configure endpoints for WCF Web services that are exposed through HTTP requests instead of SOAP messages. |

從文字描述來看，`BasicHttpBinding` 會回傳 xml 格式，那 `WebhttpBinding` 則是 json，可以確定 Ajax enabled 的意義是指這個服務 response 格式不同。

這樣就可以確定做法，要將官方給的範例調整為 `WebHttpBinding` ，並且設定到 endpoint 上，就可以打完收工。

```xml
<bindings>
      <webHttpBinding>
        <binding name="secureHttpBinding">
          <security mode="Transport">
            <transport clientCredentialType="None"/>
          </security>
        </binding>
      </webHttpBinding>
</bindings>

<services>
      <service name="MySecureWCFService.Service1">
        <endpoint address=""
                  behaviorConfiguration="MySecureWCFService.Service1AspNetAjaxBehavior"
                  binding="webHttpBinding"
                  bindingConfiguration="secureHttpBinding"
                  contract="MySecureWCFService.IService1"/>
      </service>
</services>
```

# Reference

- [How to: Configure an IIS-hosted WCF service with SSL - WCF | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/framework/wcf/feature-details/how-to-configure-an-iis-hosted-wcf-service-with-ssl)
- [System-provided bindings - WCF | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/framework/wcf/system-provided-bindings)