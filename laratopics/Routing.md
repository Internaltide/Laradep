# Laravel Routing


## 基礎路由
所有的路由都應該定義在routes目錄內的路由文件，而這些路由文件預設就會自動被框架載入。<br/>
routes/web.php主要用來定義web界面的路由，並預設就會套用web中介層，以提供session跟CSRF保護<br/>
等功能；routes/api.php主要用來定義接口，預設套用api中介層。<br/>
另外，RouteServiceProvider預設還會為routes/api.php內的所有路由定義一個route group，即routes/api.php<br/>
內的所有路由的訪問URL預設都會添加前綴/api，所以不用再自行添加。<br/>
若要更改預設的前綴名稱就必須到RouteServiceProvider Class裡去更改。<br/>
```
// 定義路由
Route::get('/user', 'UserController@index');

// 閉包路由
Route::get('/user', function(){
    ......
});

// 對應的訪問網址
http://your-app.test/user
```

### 可用的路由方法
提供的HTTP動詞路由
```
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

有時候需要讓註冊一個可以應對到多個 HTTP 動詞的路由
```
// 該路由可接收get、post
Route::match(['get', 'post'], '/', function () {
    //
});

// 該路由可對應任意的HTTP動詞
Route::any('foo', function () {
    //
});
```

### CSRF保護
任何指定POST、PUT、DELETE的Web路由，都必須在表單中提供一個CSRF Token欄位；<br/>
否則，該請求將會被拒絕。
```
<form method="POST" action="/profile">
    @csrf
    ...
</form>

或

<form method="POST" action="/profile">
    {{ csrf_field() }}
    ...
</form>
```

### 重定向路由
利用Route::redirect這個方法，可以簡單快速的實現重定向，不需再定義完整的路由或者控制器
```
Route::redirect('/here', '/there', 301);
```

### 視圖路由
當路由定義的只是簡單回應一個視圖時，可以使用Route::view這個方法來快速實現。<br/>
該方法接受第一參數為URI、第二參數為視圖名稱及第三個可選的參數用來定義視圖<br/>
所需的資料。<br/>
```
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## 路由參數
### 指定必要參數
有時會希望透過URL片段來提供所需的必要參數，如以定義User ID為例
```
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```
也可以一次定義多個參數
```
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```
補充：<br/>
 - URL鑲嵌的路由參數，其名稱只能是英文字母與下底線並且需要使用大括號包住。
 - 路由參數會按定義的順序來注入到回調函數。

 ### 指定可選參數
 前述指定的參數為必要，所以少了必要參數的URL就會出現找不到路由的錯誤。<br/>
 若要指定可選參數，只需再參數名稱後面加上?即可。唯一要注意的是，可選參數<br/>
 必須要有預設值。<br/>
 ```
 Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
 ```

 ### 正則約束
我們可以替參數加上正則的約束，當給定的參數值通過正則檢查時才視為合法路由。<br/>
方式為使用where方法來指派正則約束，其接受第一參數為參數名稱，第二參數為正則表達式。
 ```
 Route::get('user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
 ```

 ### 全域約束
 可以在RouteServiceProvider的boot方法內，使用pattern方法為特定路由參數設定全域的正則約束，<br/>
 如此一來就不需要在每一個路由都重新設定一次。
```
/**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    // 指定所有了路由，只要包含路由參數id就必須通過此正則檢查
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```

## 命名路由
透過name方法為路由取名，可以更方便往後提取指定路由URL或跳轉時使用。
```
Route::get('user/profile', function () {
    //
})->name('profile');

Route::get('user/profile', 'UserProfileController@show')->name('profile');
```
```
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```
當路由存在路由參數時，需透過第二參數來傳遞路由參數
```
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

### 檢測當前路由
如果想要檢測當前請求是否指向某個命名路由，可以使用named方法，<br/>
例如在中介層檢查請求使否指向了特定的命名路由
```
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

## 路由群組
使用Route::group方法將大量的路由群聚在一起，並可為他們指定套用共享屬性，<br/>
如namedpace、中介層等等。
### 中介層
在group方法前先調用middleware來為群組內路由定義要使用的中介層，指定的中介層則會<br/>
按其在陣列的順序來執行。
```
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second Middleware
    });

    Route::get('user/profile', function () {
        // Uses first & second Middleware
    });
});
```
### 命名空間
為一群路由指派共同的命名空間，要注意的是RouteServiceProvider本身預設就提供了一個<br/>
共同命名空間**App\Http\Controllers**。所以通常要定義時，只需使用在App\Http\Controllers<br/>
之後的部分。
```
Route::namespace('Admin')->group(function () {
    // Controllers Within The "App\Http\Controllers\Admin" Namespace
});
```

