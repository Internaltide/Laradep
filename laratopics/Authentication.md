# Laravel User Authentication


## Introduction
Laravel使用者認證套件的核心設計包含了"**守衛**"與"**提供者**"。<br/>
前者定義每個請求內如何對使用者進行身分驗證，預設則使用了larave內建的session guide，其會運用<br/>
session儲存器跟cookie來保存使用者的驗證狀態資料。<br/>
後者則定義如何從永久性儲存區取得用戶資料，預設內建了Eloquent或查詢建構器兩個提供者，<br/>
若開發者有特別的需求，也可以自行建立其他類型提供者供Laravel驗證程序使用。<br/>

若你的應用程式使用的認證方式並無太多特別需求，與傳統驗證方式相同的話，在Laravel內<br/>
實作認證會顯得非常的簡單，因為幾乎所有需要的東西都可以經由設定後就直接使用。<br/>
認證設定檔被放在 config/auth.php，其中幾個選項都另有良好的文件說明，可藉此來調整認證服務的行為。<br/>

###  資料庫注意事項
 - 預設App資料夾即存在user的eloquent模型，可以做為Eloquent認證驅動的底層模型。<br/>
   如果不想使用Eloquent也可以改用database認證驅動
 - 使用者資料庫的密碼欄位最少需要60個字元長
 - 使用者資料庫需要含有一個nullable且100個字元長的remember_token欄位以支援"**記住我**"這個功能。<br/>
   在資料庫遷移程式檔中，可以使用$table->rememberToken()來建立該欄位資料

**＊快速建立認證系統**<br/>
於全新乾淨的Laravel應用程式下，執行**php artisan make:auth**與**php artisan migrate**兩個指令<br/>
框架就會自動建構好整個應用程式的認證系統。<br/>
還可於http://your-app.dev/register或自定義的URL進行認證頁面的訪問<br/>

> ## 認證快速入門
### 使用內建基礎邏輯控制
在App\Http\Controllers\Auth這個命名空間下，Laravel已內建了以下四個認證控制器，並都引用了trait來包含<br/>
各自所需要的方法，開發者不必多作修改就可以處理掉多數的使用者認證行為。
 - RegisterController - 處理使用者註冊行為(Use Trait: **Illuminate\Foundation\Auth\RegistersUsers**)
 - LoginController - 處理使用者登入行為(Use Trait: **Illuminate\Foundation\Auth\AuthenticatesUsers**)
 - ForgotPasswordController - 處理忘記密碼行為(Use Trait: **Illuminate\Foundation\Auth\SendsPasswordResetEmails**)
 - ResetPasswordController - 處理重設密碼行為(Use Trait: **Illuminate\Foundation\Auth\ResetsPasswords**)

### 快速創建視圖
Laravel 提供了一個簡單的指令**make:auth**，可以快速生成身分認證所需的所有路由與視圖
```
php artisan make:auth
```
指令被執行後，會產生以下動作：<br/>
 - 產生HomeController作為登入後的跳轉入口
 - 創建所有所需視圖並放在resources/views/auth目錄下，<br/>
 - 建立基礎的layou視圖t並放在resources/views/layouts目錄。<br/>
以上所有視圖檔案則會使用Bootstrap CSS框架。<br/>

### 客製設定
#### 客製化重導路徑 ( V5.6的版本變更較大，實作時要注意適用的版本 )
使用者認證成功後的預設重導路徑為/home，我們可以透過定義redirectTo屬性或方法來改變重導路徑<br/>

透過相關驗證控制器的屬性 $redirectTo：<br/>
定義LoginController、RegisterController、ResetPasswordController的$redirectTo屬性來變更重導路徑<br/>
```
protected $redirectTo = '/';
```
接著，必須再更改中介層RedirectIfAuthenticated之handle方法內的邏輯，改用新的重導路徑URI<br/>

透過相關驗證控制器的 redirectTo 方法定義：<br/>
當需要動態定義重導路徑時，可改採用redirectTo方法。redirectTo方法的優先層級會高於redirect屬性<br/>
```
protected function redirectTo()
{
    return '/path';
}
```

#### 自訂驗證欄位
預設，Laravel驗證會使用email欄位，但可以透過在LoginController內定義<br/>
方法**username**來改變使用的驗證欄位。
```
public function username()
{
    // 驗證時，改用username作為驗證欄位
    return 'username';
}
```

