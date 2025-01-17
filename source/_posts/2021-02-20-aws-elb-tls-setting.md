---
title: '[AWS] Elastic Load Balancer變更TLS的設定'
date: 2021-02-20 14:33:28
updated: 2021-02-20 14:33:28
categories:
- AWS
tags:
- AWS
---

網站的安全性中，其中有一項是關於 TLS 的檢查，建議是v1.2版以上。那服務如果是架設在 AWS 的 ELB 上，應該要怎麼做調整呢？

<!-- more -->

# TLS簡介

在瀏覽網站的時候，現在大部分的網站都會是 `https` ，與 http 的差異也就是加密，這個加密的方法是透過所謂的 SSL / TLS 來做進行。那為什麼要進行加密呢，因為 http 是明碼的傳輸，也就是說只要你想看，就能看到有哪些資料被傳送並且有機會進行修改。

SSL 的全名是 Secure Sockets Layer，用來確保瀏覽器與伺服器之間的資料傳輸，減少被修改的可能性，但因為存在著一些問題無法解決，因此在2016年的時候被 TLS 取代。

TLS 的全名是 Transport Layer Security，比 SSL 更強大的加密，並且解決掉了部分的問題。

> 現在外面所發的憑證基本上都是 TLS，但因為大家都習慣了，因此還是繼續稱呼為 SSL 憑證。

# 調整ELB設定

ELB 是 AWS 中的一個服務，全名是 Elastic Load Balancer 或稱為 Network Load Balancer，TLS 則是透過 **Security policy** 來做設定。

從上面介紹可以得知，會需要設定 TLS 的只有 443 也就是 https 那組 Listener ，我們可以看到預設的 **Security policy** 是 **ELBSecurityPolicy-2016-08** 。

![ELB Listeners](ELB-Listeners.png)

接著把 443 打勾並且點擊上方的 edit 按鈕，可以進入到編輯的頁面。

![listener setting](listener-setting.png)

請找到 **Security policy** 這個欄位，並且[參考這張表](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html#describe-ssl-policies)，重新調整為你所需要的即可。

因為我的目的只有要先關閉，TLS v1.0 以及 V1.1，因此最終選擇的是 **ELBSecurityPolicy-TLS-1-2-Ext-2018-06**，當然可以的話就選擇最安全的設定 **ELBSecurityPolicy-FS-1-2-Res-2020-10**。

> 如果選擇的越安全，有可能瀏覽器支援度越低，還是要根據使用者的瀏覽器使用狀況來做決定。

![security_policy_fs](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/security_policy_fs.png)

![security_policy_tls](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/images/security_policy_tls.png)

# 參考

* [What is Transport Layer Security (TLS)?](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)
* [簡介 SSL、TLS 協定](https://ithelp.ithome.com.tw/articles/10219106)
* [何謂SSL、TLS 以及HTTPS？為什麼網站都須要安裝SSL？](https://www.nss.com.tw/whyssl/)
* [TLS listeners for your Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html#describe-ssl-policies)