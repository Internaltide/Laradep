# Laravel URL Generation

> Laravel 提供了好幾個方法來協助你產生應用程式的URL。當然，其主要是用在<br/>
> 於協助建立模板或API響應的連結。或者，它也可用來產生跳轉到應用程式其他<br/>
> 位置的重定向連結。<br/>

## 基礎
### 建立基礎URL
輔助方法url可以用來建立應用程式中任意一種URL，並會根據請求的內容自動套用正確的主機名稱與協定(Http or Https)。
```
$post = App\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

### 存取當下之URL
調用輔助方法url時，若沒有給定路徑，則會直接返回一個Illuminate\Routing\UrlGenerator實例供你存取當下URL的相關資訊。
```
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();

// Get the full URL for the previous request...
echo url()->previous();
```

所有相關的url輔助方法你也可以透過URL facade來進行存取。
```
use Illuminate\Support\Facades\URL;

echo URL::current();
```

## 命名路由之URL
輔助方法route可以用來建立命名路由對應的URL。這種方式可以更方便你建立URL而無須直接<br/>
與實際的路由設定產生耦合。這樣一來，當你的路由定義產生異動時，你也就無須更動你建立<br/>
URL的程式碼。<br/>

以下面這個路由定義為例：
```
Route::get('/post/{post}', function () {
    //
})->name('post.show');
```

使用輔助方法route來建立存取該路由的URL
```
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```
```
// 因為通常會使用模型主鍵值來產生URL，所以也可直接傳遞Eloquent模型進去，它會自動擷取模型主鍵的值
echo route('post.show', ['post' => $post]);
```

### URL簽章
Laravel讓你可以輕鬆地**為命名路由建立**具簽章形式的URL，該類型的網址會在其Query String後面<br/>
附加上一串雜湊字串，可供Laravel檢查網址在創建後是否曾被異動過。這對於那些需要額外保護<br/>
的公開網址特別有用。例如你想透過郵件傳送一個取消訂閱的連結給客戶，你就可以透過url facade的<br/>
signedRoute方法來建立簽章式的URL。
```
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

你也可以創建一個臨時性的簽章URL，只要改用temporarySignedRoute方法即可
```
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

### 驗證簽章網址
#### 使用請求實例的hasValidSignature方法
要驗證請求是否具有有效的簽章，你可以使用該請求實例來調用hasValidSignature這個方法進行檢查
```
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

#### 使用中介層
你也可以使用中介層來處理簽章的驗證，Laravel預設就內建了可用的簽章驗證中介層，Illuminate\Routing\Middleware\ValidateSignature，<br/>
使用前，記得將該中介層加入到HTTP kernel 的 routeMiddleware陣列中，才能在於路由中套用<br/>
該中介層，否則將出現找不到中介層的錯誤。
```
/**
 * The application's route middleware.
 *
 * These middleware may be assigned to groups or used individually.
 *
 * @var array
 */
protected $routeMiddleware = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
```

一旦你將該中介層註冊到Http kernel後，就可以在路由中套用該中介層檢查簽章。<br/>
當檢查出請求中並未包含有效簽章時，中介層會回應一個403錯誤。
```
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

## 對應控制器行為之URL
你可以使用action這個輔助方法來生成特定控制器行為的URL，你無須傳地完整的命名空間，<br/>
因為預設就已經綁定了命名空間前綴App\Http\Controllers，所以只要傳遞相對的命名空間即可。
```
$url = action('HomeController@index');
```

如果控制器方法接受路由參數，你可以透過第二參數來把它傳遞到action方法中。
```
$url = action('UserController@profile', ['id' => 1]);
```

## 路由參數預設值
一些應用程式的多個請求可能都需要同一個路由參數值，例如辨識語系用的路由參數{locale}。<br/>
這會使得每一項路由的URL產生都要透過輔助函數傳遞相同的參數值，會是一項很笨重的瑣事。
```
Route::get('/{locale}/posts', function () {
    //
})->name('post.index');
```

你可透過在中介層中使用URL::defaults定義該路由參數的預設值。如此一來，該請求的生命週期內<br/>
都將套用該預設值。一旦定義了路由參數預設值，在應用輔助函數產生URL時就可以不用再傳遞一<br/>
次該參數值。
```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    public function handle($request, Closure $next)
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```
