# Laravle Session

> ~~~
> 由於 HTTP 驅動的應用程式是屬於無狀態的，所以 Session 提供了一種可跨越多個請求來儲取使用者資訊的方法。
> Laravel 內建了直觀且一致的 API 來存取各種 Session 後端，並支援目前熱門的後端驅動，像是 Memcached、Redis
> 和資料庫。
> ~~~

## Session配置
config/session為Laravel Session的配置檔案。預設，Laravel會使用file這個session驅動，然而在正式的線上環境<br/>
通常會考慮使用memcached或redis，以其有更好的Session存取效能。配置檔中，Session driver 設定選項是用來定義<br/>
每個請求所儲存的 Session 資料位置，而 Laravel 則內建幾個馬上可以應用的驅動。
 - file：Session 會被儲存到 storage/framework/sessions 中。
 - cookie：Session 會被儲存到安全且被加密的 Cookie 中。
 - database：Session 會被儲存到關聯資料庫中。
 - memcached / redis：Session 會被儲存到其中一個快速且基於快取的儲存系統中。
 - array：Session 會被儲存到一個 PHP 陣列中且不會被保留。
 > ~~~
 > 陣列式驅動主要被用於測試的時候，並防止 Session 資料被永久儲存。
 > ~~~

## 使用前提
### Database
當你使用了資料庫這個session驅動，就需要建立一個儲存session資料的資料表格。以下是使用 Schema 建立資料表的範例：
```
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->unsignedInteger('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});
```

你可以直接透過Artisan指令來創建該遷移檔案
```
php artisan session:table

php artisan migrate
```

### Redis
在使用redis這個session驅動前，你須先透過composer安裝predis/predis這個套件(~1.0)，也需要在config/database<br/>
中先配置好你的redis。接著，在 session 設定檔中，使用 connection 選項指定 Session 要使用哪一個 Redis 連線。

## 使用Laravel Session

### 取得資料
Laravel 主要提供了兩種方法來使用 Session 資料：
#### 透過request實例
 ```
 <?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}
 ```
