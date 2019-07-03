# Laravel Http Requests

>

## 獲得請求
若想要獲得請求實例，可透過在控制器的建構式或其他任意方法進行Illuminate\Http\Request的型別提示，
服務容器會自動進行依賴注入。
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

### 依賴注入與路由參數
若您的路由有使用到路由參數，則該參數應列在注入的依賴之後
```
Route::put('user/{id}', 'UserController@update');
```
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     *
     * @param  Request  $request
     * @param  string  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```

### 在路由閉包內獲取請求實例
您也可以在路由閉包中透過型別提示來取得Illuminate\Http\Request 實例
```
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

### 請求路徑與方法
Laravel Illuminate\Http\Request 繼承自Symfony\Component\HttpFoundation\Request，並且提供了<br/>
一系列的請求檢查方法

#### 取得請求的路徑
path方法會返回請求得路徑資訊，所以假若今天有一個請求來自**http://domain.com/foo/bar**，則path方法<br/>
就會回傳**foo/bar**。
```
$uri = $request->path();
```

#### 判斷請求路徑是否符合規則
使用is方法來判斷請求是否與指定的規則相互匹配
```
if ($request->is('admin/*')) {
    //
}
```

#### 取得請求的URL
欲取得請求的完整URL，可以使用url或fullUrl兩種方法。兩者的差異在於url不包含查詢字串，但fullUrl則包含。
```
// Without Query String...
$url = $request->url();

// With Query String...
$url = $request->fullUrl();
```

#### 獲取請求方法(Get、Post...)
使用method方法來取得請求使用的方法；使用isMethod判斷請求是否使用指定的方法
```
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

### PSR-7 型標準請求
PSR-7 標準制定了 HTTP 訊息的介面，其中包含了請求及回應。如果您不想要典型的Laravel請求而是想用<br/>
PSR-7標準請求，則必須先安裝一個必要套件，用來對典型的Laravel請求實例進行轉換成以支援PSR-7實作的請求實例。
```
composer require symfony/psr-http-message-bridge

composer require zendframework/zend-diactoros
```
一旦安裝完相關套件，透過ServerRequestInterface的型別提示，就可以注入PSR-7請求實例。
```
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    //
});
```
PS. 若在路由或控制器返回PSR-7響應，則框架會自動將其轉換回Laravel響應。


## 輸入修剪與標準化
預設，Laravel包含了 TrimStrings 和 ConvertEmptyStringsToNull 兩種全域型的中介層，提供了<br/>
輸入字串的修剪處裡與空資料轉換成Null值的動作，讓開發者無須特別擔心數據規範的問題。<br/>

如果你不想要這類的行為，就必須將兩個中介層從 App\Http\Kernel 的$middleware屬性中移除。

## 獲取輸入資料
### 取得所有提交資料
```
// Return Array Data
$input = $request->all();
```

### 取得指定的資料
```
$name = $request->input('name');

// With Default Value Assign
$name = $request->input('name', 'Sally');
```

#### 取得表單提交的陣列資料
「陣列」形式的輸入資料，可以使用「點」語法取得陣列
```
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

### 從查詢字串取得數據
input方法會從整份提交的請求中獲取資料，單然也包含查詢字串；但是query方法則只會從查詢字串中取得資料
```
$name = $request->query('name');

// With Default Value Assign
$name = $request->query('name', 'Helen');
```
當為傳遞任何參數，query方法的行為就是取得所有的查詢字串資料
```
$query = $request->query();
```

### 透過動態屬性取得資料
你也可以透過Illuminate\Http\Request的動態屬性來取得資料。舉例來說，如果表單包含一個<br/>
名稱為name的欄位，你就可以透過下列方式取得資料。
```
$name = $request->name;
```
PS. Laravel處理動態屬性的優先順序仍為先從請求本體中查詢，其次為路由參數。

