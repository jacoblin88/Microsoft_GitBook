# Structure

### ApplicationGateWay

* 底下有多個HTTP Listener
* 一組防火牆規則可作用的最大範圍
* 以網際網路來說，可把它想成一個統一的publicIP以下則都是轉發的結果。

### HTTP Listener(接聽器)

* 通常會搭配一個有效域名 => 客戶認知的單一作用範圍
* 以一個HTTP Listener匹配一個WAF規則，該域名以下的**所有路由**規則都會納入規則內。
  * 所以根目錄所匹配的規則，該規則會於HTTP Listener內生效
  * ex: https://www.abc.com/
* 底下有多個路由規則
  * 舉例:
    * https://www.abc.com/route1
    * https://www.abc.com/route2

### 路由規則

* 會搭配一個後端集區(一個邏輯單位)
* 根目錄不會有額外的路由
  * 如果需將防火牆規則放到某域名的**根目錄=>則該防火牆規則一定會作用在該 HTTP Listener底下所有目錄。**
