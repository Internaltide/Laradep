# Laravel CSRF Protection

> CSRF是 Cross-site request forgery 的縮寫，一種跨站請求偽造，也可以稱為XSRF。<br/>
> 對比於XSS攻擊，XSS是利用User對網站的信任；CSRF則是利用網站對登入使用者的信任。<br/>
> 而且，簡單的身分驗證只能保證請求發自某用戶的瀏覽器，卻無法保證請求本身是來自用戶。<br/>
> 攻擊者會通過一些技術手段欺騙用戶的瀏覽器去存取一個自己曾經認證過的網站並執行<br/>
> 一些惡意操作，如發信、甚至轉帳造成用戶損失<br/>
>
> Refrence: https://blog.techbridge.cc/2017/02/25/csrf-introduction/


## 簡介
### 保護機制
在後端，Laravel會自動為每一個用戶Session產生一個隨機Token，這個Token用來檢驗已通過身份驗證的User<br/>
是否真的是請求的發出者。無論什麼狀況，應用程式的網頁表單都應該包含一個CSRF Token的隱藏欄位，<br/>
以便web中介層可以正常驗證請求並發揮CSRF保護的作用。<br/>

我們可以使用 @csrf Blade指令来產生Token字段：
```
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```
VerifyCsrfToken 中介層會負責比對請求中的token是否與會話中的token相符，以判斷請求<br/>
來源是否真的來User自行發起。而該中介層預設就包含在web中介層中，所以route/web內定義的<br/>
路由都會自動套用CSRF保護機制。所以，若表單為設置CSRF字段的話，就會提交失敗。

### CSRF Token 與 Javascript
在Laravel建構Javascript驅動的應用時，仍可以很方便的透過Javascript Http Lib來對每一個請求附加<br/>
CSRF Token。預設，resources/assets/js/bootstrap.js會使用Axios Http Lib註冊一個csrf-token Meta Tag。<br/>
如果用的不是這種函式庫，就必須自行配製產生csrf token的行為了。

## 設定CSRF保護的例外URIs
某些情境下，可能會希望某些URI不要有CSRF的保護。典型來說，可以透過將路由放到routes/web以外<br/>
的地方來避過CSRF檢查。但Larvel允許你經由VerifyCsrfToken 中介層來設定例外排除，方法就是把要排除<br/>
的URI添加到VerifyCsrfToken中介層類別的$expect陣列屬性中。
```
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ];
}
```
> PS. 在執行測試時，CSRF保護機制將會自動停用

## X-CSRF-TOKEN
除了檢查Post過來的CSRF Token外，VerifyCsrfToken中介層還會檢查請求Header中是否<br/>
含有X-CSRF-TOKEN Meta Tag。所以，你也可以將CSRF Token放到Header裡頭。
```
<meta name="csrf-token" content="{{ csrf_token() }}">
```
加入Meta Tag後，就可以用諸如Jquery這類的Lib將token加到所有請求Header中。<br/>
這樣就可以讓所有的Ajax請求都具有CSRF的保護。
```
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

## X-XSRF-TOKEN
Laravel還會將CSRF Token儲存在名為X-XSRF-TOKEN的cookie中，而這個cookie<br/>
則會被包含在每一次的請求回應中，並方便我們直接例用來設定X-XSRF-TOKEN<br/>
Request Header。不過，對於一些像是Angular或Axios則會自動處裡掉而無須手動<br/>
介入處理。