### 獲取JSON資料
如果提交到應用的是JSON數據，只要Content-Type設定為正確的application/json，你就可以透過input來取得<br/>
JSON資料，並使用點式語法來讀取JSON數據。
```
$name = $request->input('user.name');
```

### 獲取輸入的部分資料
如果你只需要部分的提交資料，可以使用only或expect兩種方法來進行限定。
```
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```
PS. 注意!! only方法會回傳所有你指定鍵值的資料，但若指定鍵值並不存在則不返回。

### 檢查輸入資料是否存在
使用has方法來判斷資料是否存在，存在式會回傳true
```
if ($request->has('name')) {
    //
}
```
也可以傳遞陣列，一次判斷多筆資料
```
if ($request->has(['name', 'email'])) {
    //
}
```
判斷資料是否存在並且不為空
```
if ($request->filled('name')) {
    //
}
```

## 舊有輸入
### 自動保留輸入資料
Laravel允許你將此次請求的資料保留至下一個請求階段，這對於因表單驗證失敗而需重新填充表單值<br/>
的行為相當有用。如果使用的是內建的表單驗證功能，Laravel會自動保留而無須手動調用方法處理資料保留。

### 手動將輸入資料快閃到session中
Illuminate\Http\Request實例的flash方法會將本次的請求資料快閃到Session中，這讓下一次請求時可以使用它們。
```
$request->flash();
```
也可以改用flashOnly或flashExcept加以限制要保留的資料集，比如排除掉像密碼這類的敏感性資料。
```
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

### 快閃資料後重新跳轉
如果常常需要在快閃輸入資料後重新跳轉回上一頁，可以在使用redirect方法的同時，鏈式調用withInput方法
```
return redirect('form')->withInput();

return redirect('form')->withInput(
    $request->except('password')
);
```

### 取得舊輸入資料
使用Request實例的old方法可以取得上一次請求所快閃至session的資料
```
$username = $request->old('username');
```
Laravel也提供了等價的全域輔助方法old，這方法可以更方便地使用在Blade視圖中，如果資料不存在時則回傳Null。
```
<input type="text" name="username" value="{{ old('username') }}">
```

## Cookies
### 從請求中取得Cookie
由Laravel建立的Cookie都會經過加密並簽署。這意味著當客戶端擅自異動Cookie值時，Cookie就會失效。<br/>
你可以使用Request實例的cookie方法來從請求中取得Cookie資料。
```
$value = $request->cookie('name');
```
或者，也可以改用Cookie Facade的get方法來取得
```
$value = Cookie::get('name');
```

### 附加Cookie資料到響應中
在傳出去的Illuminate\Http\Response實例調用cookie方法即可夾帶Cookie資料，常用的三個參數定義依序為<br/>
cookie名稱、cookie值及cookie有效時間
```
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```
除了前述三個常用餐數外，cookie方法還接受其他幾個較不常用的餐數。而這些參數通常與原生PHP的setcookie函數<br/>
所用的參數有著相同的目的與定義。<br/>
 - path：cookie的服務器路徑<br/>
 - domain：cookie的域名<br/>
 - secure：是否需要透過加密連線傳輸<br/>
 - httpOnly：設定為false時，cookie似乎就不會加密了?(待測試)
```
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```
或者也可使用Cookie Facade的queue來附加一組 Cookies 到即將輸出的回應上。queue 方法接受一個  Cookie 實例或<br/>
建立 Cookie 實例所需的參數。在發送回應到瀏覽器之前，這些 cookie 會被附加到即將輸出的回應上。
```
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