### 子網域路由
為一群路由定義共同的網域，定義的網域也可配置路由參數，做為注入到<br/>
閉包或控制器中的參數。定義子網域路由時，需使用方法domain
```
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### 路由前綴
為一群路由定義其URL的前綴
```
Route::prefix('admin')->group(function () {
    // 此路由對訪問URI為/admin/users
    Route::get('users', function () {
        // Matches The "/admin/users" URL
    });
});
```

### 路由名稱前綴
為一群有命名的路由定義其名稱的前綴
```
Route::name('admin.')->group(function () {
    // 此路由的完整名稱會是admin.users
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

### 整合式路由(之前版本存在的方法，但5.6版官方文件卻未提及，遭移除? 待測試)
整合式地指定各類共享屬性
```
Route::group([
    'namespace' => 'Admin',
    'domain' => 'www.whatsoft.com',
    'prefix' => 'admin',
    'middleware' => 'auth'
    ], function () {
    Route::get('/', function ()    {
        // 使用 Auth 中介層
    });
});
```

## 路由模形綁定
一般透過路由參數注入ID到閉包或控制器Action時，就是為了利用該ID取得對應的User模形。<br/>
現在提供透過路由直接綁定，就可以更方便地直接注入完整的User模形來直接使用。<br/>
而當框架發現注入的模型不存在時，會自動產生404 HTTP response

### 隱式綁定
當路由參數名稱與注入的Eloquent模形物件變數相符時( 指{user} same with $user )，<br/>
Laravel就會自動注入與請求URI的ID值對應的模型實例。
```
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```
Ex. api/user/A123即會注入id為A123的User模形

#### 自訂鍵名
預設就是用id欄位來取得User模形，如果想改用其他欄位則需到User Eloquent Model中定義<br/>
getRouteKeyName方法，此法可以覆寫預設的id欄位。
```
/**
 * Get the route key for the model.
 *
 * @return string
 */
public function getRouteKeyName()
{
    return 'slug';
}
```
所以，前面例子來說，就會並成api/user/A123即會注入slug為A123的User模形

### 顯式綁定
注入之User模型所用的變數名稱為$user其實是相當直覺的，只是對於隱性綁定而言，<br/>
我們的路由參數也勢必要取成{user}才行。當你不想使用{user}這樣路由參數來代表使用者id時，<br/>
可以改用顯式綁定，明確告知要用什麼樣的路由參數名稱來綁定User模形。這樣一來就可以使用<br/>
自訂的路由參數而不必非得與注入的物件變數名稱一樣。<br/>
```
// 需在RouteServiceProvider的boot Method裡頭使用Route::model來定義
public function boot()
{
    parent::boot();

    Route::model('userid', App\User::class);
}
```
```
/**
 * 路由參數名稱為userid與變數$user並不相同，未符合隱性綁定的規則。但因為前面已經
 * 做過顯示綁定，明確告知了要用userid這個名稱來綁定User模形，所以此條路由定義還是
 * 能達到自動注入的效果。
 */
Route::get('profile/{userid}', function (App\User $user) {
    //
});
```

#### 自訂解析邏輯
如果你希望使用自己的解析邏輯，那麼你必須在RouteServiceProvider的boot 裡使用 Route::bind 方法。<br/>
你傳遞至 bind 方法的閉包會取得 URI 的部分值，然後回傳你想注入至路由的類別實例。
```
public function boot()
{
    parent::boot();

    Route::bind('user', function ($value) {
        return App\User::where('name', $value)->first() ?? abort(404);
    });
}
```

## 回調路由
使用Route::fallback定義一條回調路由，當出現請求無法匹配任何一條路由時，就會自動轉給<br/>
回調路由來處理。
```
Route::fallback(function () {
    //
});
```

## 速率限制
透過使用throttle中介層來提供存取速率的限制，並接受兩種參數，一為請求次數上限，<br/>
一為時間區段。以 **'throttle:5,2'** 例子來看，就是限制每兩分鐘內，至多可以接受5次請求。

### 靜態設定
```
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```
### 動態設定
可以將次數上限值改成字串型態的名稱如rate_limit，該名稱將會映射到模形的同名屬性($model->rate_limit)<br/>
而該模性類別屬性的值就代表當下的存取次數上限。
```
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

##  表單方法欺騙
現行Html表單並不直接支援PUT、PATCH 與 DELETE這三種HTTP動詞，若要使用就必須做一些特別的處理。<br/>
在Laravel中可簡單藉由定義名為_method的隱藏屬性來支援上述三種HTTP動詞(相關特別處理會由框架底層<br/>
自行處理掉)。
```
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```
或
```
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

## 訪問當前路由
欲取得處理當下請求得路由之訊息資料，可利用current、currentRouteName 及 currentRouteAction三種方法。
```
$route = Route::current();

$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```