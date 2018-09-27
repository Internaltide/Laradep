# Laravel Middleware

> 中介層提供了一個方便的機制，讓我們可以在Http請求進入應用程式前進行必要的過濾。<br/>
> 舉例來說，Laravel提供了一中介層可以檢查使用者是否已經通過身分認證，如果尚未通過，<br/>
> 則將使用者跳轉回登入頁面。其他還有，CORS中介層為所有離開應用的響應添加合適的表頭，<br/>
> logging中介層為所有傳入的請求進行記錄。所有Laravel中介層都會放在app/Http/Middleware目錄下。<br/>
>
> 試著將中介層想成是Http請求需要通過的一系列之層，每一層都可以對請求進行檢查甚至拒絕請求。

## 定義中介層
使用Artisan建立中介層檔案，而創建的中介層預設就放在app/Http/Middleware
```
php artisan make:middleware CheckAge
```
以下面這個例子而言，請求中的age需大於200才會進入訪問，否則，跳轉回home這個URI
```
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }

        return $next($request);
    }
}
```
定義時，要讓請求繼續傳遞到應用程式的邏輯處理，必須使用$next這個回調函數並把$request參數傳遞進去。<br/>

> 所有中介層都是透過服務容器解析的，所以我們也可以在中介層類別的建構式中透過型別提式來將依賴注入。<br/>

### 前置中介層與後置中介層
這邊指的是在Http請求前或後運行的中介層，但無論前或後，都取決於中介層本身的定義。<br/>
**在請求前運行中介層任務**
```
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```
**在請求後運行中介層任務**
```
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

## 註冊中介層
### 全域中介層
如果想要某一個中介層在每一個Http請求都被運行，可以將該中介層類別列入<br/>
app/Http/Kernel.php的$middleware屬性中。

### 指派中介層到路由
若要在路由中使用自定的中介層，則必須先到app/Http/Kernel.php中將自訂的中介層添加到$routeMiddleware屬性<br/>
，並指定好自己要的鍵名，該鍵名會成為後續在路由中指定用的中介層名稱。
```
// Within App\Http\Kernel Class...

protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```
一旦中介層被定義到Http Kernel後，就可以為路由分配使用該中介層
```
Route::get('admin/profile', function () {
    //
})->middleware('auth');
```
一次配置多個要使用的中介層
```
Route::get('/', function () {
    //
})->middleware('first', 'second');
```
指定中介層時，使用完整的中介層類別名稱
```
use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);
```

### 中介層群組
有時候你可能希望使用單一名稱就可配置一系列中介層，讓整個中介層配置更為便利。<br/>
利用app/Http/Kernel.php的$middlewareGroups屬性就可以定義類似的中介層群組。
```
/**
 * The application's route middleware groups.
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],

    // 其他自定義中介層群組
    'GroupName' =>[
        ...
    ],
];
```
配置單一中介層與中介層群組的方式並無不同，僅僅只是中介層群組可方便地一次配置要用的多個中介層。
```
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});
```

## 中介層參數
中介層的handle方法亦可接受額外的參數。例如定義了一個中介層用來檢查已驗證使用者是否為<br/>
特定角色時，就必須定義額外傳遞的參數
### 傳入參數
配置路由時，透過使用符號 **:** 來串接使用的中介層參數，多個參數間則用 **,**串接。
```
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```
### 接受參數
```
<?php

namespace App\Http\Middleware;

use Closure;

class CheckRole
{
    /**
     * Handle the incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

## 可中斷的中介層
有時候會需要在Http響應返回到瀏覽器後才運行特定中介層。例如，session中介層就是在Http響應返回到<br/>
瀏覽器後才將session資料寫入儲存器中。這時候如果中介層有定義一個terminate方法，該方法會自動在Http<br/>
響應返回到瀏覽器後被觸發調用。
```
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // Store the session data...
    }
}
```
一旦建立了terminate方法，則該中介層就必須加到app/Http/Kernel.php的全域中介層清單中。(不太理解，Why??)<br/>

預設，調用terminate方法是透過服務容器產生的新中介層實例來調用，如果想要handle跟terminate都是透過<br/>
同一個中介層實例，則必須在容器註冊中介層的同時將其註冊成singleton類型才可以。