上述範例中，是透過在request實例鏈式調用session方法與get方法來取得session資料。get方法中你還可以透過第二參<br/>
數來傳遞預設值，在無法透過指定鍵名取得session資料時，就會返回該預設值。同樣狀況下，如果是用一個閉包函數<br/>
做為第二參數，則返回閉包函數執行後的結果。
```
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

#### 全域的輔助方法 session
你也可以使用輔助方法session來存取session中的資料，如果調用時只傳入一個字串參數，則會回傳session鍵符合該<br/>
字串參數的資料。但當傳入的是一組含有鍵與值的陣列時，行為就會變成是將值存進session中。
```
Route::get('home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

> ~~~
> 不論是透過 HTTP 請求實例，還是使用全域的 session 輔助函式，這兩者之間並無實際上的差異。都能夠透過
>  assertSessionHas 方法在所有測試案例中測試這兩種方法。
> ~~~

#### 取得所有session資料
使用all這個方法將session中的資料全部取出。
```
$data = $request->session()->all();
```

#### 判斷資料是否存在session中
使用has這個方法判斷資料是否存在於session中，若存在返回true，否則返回null。
```
if ($request->session()->has('users')) {
    //
}
```

如果session中可能存有null這類的資料，則可以改用exists方法，exists 方法會在該值存在時回傳  true。
```
if ($request->session()->exists('users')) {
    //
}
```

### 存取資料
要將資料存入session中，通常是透過請求實例調用put方法或使用輔助方法session。
```
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global helper...
session(['key' => 'value']);
```

#### 將 Session 值存入陣列中
push方法可用來將資料存入session中的某個陣列。例如，如果 user.teams 鍵中有一組儲存團隊名稱的陣列資料，你就<br/>
可以將一個新值推入該陣列中，就像下面範例一樣。
```
$request->session()->push('user.teams', 'developers');
```

#### 取出並刪除session資料
使用pull方法來取得資料時，會在資料取出後一併將資料刪除。
```
$value = $request->session()->pull('key', 'default');
```

### 快閃資料
有時你可能希望將資料儲存入到 Session 後，只能在下一個請求前維持有效。你可以使用flash 方法來達到目的。<br/>
而這類的快閃資料主要用於短期的狀態訊息。
```
$request->session()->flash('status', 'Task was successful!');
```

如果你想讓資料在session中多停留幾個請求的時間，你可以使用reflash方法。該方法預設是將**所有快閃資料**多保留<br/>
額外幾個請求的時間，若你只是想針對**特定的快閃資料**，可以使用keep方法。
```
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

### 刪除資料
forget方法可以用來將特定資料從session中刪除，如果要將所有的session資料刪除，可以改用flush方法。
```
$request->session()->forget('key');

$request->session()->flush();
```

### 重新產生Session ID
重新產生 Session ID 通常是為了預防惡意使用者利用 [session fixation](https://github.com/Internaltide/Laradep/blob/master/laratopics/Session.md#session-fixation) 來攻擊你的應用程式。如果使用的是內建的<br/>
LoginController，Laravel就會在認證過程中自動重新產生 Session ID。然而，如果你需要手動重新產生 Session ID，<br/>
可以使用 regenerate 方法。
```
$request->session()->regenerate();
```

## 新增自訂的Session驅動
### 驅動實作
你自定的session驅動必須實作SessionHandlerInterface這個介面，介面包含了幾個必要的方法需要去實作。以一個
基本的 MongoDB 實作大概會像下面這樣：
```
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```
> ~~~
> Laravel並未內建置放客製session驅動程式檔的目錄，你可以自行將其規劃放在任意的目錄下。上述範例中，是
> 示範將其放在Extensions目錄下。
> ~~~

單從字面上來看，可能較難理解每一個方法的目的，以下是針對這幾個方法的簡單描述：
 - open: 該方法通常僅用在基於檔案的儲存系統上。但因為 Laravel 內建就有 file 的session驅動，所以你可以<br/>
   讓這個方法保持為空而幾乎不用寫任何東西。但基於不良的介面設計，PHP 會要求必需實作此方法。

 - close: 該方法的情況與open方法很相近，通常可以忽略，因為多數的session驅動並不需要。

 - read: 該方法必須根據給予的 $sessionId 回傳相關聯且為字串版本類型的 session 資料。在驅動中存取資料都不<br/>
   需要做任何的編碼或序列化的動作，因為 Laravel 會自動幫你完成序列化。

 - write: 該方法會依給予的 $sessionId 將給定的 $data 字串存入永久儲存系統，像是MongoDB、Dynamo等等。<br/>
   同樣地，你不應該執行任何序列化，因為Laravel 都已經為你處理好了。

 - destroy: 該方法用來將與$sessionId關聯的資料從永久儲存系統中移除。

 - gc: 該方法用來刪除給予之 $lifetime 之前的所有資料。$lifetime 是一個 UNIX 的時間戳記，不過在 Memcached<br/>
   和 Redis 這類會自動過期的系統中，可以讓此方法保持為空。

### 註冊驅動
一旦你的驅動程式實作好了，你就可以準備在框架中註冊使用它，要新增額外的session驅動，可以在 Session facade<br/>
上調用 extend 來進行註冊。通常應該在服務提供者的boot方法中調用extend進行註冊，如AppServiceProvider或其它的<br/>
服務提供者類別。
```
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Session::extend('mongo', function ($app) {
            // Return implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```
一但新的驅動被註冊好之後，就可以在config/session設定檔中配置使用該客製驅動。以上面的例子來說，你就可以<br/>
使用配置mongo來使用你客製好的驅動。



## 補充
**Session Fixation**
> ~~~
> 一種session的攻擊方式，攻擊者誘使受害者使用特定的 Session ID 登入網站，而攻擊者就能取得受害者的身分。
> 步驟大致為：
> 1. 攻擊者會先使用自己的帳號登入網站以取得一個有效的 Session ID
> 2. 使用社交工程等手法誘使受害者點選連結，讓受害者使用該 Session ID 登入網站
> 3. 受害者輸入帳號密碼成功登入網站
> 4. 攻擊者使用該 Session ID，操作受害者的帳號
> ~~~