### 創建Cookie
如果你想建立一個隨後可附加至響應的Symfony\Component\HttpFoundation\Cookie實例，可以使用<br/>
全域輔助方法cookie來建立。該創建的cookie實例如未被附加到響應，則不會傳遞到前端瀏覽器。
```
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

## Files
### 取得上傳的檔案
使用file方法從請求中取得上傳的檔案，該方法會回傳一個Illuminate\Http\UploadedFile實例。該實例則是延伸自<br/>
PHP SplFileInfo 類別，提供了一系列與檔案互動的方法。
```
$file = $request->file('photo');
```
也可透過動態屬性來取得上傳的檔案
```
$file = $request->photo;
```

### 判斷上傳的檔案是否存在於請求
使用hasFile方法檢查請求中是否存在上傳的檔案
```
if ($request->hasFile('photo')) {
    //
}
```

### 驗證上傳是否有效
除了檢查上傳的檔案是否存在外，你也可以透過 isValid 方法驗證上傳的檔案是否有效
```
if ($request->file('photo')->isValid()) {
    //
}
```

###  檔案路徑與擴展
UploadedFile類別提供了存取檔案完整路徑與擴展名。extension方法會重新根據檔案內容來判斷<br/>
並猜測可能之擴展，所以結果可能會與客戶端提供的不一致。(避免直接信任用戶端數據，可能有刻意的附檔竄改?)
```
$path = $request->photo->path();

$extension = $request->photo->extension();
```
PS. UploadedFile實例還有許多有用的方法，可自行參閱[API Documents](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html)

### 移動上傳的檔案
UploadedFile亦提供了相關方法來將上傳檔案移置到正確的儲存位置，在此之前你必須準備好配置的filesystem。

#### 自動生成檔名
第一參數為檔案儲存目錄位置，目錄需相對於設定檔的根目錄設定。另外，Laravel會自動產生unique ID作為檔案名稱，<br/>
所以設置的目錄參數並無須包含檔名。第二參數為可選的儲存磁碟名稱。<br/>
Method: **store**
```
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

#### 手動指定檔名
第一參數為檔案儲存目錄位置，定義同store。第二參數為指定的上傳檔案名稱。<br/>
第三參數為可選的儲存磁碟名稱。<br/>
Method: **storeAs**
```
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## 配置可信任的代理
Https連線建立相對於Http連線來說，其耗費的伺服器資源更甚，在連線請求多時，常導致伺服器負擔過重。<br/>
現今的負載平衡器常常會提供SSL Termination的功能，簡單說就是一種接管加密連線處理工作的功能，對外用Https<br/>
對內與web server的溝通則改用Http處理。因此，當加密工作轉交給前方的負載平衡器後，處於負載平衡後的<br/>
web服務器就能更專注在服務請求的工作處理上。這種中止於負載平衡器的加密連線就被稱做SSL Termination。<br/>

當您的Laravel應用是運行在具SSL Termination的負載平衡機制之後，你可能會發現你的Laravel應用並無法<br/>
為資源(由其是敏感性資料)建立加密連結，這是因為應用正在負載平衡器上透過80埠來進行流量的轉發，所以<br/>
Laravel會不知道應該建立的是加密連結。<br/>

為了解決這個問題，你必須使用Laravel內建的App\Http\Middleware\TrustProxies中介層，它使你能夠快速<br/>
地定義應用應該信任的負載平衡器或代理。你所定義的可信任代理必須列於中介層的$proxies屬性。除此之外，<br/>
也可設定那些標頭是可信的並且是受信任代理用來傳遞資訊用的。<br/>
(看完說明還是不理解加入信任代理與能否產生加密連結有何關係......)
```
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Fideloper\Proxy\TrustProxies as Middleware;

class TrustProxies extends Middleware
{
    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * The headers that should be used to detect proxies.
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```
PS. 若使用的是AWS Elastic Load Balancing，則標頭部分必須改用Request::HEADER_X_FORWARDED_AWS_ELB。<br/>
       更多可用的標頭常數可參閱線上文件[Symfony Trusting Proxies](https://symfony.com/doc/current/deployment/proxies.html)

### 信任所有的代理
如果您使用的是AWS或其它的雲端負載平衡服務，你可能會無法得知實際負載平衡器的IP位址，<br/>
所以可以改設成 **\*** 來信任所有的代理。
```
/**
 * The trusted proxies for this application.
 *
 * @var array
 */
protected $proxies = '*';
```
