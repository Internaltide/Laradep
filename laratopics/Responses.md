# Laravel Http Responses

>

## 建立回應
#### 字串與陣列
所有路由與控制器都該回傳一個回應到瀏覽器。Laravel 提供了數個方式來回傳回應，<br/>
而最常見的就是回傳一個字串型態的資訊。回傳字串資訊時並無須額外的處理，Laravel<br/>
會自動將字串轉換成完整的Http回應。
```
Route::get('/', function () {
    return 'Hello World';
});
```
若回傳的是陣列資料，Laravel則自動轉換為JSON型態的回應
```
Route::get('/', function () {
    return [1, 2, 3];
});
```
> ~~~
> 若回傳的是Eloquent集合，則Laravel也會自動轉換成JSON回應
> ~~~

#### 回應物件
通常來說，你不會只從路由中回傳簡單的字串或陣列。取而代之地，反而是回傳完整的 Illuminate\Http\Response 實例<br/>
或視圖。若是回傳完整的Response實例，就可自訂自己的Http狀態碼及標頭。<br/>
Response繼承自Symfony\Component\HttpFoundation\Response，擁有一系列的方法可以方便地創建Http Response。
```
Route::get('home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

### 附加標頭到回應
多數的Response實例方法都是可以鏈接的，這讓我們可以更簡潔地創建回應。例如，在回應傳遞到使用者端時，<br/>
使用header方法可將一群標頭附加到回應之中。
```
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            ->header('X-Header-Two', 'Header Value');
```
亦可使用withHeaders方法將陣列中的每個標頭都加到回應中
```
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
```

### 附加Cookie到回應
使用cookie這個方法可以輕易地將Cookie資料附加到回應中。
```
return response($content)
                ->header('Content-Type', $type)
                ->cookie('name', 'value', $minutes);
```
cookie方法也接受一些不常用的參數。通常這些參數會和 PHP 原生的 setcookie 方法的參數具有相同的意義和目的。
```
->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
```
或者也可使用Cookie Facade的queue來附加一組 Cookies 到即將輸出的回應上。queue 方法接受一個  Cookie 實例或<br/>
建立 Cookie 實例所需的參數。在發送回應到瀏覽器之前，這些 cookie 會被附加到即將輸出的回應上。
```
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

### Cookie與加密
預設，所有Laravel產生的Cookie會被加密與簽署，使它無法在客戶端被串改或讀取。<br/>
如果你想要停用應用程式為Cookie 加密的行為，你可以利用 App\Http\Middleware\EncryptCookies<br/>
中介層的 $except 屬性。
```
/**
 * The names of the cookies that should not be encrypted.
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```

## 重導向
重導回應是一個Illuminate\Http\RedirectResponse實例，其包含了重定向所需的一些標頭。產生這類實例<br/>
的最簡單方式為使用全域的輔助方法redirect。
```
Route::get('dashboard', function () {
    return redirect('home/dashboard');
});
```
有時可能會希望將使用者重導到先前使用的頁面，像是在送出的表單是無效的時候。你可以使用全域的 back 輔助函式。<br/>
由於這個功能會用到 Session，需先確認使用 back 函式的路由有用到 web 中介層群組或套用到所有 Session 中介層。
```
Route::post('user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

### 重導至命名路由
當你調用redirect輔助方法卻未傳遞參數時，該方法就會回傳一個Illuminate\Routing\Redirector實例。<br/>
此時你就可以調用Illuminate\Routing\Redirector上頭的方法，例如鏈結調用route方法來重導至命名的路由。
```
return redirect()->route('login');
```
當路由含有路由參數時，可藉由route方法的第二個參數來傳遞
```
// For a route with the following URI: profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

#### 透過Eloquent模型填充參數
如果你正要重導到路由模型綁定類型的路由，可以直接傳入模型本身。模型ID值會被自動取出
```
// For a route with the following URI: profile/{id}

return redirect()->route('profile', [$user]);
```
路由模型綁定使用的綁定欄位為ID欄位，你應該在Eloquent模型上覆寫getRouteKey方法，<br/>
改回傳實際檔定用欄位的值。<br/>
( 相對於getRouteKeyName覆寫的是綁定用的**欄位名稱**；getRouteKey則是覆寫要取出的**路由參數值** )
```
/**
 * Get the value of the model's route key.
 *
 * @return mixed
 */
public function getRouteKey()
{
    return $this->slug;
}
```

### 重導至控制器行為
你也可以重導到控制器行為，使用的方法為透過Illuminate\Routing\Redirector實例調用action。
```
// 您無須定義完整的控制器命名空間，因為 RouteServiceProvider 預設已設立了控制器的基礎命名空間
return redirect()->action('HomeController@index');
```
如果含有路由參數，則藉由第二個參數傳遞
```
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
```

### 重導至外部網域
有時會需要重導至網域外部的應用程式，可以透過away方法來實現，Laravel會創建一個無須額外的URL編碼、驗證或認證的<br/>
RedirectResponse實例。
```
// away($path, $status = 302, $headers = [])，away方法接受的是完整的URL
return redirect()->away('https://www.google.com');
```