#### 客製使用的守衛配置
自訂用於認證與註冊用的守衛，可在LoginController、RegisterController及ResetPasswordController內部
定義guard方法，並讓該方法回傳使用特定配置的守衛實例。
```
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    // 亦即回傳使用名為guard-name之守衛配置的守衛實例
    return Auth::guard('guard-name');
}
```

#### 自定義驗證與儲存
透過修改RegisterController內的validator跟create兩個方法來定義新的表單驗證規則與新用戶資料儲存方式<br/>
method: **validator**
```
protected function validator(array $data)
{
    return Validator::make($data, [
        'name' => 'required|string|max:255',
        'email' => 'required|string|email|max:255|unique:users',
        'password' => 'required|string|min:6|confirmed',
    ]);
}
```
method: **create**
```
protected function create(array $data)
{
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => Hash::make($data['password']),
    ]);
}
```

### 取得已驗證的使用者
要取得已驗證用戶的方法有二，一是透過facade Auth，另一則是透過request實例
#### via facade Auth
```
use Illuminate\Support\Facades\Auth;

// Get the currently authenticated user...
$user = Auth::user();

// Get the currently authenticated user's ID...
$id = Auth::id();
```
#### via Illuminate\Http\Request Instance
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProfileController extends Controller
{
    /**
     * Update the user's profile.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        // returns an instance of the authenticated user...
        $request->user();
    }
}
```

### 確認當下使用者是否通過驗證
```
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
} else {
    // The user is not logged in...
}
```
PS. 雖然可用Auth facade的check方法來確認使用者是否通過驗證，但在判斷是否允許存取路由與控制器時<br/>
還是傾向建議使用中介層來處理。

### 保護路由
Laravel內建了auth中介層，其可用來限制只有通過驗證的使用者可以訪問特定路由或控制器。<br/>
預設該中介層已經註冊進核心了，剩下要做的就只是在路由或控制器內套用該中介層。<br/>
#### 套用到路由
```
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth');
```
#### 套用到控制器
```
public function __construct()
{
    $this->middleware('auth');
}
```

### 重導未認證使用者
當中介層auth偵測到未通過認證的使用者時，將會返回Json格式的401回應；或者，若請求非Ajax的話，<br/>
則將使用者重導到名稱為login的命名路由。<br/>

該行為被定義在app/Exceptions/Handler.php的unauthenticated方法，開發者可自行更改預設的行為<br/>
```
use Illuminate\Auth\AuthenticationException;

protected function unauthenticated($request, AuthenticationException $exception)
{
    return $request->expectsJson()
                ? response()->json(['message' => $exception->getMessage()], 401)
                : redirect()->guest(route('login'));
}
```

### 指定守衛
為中介層auth指定要使用哪一個守衛來處理用戶的認證，使用的關鍵字必須
是設定檔auth.php內的守衛陣列鍵值之一
```
public function __construct()
{
    $this->middleware('auth:api');
}
```

### 登入限制
該功能會限制使用者在多次認證失敗後，將會有長達一分鐘的時間無法登入，<br/>
並以用戶名/電子郵件及其所在位置IP作為登入限制的綁定。<br/>
該功能的相關實作被定義在Illuminate\Foundation\Auth\ThrottlesLogins這trait裡面，<br/>
如果使用的是內建的LoginController，則該trait預設就會載入。<br/>

> ## 手動建立使用者認證
當你選擇不要使用Laravel內建的認證程式時，可以選擇刪除這些控制器，並自行<br/>
使用Laravel相關的認證類別來重新實作用戶認證。<br/>

實作時，我們需透過facade來使用Laravel的認證服務。首先，先看一下Auth的認證處理方法attempt
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // Authentication passed...
            return redirect()->intended('dashboard');
        }
    }
}
```
attempt接受以鍵值對陣列作為第一個參數，該參數數值被用來在資料表中進行查詢。<br/>
以上例來說，會先以查詢email欄位，如果用戶存在，則再進行下一步的密碼雜湊比對，<br/>
比對成功就回傳true，反之false。<br/>
另外，有一點要注意的是，傳遞進去的密碼並無須手動雜湊加密，因為框架會自行於比對前自動處理掉。<br/>

認證成功後，再用重導向器的intended方法將使用者重導向到他們原本想要訪問的頁面，<br/>
並在中介層auth攔截請求前被執行。倘若訪問的頁面不存在，就會改回傳設定好的備用頁面。

### 指定額外條件
除了帳號密碼以外，我們也可以傳進要額外認證的欄位，比如說我們希望驗證使用者的active旗標是否為1
```
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // The user is active, not suspended, and exists.
}
```

### 存取特定的守衛實例
在程式中可以使用Auth facade的guard方法指定其他的守衛，這方便我們在應用程式的<br/>
不同地方使用完全不同的認證模型與使用者表。
```
if (Auth::guard('admin')->attempt($credentials)) {
    //
}
```

### 登出應用程式
可以使用Auth facade的logout方法執行登出，其會從session中清除掉使用者的所有驗證資訊。
```
Auth::logout();
```

### 記住使用者
如果想要提供記住我這類的功能，可以在使用attempt方法時傳遞第二參數給該方法，<br/>
當參數值為真時，使用者驗證資料就會被永久保存直到登出。當然，使用者資料表必須<br/>
含有remember_token欄位以保存remember me的token。
```
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```
PS. 如果使用的是內建的LoginController，則記住我的功能都已經由對應的trait實現了<br/>

使用Auth facade 的 viaRemember方法來判斷使用者是否已經被記住了
```
if (Auth::viaRemember()) {
    //
}
```

### 其它可使用的認證方法
#### 用使用者實例做認證
對有實作Illuminate\Contracts\Auth\Authenticatable介面契約的使用者，可將該物件使用在<br/>
Auth facade 的 login方法以執行驗證動作。例如，預設的App/User模型就已經實作了該介面。<br/>
```
Auth::login($user);

// Login and "remember" the given user...
Auth::login($user, true);

// 使用了不同的守衛進行驗證
Auth::guard('admin')->login($user);
```
#### 用使用者ID做驗證
直接使用主鍵ID值來進行使用者驗證
```
Auth::loginUsingId(1);

// Login and "remember" the given user...
Auth::loginUsingId(1, true);
```

#### 一次性的使用者認證
使用Auth facade的once來做一次性的登入驗證，該法不會使用session或cookie來保存驗證資料，<br/>
非常適合使用在無狀態的API存取
```
// method once has same para with method attempt
if (Auth::once($credentials)) {
    //
}
```

> ## HTTP基礎認證
一種快速的用戶認證方式，不需要登入頁面，於瀏覽器訪問時，會自動提示輸入驗證資訊。<br/>
Laravel有內建一組中介層auth.basic可供使用，直接套用在路由上即可。<br/>
預設使用email欄位作為用戶名
```
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic');
```

### FastCGI注意事項
使用FastCGI時，Http基礎認證可能無法正常工作，此時需要添加一些rewrite的規則到.htaccess文件
```
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

### 無狀態Http基礎認證
使用Http基礎認證時，若不想在session保存用戶唯一識別的cookie，特別是API存取時，
就需自行定義一個調用onceBasic方法的中介層，如果onceBasic沒有回應，則請求會進一步
傳遞進應用程式內部。
```
<?php

namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```
在路由設定中套用中介層
```
Route::get('api/user', function () {
    // Only authenticated users may enter...
})->middleware('auth.basic.once');
```

### 登出
欲手動登出用戶時，可以使用Auth facade的logout方法，該法會清除掉session中的驗證資訊。
```
use Illuminate\Support\Facades\Auth;

Auth::logout();
```
#### 清除其它裝置上的認證資訊
Laravel提供了一個特別的機制，就是可以登出從其它裝置登入的認證資訊，但會保留當下裝置的登入認證。<br/>
首先，需先在app/Http/Kernel.php內的web中介層群組設定區塊內容中取消<br/>
Illuminate\Session\Middleware\AuthenticateSession這個中介層的註解。
```
'web' => [
    // ...
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    // ...
],
```
接著調用Auth facade的logoutOtherDevices方法，該方法需要用戶的密碼作為參數傳入，<br/>
而該密通常藉由表單來取得。常見使用方式就是再登入驗證成功後，調用該方法以從其它裝置登出
```
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($password);
```

> ## 其它
### 新增客製的守衛
使用Auth Facade的extend方法來擴充可使用的客製守衛，預設支援session與token兩種。<br/>
可直接在內建的AuthServiceProvider中進行初始化。
```
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::extend('jwt', function ($app, $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...，該介面也定義了幾個必須實作的方法
            // $config['provider']即auth.php內guard區塊的provider設定值
            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```
接著，就可以在設定檔auth.php中配置使用自訂的守衛
```
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

#### 閉包守衛
一個最簡單且基於Http請求的認證實作，直接使用閉包快速定義一個守衛，無須額外建立獨立的守衛程式檔。<br/>
同樣地，需於AuthServiceProvider中擴充，使用方法為Auth facade的viaRequest方法，該方法接受兩個參數，<br/>
第一個為自訂的守衛名稱，方便爾後在auth.php中進行配置；<br/>
第二個則是定義認證邏輯的閉包。<br/>
```
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Register any application authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('custom-token', function ($request) {
        return User::where('token', $request->token)->first();
    });
}
```
設定檔auth.php中配置使用閉包守衛
```
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

### 新增自訂的提供者
若您並不是使用傳統的關聯式資料庫來儲存你的用戶資料，那就勢必得自行客製提供者，
並且需在AuthServiceProvider中使用Auth Facade的provider方法來進行擴充。
```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use App\Extensions\RiakUserProvider;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```
在auth.php設定檔配置使用客製的提供者
```
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],

// 在您使用的守衛配置內指定使用客製的提供者users
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

### 提供者兩個重要的介面
#### User Provider Contract (Illuminate\Contracts\Auth\UserProvider)
此介面應該由驅動類別來實作，即auth.php的provider區塊內定義的driver。以eloquent driver為例，<br/>
其對應的驅動類別就是**Illuminate\Auth\EloquentUserProvider**<br/>
實作該介面主要是為了從永久性儲存系統取得並返回Illuminate\Contracts\Auth\Authenticatable實例。<br/>
藉由UserProvider與Authenticatable兩個介面，無論使用者資料如何被儲存或用了何種類型的提供者，<br/>
皆不影響Laravel驗證的運行。
```
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider {

    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);

}
```
 - retrieveById<br/>
   該方法接受用戶的主鍵作為參數，如mysql的Autoincrement ID，並返回具相同鍵值的Authenticatable實例<br/>
 - retrieveByToken<br/>
   該方法接受具唯一性的identifier與remember me token兩個參數來檢所並返回匹配的Authenticatable實例<br/>
 - retrieveByCredentials<br/>
   該方法接受於登入時傳遞給 Auth::attempt 方法之陣列型態的認證資料，並使用$credentials['username']到底層的永久性儲存系統進行查詢，<br/>
   取得匹配的使用者。接者返回Authenticatable實例。**注意，該法不應該企圖做任何密碼的驗證或是認證**<br/>
 - updateRememberToken<br/>
   該方法接受新的token值來更新$user的remember_token欄位，而在使用"記住我"登入成功或登出時，就應該指派新的token予以更新<br/>
 - validateCredentials<br/>
   該方法應該使用$credentials來與$user來進行比對認證。舉例來說，可能會使用Hash Facade的check方法來比較$credentials['password']與<br/>
   $user->getAuthPassword()的回傳值，並回傳布林值以表示密碼是否為有效密碼。

#### Authenticatable Contract
此介面通常由Model實作，對應auth.php的provider區塊內的model設定。所以前述的Authenticatable實例，<br/>
指得就是實作了Authenticatable介面的User模型。<br/>
再次提醒，UserProvider 的 retrieveById 和 retrieveByCredentials 方法都需要回傳 Authenticatable 的實例。
```
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable {

    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();

}
```
getAuthIdentifierName回傳使用者表格的主鍵欄位名稱；<br/>
getAuthIdentifier方法回傳使用者的主鍵值；<br/>
getAuthPassword方法回傳使用者雜湊後的密碼。<br/>
這個介面允許認證系統與任何使用者類別合作，無論你使用何種 ORM 或儲存抽象層。<br/>
Laravel的app資料夾中就內建了實作這個介面的 User 類別，我們可以觀察這個類別作為實作時的參考。<br/>

### 相關驗證事件
Laravel提供了數種在驗證過程中的事件，我們可以在EventServiceProvider中設置這些事件的監聽器。
```
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Auth\Events\Registered' => [
        'App\Listeners\LogRegisteredUser',
    ],

    'Illuminate\Auth\Events\Attempting' => [
        'App\Listeners\LogAuthenticationAttempt',
    ],

    'Illuminate\Auth\Events\Authenticated' => [
        'App\Listeners\LogAuthenticated',
    ],

    'Illuminate\Auth\Events\Login' => [
        'App\Listeners\LogSuccessfulLogin',
    ],

    'Illuminate\Auth\Events\Failed' => [
        'App\Listeners\LogFailedLogin',
    ],

    'Illuminate\Auth\Events\Logout' => [
        'App\Listeners\LogSuccessfulLogout',
    ],

    'Illuminate\Auth\Events\Lockout' => [
        'App\Listeners\LogLockout',
    ],

    'Illuminate\Auth\Events\PasswordReset' => [
        'App\Listeners\LogPasswordReset',
    ],
];
```