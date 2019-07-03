# Laravel 視圖


## 創建視圖
視圖包含了應用程式所使用的HTML並且將控制器與應用程式與顯示的邏輯區分開來。預設，視圖檔案<br/>
會存放在resources/views目錄下，一個簡單的視圖就像下面所示。
```
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

將上述視圖存成resources/views/**greeting**.blade.php後，就可以透過輔助方法 **view** 來返回試圖內容。
```
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```
在view方法中，第一個參數對應著視圖檔案名稱，第二個參數則是要傳遞進視圖的資料，需使用陣列格式。<br/>
本例中，陣列鍵值為name的資料即可以在視圖中以符合Blade語法的方式將資料嵌入，如 **{{ $name }}** 。<br/>

你也可以將視圖檔案放在resources/views目錄的次目錄下，然後使用點符號 **.** 的串接來代表視圖。例如<br/>
resources/views/admin/profile.blade.php這個視圖為例，我們就可以使用下列方式來返回
```
return view('admin.profile', $data);
```

### 判斷視圖是否存在
```
use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {
    //
}
```

### 取得第一個可用視圖
在一些允許視圖客製的場景中，我們可能會有像下面這樣的控制邏輯存在。<br/>
 - 首先，先判斷是否存在客製視圖文件<br/>
 - 接著，再來決定是使用客製視圖還是預設視圖<br/>

現在可以透過 view facade 的 first方法來處理，其第二個參數依舊為要傳遞的視圖資料，<br/>
第一個參數則為存有一系列視圖名稱的陣列，first方法會依序遍歷陣列直到找到第一個<br/>
存在的視圖後返回。所以以下面為例，當找不到custom.admin視圖就會使用預設的admin視圖。
```
return view()->first(['custom.admin', 'admin'], $data);
```

## 傳遞視圖資料

如同前面例子一樣，透過view輔助方法的第二個參數傳遞視圖資料。
```
return view('greetings', ['name' => 'Victoria']);
```

透過在view方法後面鏈接with方法來帶入視圖資料
```
return view('greeting')->with('name', 'Victoria');
```

### 在所有視圖中共享資料
使用view facade的share方法可以在所有視圖中共享資料。一般用法會在服務提供者內<br/>
的boot方法進行定義。
```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

## 視圖組件(View Composer)
視圖組件是一個在視圖渲染時所調用的類別方法或回調方法，用來定義共享的資料。<br/>
它可以是一個單獨的viewComposer類別也可以是一個簡單的閉包。與view facade share 差異在於<br/>
 - 前者share方法直接定義一組所有視圖共享的資料<br/>
 - 後者composer方法定義視圖與共享資料的綁定關係，換句話說不是所有試圖都可以使用試圖<br/>
 組件定義的資料。<br/>

如果每次渲染時，你都有一組資料須要綁定到視圖時，視圖組件可以幫你將邏輯統一組織到單一位置。<br/>

以下面例子來說，我們在服務提供者內註冊了一個視圖組件，並透過使用view facade來存取底層<br/>
Illuminate\Contracts\View\Factory的介面實作。記住，Laravel並未提供存放試圖組件的目錄，所以<br/>
你必須自行根據需求來創建目錄。例如，App\Http\ViewComposers。

### 視圖組件綁定
```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function boot()
    {
        // Using class based composers...
        View::composer(
            'profile', 'App\Http\ViewComposers\ProfileComposer'
        );

        // Using Closure based composers...
        View::composer('dashboard', function ($view) {
            //
        });
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```
PS. composer方法的第一個參數為Blade視圖名稱，第二個參數為定義的共享資料(ViewComposer或Closure)。

> ~~~
> 記住!! 若你是在另外創建的服務提供者內註冊試圖組件的綁定，記得要將該服務提供者
> 加入到config/app的 providers陣列中。
> ~~~

### 視圖組件定義
每次視圖渲染前，ViewComposer的compose方法都會被Illuminate\View\View實例來調用。<br/>
compose方法則使用$view->with()來綁定資料到視圖。
```
<?php

namespace App\Http\ViewComposers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileComposer
{
    /**
     * The user repository implementation.
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * Create a new profile composer.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // Dependencies automatically resolved by service container...
        $this->users = $users;
    }

    /**
     * Bind data to the view.
     *
     * @param  View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```
PS. 所有視圖組件也是透過服務容器來解析依賴，因此也可以在組件類別的建構式透過型別提示來注入依賴。

### 視圖組件多重綁定
你也可以將一組共享資料一次綁定到多個視圖上。方法一樣，只是原本的Blade視圖名稱需改成<br/>
陣列格式傳遞。
```
View::composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

使用wildcard **\*** 將組件綁定到所有視圖上
```
View::composer('*', function ($view) {
    //
});
```

## 視圖建構器
視圖建構器幾乎和視圖組件運作方式一樣；只是視圖建構器會在視圖初始化後就立刻執行，
不像視圖組件會一直等到視圖即將被渲染時才執行。
要註冊一個建構器，只要使用 creator 方法：
```
View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```
