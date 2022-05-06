# Tunning Procedures

### _識別框架_

* 確認想要作用的範圍 => 建議以HTTP Listner為一個單位(即一個Domain)
* 如果需作用在根目錄上=>不須考慮其他路由(因為皆會作用)
* 如果只需作用於特定路由而部分路由不作用=>根目錄必定不能作用
  * 原因為 Azure的 Custom Rules 在設定路由白名單時，仍然會對部分post的請求做出反應(JSON)
  * 舉例:
    * 域名 https://abc.om/
      * 路由:
        * route1 => https://abc.com/route1
        * route2 => https://abc.com/route2
      * 需求範圍:**根目錄、route1**
      * 如設定 custom rule:
        * RequestUri contains /route2 => allowed
        * route2底下的路由如有使用 json的資料格式作為POST資料，仍有機率遭到阻擋。
      * 所以仍需對route2做Tuning。

### Detection Mode

* 將所有受控規則(OWASP規則)開啟，並側錄所有流量。
* 通常需要側錄一個完整的使用者週期。

### 例外排除

1. 由原始碼找到所有使用的路由表(routing table)
2. 統計路由表內觸發告警的數量，並由高至低排列

```
let request_uri_list = pack_array(
    'uri1',
    'uri2',
);
let prefix_regex = strcat("(?i)^(", strcat_array(prefix_list, ")|("), ")");
AzureDiagnostics
| where requestUri_s matches regex prefix_regex
| summarize cnt=count() by requestUri_s
| sort by cnt
```

3\. 從欄位details統計每URI底下，觸發alert的**欄位**

```
AzureDiagnostics
| where request_Uri_s =~ "uri"
| where Category contains "ApplicationGatewayFirewallLog"
| summarize cnt=count() by details_message_s
| sort by cnt
```

4\. 由details\__data\__s欄位，判斷是否為False Positive

5\. 制定排除規則

1. 確定欄位為POST或GET的參數
   1. POST:確認ContentType編碼是否為
      1. application/x-www-form-urlencoded
      2. multipart/form-data
      3. application/json(CRS 3.2才支援)
      4. 如為上述情況=> 可使用custom rules作為排除
      5. 否則使用**原則設定**做為排除
   2. GET:Custom rules作為排除
2. 其他欄位則使用custom rules作為排除。
3. 如使用cookie數量太多，且作用域不明=> 統計所觸發的受控規則作為排除