### 重導並快閃資料到session
跳轉到新的位置及快閃資料到Session常常是需要同時執行的。典型狀況是一個成功的行為被執行後，將成功訊息<br/>
快閃到session中。
```
Route::post('user/profile', function () {
    // Update the user's profile...

    return redirect('dashboard')->with('status', 'Profile updated!');
});
```
當使用者成功被跳轉後，你就可以將先前快閃進session的資料顯示到Blade視圖中
```
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

## 其它回應類型
Response輔助方法還可以建立多種不同的回應實例，當調用response方法卻未傳遞參數時，實作<br/>
Illuminate\Contracts\Routing\ResponseFactory這個介面的實例就會被回傳，該實例提供了好幾個方法<br/>
可以產生其它類型回應。
### 視圖回應
如果你需要控制狀態碼與標頭而且僅需回傳視圖，可以鏈結調用view方法來達成。
```
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```
當然，如果不需要控制狀態碼與標頭，則可改用全域輔助方法view。

### JSON回應
使用json方法來產生JSON回應，該方法會自動設定好Content-Type標頭為application/json並接受一個<br/>
陣列參數，內部會調用PHP原生的json_encode來將之轉換為JSON格式資料。
```
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```
如果需要的是一個JSONP回應，你應合併調用json方法與withCallback方法，<br/>
withCallback方法會替JSON數據附加上Javascript的封裝，而添加的Javascript部分就是所謂的padding。<br/>
一般形式會類似callback( {"id":123, "name":"Sala"} )
```
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```
[What is JSONP?](https://github.com/Internaltide/Laradep/blob/master/laratopics/Responses.md#JSONP)

### 檔案下載
使用download方法可以產生一個回應強制使用者的瀏覽器下載檔案。download方法接受一個檔案名稱作為<br/>
該方法的第二個參數，這可以定義下載檔案的使用者所看到的檔案名稱。最後，你可以將一組HTTP header<br/>
的陣列資料作為第三個參數傳入到該方法。
```
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);

return response()->download($pathToFile)->deleteFileAfterSend(true);
```
> ~~~
> 注意!! 管理檔案下載的套件 Symfony HttpFoundation會要求被下載的檔案名稱必須為 ASCII。
> ~~~

#### 串流下載
有時候你不會希望將操作完後得到資料寫入檔案，而是直接將這類字串回應轉換成可下載的回應。此時，<br/>
你可以改用streamDownload方法來。該方法可接受一個回調用的閉包函數、檔名及可選且存有標頭資料的陣列。
```
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

### 檔案回應
file方法則用來展示一個檔案的內容，像是圖檔或PDF，它們會直接被顯示在User的瀏覽器上頭，而不是<br/>
強制下載。該法接受檔案路徑做為第一個參數及存有標頭資料的陣列做為第二個參數。
```
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## 回應巨集
如果你想要客製一個可以在各種路由或控制器重用的回應，可以使用Response Facade的micro方法來達成。<br/>
常見的是在服務提供者內調用macro來客製自己的回應。
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Response;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * Register the application's response macros.
     *
     * @return void
     */
    public function boot()
    {
        Response::macro('caps', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```
macro方法接受一個回應的名稱作為它的第一個參數，並將閉包作為它的第二個參數。當從一個ResponseFactory實作或<br/> response 輔助函式中呼叫巨集名稱時，該巨集的閉包會被執行。
```
return response()->caps('foo');
```

>
> ## JSOP
>
> JSONP是JSON with Padding的縮寫，一種在JSON數據基礎上附加Javascript的封裝訊息，一般形式如下：<br/>
> **callback(** {"id":123, "name":"sala"} **)**<br/>
>
> 其中，添加的Javascript部分即稱為**padding**，將JSON數據做為指定函數的參數，就能將JSON數據<br/>
> 放進函數內部處理。因此透過HTML中的script元素將JSOP數據載入後，就可以調用回調函數並於其內<br/>
> 部操作JSON數據。注意!! 回調函數必須事先在讀取JSOP數據的script元素所在頁面中指定好。如下：<br/>
> &lt;script src="https://api.whatsoft.com/v1/users?callback=callback"&gt;<br/>
>&lt;script src="https://api.whatsoft.com/v1/users?callback=cbfunc"&gt;<br/>
>
> **為何要這麼使用？**<br/>
> 這是為了解決因同源政策導致無法跨域訪問的問題。典型的XHTTPRequest，因為安全性問題，<br/>
> 會受到同源政策的限制 ，使其只能存取同一個網域內的資源，導致諸多不方便。為了突破這限制，<br/>
> 遂使用不在同源政策限制範疇內的script元素，利用其載入JSONP來做跨域的訪問。<br/>
>
> **JQuery對JSONP的支援**<br/>
> 通過在URL的查詢參數值後面附加符號 **?** ，JQuery便能識別出這是來自JSONP的訪問，然後<br/>
> 自動產生回調函數，並設置該函數名進行處理<br/>
> ~~~
> <script>
>     (function(){
>         var api = "https://api.whatsoft.com/v1/users/123?callback=?";
>         $.getJSON( api, function(data){
>             // 操作data這個JSON數據
>         });
>     })();
> </script>
> ~~~
>