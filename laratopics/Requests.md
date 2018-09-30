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
## Cookies
## Files