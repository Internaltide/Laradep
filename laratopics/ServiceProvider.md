# Laravel Service Provider(服務提供者)

> 應用程式及所有Laravel核心服務都是藉由服務提供者發起的。包含了<br/>
> 容器綁定、事件監聽器、中介層乃至於路由等的註冊。所以服務提供者實為<br/>
> 設定應用程式之核心所在。<br/>
>
> 在Laravel設定檔目錄下的app.php中，providers陣列即用來定義應用程式會用到的<br/>
> 所有服務提供者類別。

## 編寫服務提供者
所有服務提供者都必須繼承自 Illuminate\Support\ServiceProvider 基底類別，<br/>
且須實作register方法，再視需要實作boot方法，大部分的服務提供者實作都會包含該兩項方法。<br/>
而在你的服務提供者中的任何方法，永遠可以使用 $app 屬性來存取服務容器。

### 透過指令產生provider程式檔
```
php artisan make:provider RiakServiceProvider
```

### 實作Register Mehod(Required)
主要實作如何將事物綁定至容器中，永遠別嘗試在這個方法註冊事件監聽器、路由或實作<br/>
其他功能，以避免不可預期的情形發生。
```
public function register()
{
    $this->app->singleton(Connection::class, function ($app) {
        return new Connection(config('riak'));
    });
}
```

### 實作Boot Method(Optional)
若有需要對物件進行初始化或使用其它相依物件時，可以將相關邏輯寫在boot method中。<br/>
另外，由於boot設計成在**所有**其他的服務提供者被註冊後才被呼叫，這意味著在此方法中<br/>
你可以在boot使用型別提示來取得已註冊好的服務，而不會產生尚未綁定的錯誤。
```
// 如此例，composer service的視圖註冊便是放在boot()中來實作
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
}
```

### 類別屬性bindings與singletons
當Provider存在大量simple binding時，手動定義每個綁定也是一項挺麻煩的瑣事。<br/>
此時可以利用bindings跟singletons兩個屬性，將綁定的關係放於此兩個陣列當中。<br/>
當該Provider被載入時，就會自動根據這兩個屬性的內容來進行綁定。
```
<?php

namespace App\Providers;

use App\Contracts\ServerProvider;
use App\Contracts\DowntimeNotifier;
use Illuminate\Support\ServiceProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\DigitalOceanServerProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
    ];
}
```

## 註冊服務提供者
將你的服務提供者加到config/app.php內的provider陣列中，預設，<br/>
Laravel的核心服務都會列於此陣列當中，我們只需要將新的疊加在陣列<br/>
尾端即可。
```
'providers' => [
    /*
     * Laravel Framework Service Providers
     */
    Illuminate\Auth\AuthServiceProvider::class,
    ...

    /*
     * Package Service Providers
     */
    ...

    /*
     * Application Service Providers
     */
    App\Providers\AppServiceProvider::class
    ...,
],
```

## 延遲服務提供者
選擇延緩Provider的註冊綁定，可以有效提升應用程式的效能，因為不用再每個請求都將其載入。<br/>
要延緩提供者載入，只需將 defer 屬性設定為 true，並定義一個 provides 方法。<br/>
provides 方法會回傳提供者所註冊的服務容器綁定。
```
class RiakServiceProvider extends ServiceProvider
{
    protected $defer = true;

    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    public function provides()
    {
        return [Connection::class];
    }

}
```