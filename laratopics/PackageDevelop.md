# Laravel Package Development

> ~~~
> Package 套件是為 Laravel 新增功能的主要方式。就像需用來處理時間的 Carbon 或像是 Behat 這種測試框架，
> 透過開發新套件就是一個相當好的方式。
> 當然，套件有相當多形式，有些套件是獨立套件，也就是可以執行在任何 PHP 框架上。像 Carbon 和 Behat 就是
> 獨立的套件。而這些套件都可以被 Laravel 使用，只需要在 composer.json 檔案中引入它們即可。
> 另一方面，一些套件則是專門用在 Laravel。它們可以有路由、控制器、視圖和設定等用來強化 Laravel 應用。
> 這份指南裡則以開發 Laravel 專屬套件為主要目標進行說明。
> ~~~

## FACADES 注意事項
撰寫 Laravel 應用程式時，不論你是使用 Contract 或 Facade 都不會造成太大的差別，因為它們在本質上都提供了<br/>
相同程度的可測試性。然而在撰寫套件時，套件通常會無法使用 Laravel 的測試輔助函式。如果你想要撰寫套件<br/>
的測試就像在 Laravel 的環境中，可以使用 Orchestral Testbench 套件。

## 套件 Discovery
在Laravel 應用程式內的config/app.php這個設定檔中，providers 選項被用來定義了該被 Laravel 載入的服務提供者<br/>
清單。當有人在安裝你的套件時，你通常會想要你的服務提供者能被包含到這個清單中，而不是要求使用者手動<br/>
新增你的服務提供者到該清單上。你可以在套件中 composer.json 的 extra 部分來定義該提供者。除了服務提供者，<br/>
你也可以列出想要註冊的任何 facades。
```
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```
而一旦你的套件配置了Discovery，Laravel 會在套件安裝的時候自動註冊該套件的服務提供者與 Facade，這可為你的<br/>
套件使用者打造更為便利的安裝體驗。

### 選擇退出套件 Discovery
如果你是套件的使用者，當你想停用套件的發現功能時，可以將該套件名稱列在 composer.json 裡 extra 區塊內的<br/>
dont-discover 配置清單中。
```
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

你也可以在應用程式的 **dont-discover** 直接使用 **\*** 字元來為所有套件停用Discovery功能
```
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

## 服務提供者
服務提供者是套件與 Laravel 之間的連接橋梁。服務提供者負責將事物綁定到 Laravel 的服務容器中，並告訴<br/>
Laravel 要到何處載入套件的資源，像是視圖、設定和本地化檔案等。<br/>

服務提供者繼承自 Illuminate\Support\ServiceProvider 並包含了兩個重要的方法 register 和 boot。基底的服務提<br/>
供者類別會放在 illuminate/support Composer 套件中，你必須將其新增到自己的套件依賴中。

## 資源
### 設定
一般來說，你必須要將你的套件設定檔發佈到應用程式的config目錄下，這能夠讓套件的使用者可以輕鬆地覆蓋<br/>
套件的預設設定。而為了要讓套件設定檔可以被發佈，你必須在服務提供者的 boot 方法內調用 publish 方法。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ]);
}
```
現在，當套件使用者執行了指令 vendor:publish，檔案就會被複製到指定的發佈位置。當然一經發佈的設定檔，<br/>
它的值就可以像任何其它設定檔案一樣被存取。
```
$value = config('courier.option');
```
> ~~~
> 注意!! 你不應該在設定檔中設定閉包。因為使用者執行 Artisan 的 config:cache指令時，它們並無法如預期地被
> 進行序列化。
> ~~~

#### 預設的套件設定
你也可以將自己的套件設定檔與應用程式中已發佈的副本進行合併。這可讓使用者在已發佈副本上只定義實際<br/>
想要覆寫的選項。若要合併設定，請在服務提供者的 register 方法中調用 mergeConfigFrom 方法。
```
/**
 * Register bindings in the container.
 *
 * @return void
 */
public function register()
{
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
```
PS. mergeConfigFrom 方法只會合併設定陣列的第一層。如果使用者定義了多維的設定陣列，則遺失的選項將不會被合併。

### 套件路由
如果套件包含了路由設置，你可以使用 loadRoutesFrom 方法來載入路由設定。該方法會自動判斷應用程式的路由<br/>
是否已被快取，如果路由被快取的話，就不會載入路由檔案。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadRoutesFrom(__DIR__.'/routes.php');
}
```

### 套件遷移
如果套件包含了資料庫遷移的設置，你應該使用 loadMigrationsFrom 方法告知 Laravel 如何來載入他它們。該方法<br/>
接受套件的遷移檔路徑，並作為該方法的唯一參數。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
}
```
當套件的遷移檔被註冊，它們就會在操作 php artisan migrate 指令後自動被執行。你並不需要將它們導出到應用程式<br/>
主要的遷移檔目錄 database/migrate。

### 套件翻譯
如果套件包含了翻譯檔案，你可以使用 loadTranslationsFrom 方法來載入它們。假定你的套件名稱叫 **courier**，<br/>
那你應該新增下列內容到你的服務提供者的 boot 方法。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

另外，套件語系會使用像是 **package::file.Line** 這樣的語法慣例來引用語系翻譯。所以若要從 messages 翻譯檔<br/>
中載入 courier 套件的 welcome 字詞翻譯，可以參照下方範例。
```
echo trans('courier::messages.welcome');
```

#### 發佈翻譯檔
如果你想要將翻譯檔發佈到應用程式的 resources/lang/vendor 目錄下，你必須調用服務提供者的 publishes 方法。<br/>
publishes 方法接受一組包含套件路徑的陣列和他們想要發佈的目的地位置。例如，要為 courier 套件發佈翻譯檔案，<br/>
你可以參考下列內容。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

    $this->publishes([
        __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
    ]);
}
```
現在，當套件使用者執行了 vendor:publish 指令後，套件翻譯檔就會被發佈到指定的位置。

### 套件視圖
為了註冊套件視圖到 Laravel ，你需要告訴 Laravel 視圖的位置在哪裡，而你可使用服務提供者的 loadViewsFrom <br/>
方法。該方法接受兩個參數，一是視圖模板路徑，一是套件名稱。例如，套件名稱為 courier，你就可以在服務提<br/>
供者的 boot 方法中新增下列內容。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
}
```

另外，同翻譯檔一樣，取用視圖也有其語法慣例 **package::view**。一但套件視圖在服務提供者進行了註冊，<br/>
要從套件 courier 取得 admin 視圖時，就可以參考下列範例。
```
Route::get('admin', function () {
    return view('courier::admin');
});
```

#### 覆寫套件視圖
當你使用 loadViewsFrom 方法時，Laravel 實際會為你的視圖註冊兩個位置。這兩個位置則包含了應用程式的<br/>
resources/views/vendor 目錄及你所指定的目錄。以套件 courier 為例，Laravel 會先檢查開發者在 resources/views/vendor/courier 中<br/>
是否含有自訂的視圖；如果不存在自訂版本，才會在到使用者指定的視圖目錄找尋套件視圖。這樣的設計讓<br/>
套件使用者更容易的自訂或覆寫你預設的套件視圖。

#### 發佈套件視圖
如果你想要讓套件視圖發佈到應用程式的 resources/views/vendor 目錄，可以使用服務提供者的 publishes 方法。<br/>
publishes 方法接受一組包含視圖路徑的陣列和它們想要發佈的位置。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
    ]);
}
```
現在，你的套件視圖會在使用者執行 vendor:publish 指令後被複製一份到指定的發佈位置。

## 套件指令
如果要註冊套件自己的 Artisan 指令到Laravel，請使用 commands 方法。該指令期望一組包含指令類別名稱的陣列<br/>
作為其參數。一旦註冊好指令，你就能使用 Artisan CLI 來執行它們。
```
/**
 * Bootstrap the application services.
 *
 * @return void
 */
public function boot()
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            FooCommand::class,
            BarCommand::class,
        ]);
    }
}
```

## 公開資源
你的套件可能擁有像是 JavaScript、CSS 或 images 這類的資源檔。若要將它們發佈到應用程式的 public 目錄下，需<br/>
要使用服務提供者的 publishes 方法。在下面的範例中，我們還會新增一個名為 public 的資源群組標籤，這可以被<br/>
用於發佈特定相關的資源群組。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/assets' => public_path('vendor/courier'),
    ], 'public');
}
```

現在，你的套件使用這在執行了 vendor:publish 指令後，你的資源就會被複製一份到指定的發佈位置。而由於你通常<br/>
會需要在更新套件時覆寫現有資源，你可以使用 --force 選項。
```
php artisan vendor:publish --tag=public --force
```

## 發佈檔案群組
你可能想要發佈特定的套件資源群組和其他個別資源。例如，你可能想讓套件使用者只發佈套件設定檔，而無須<br/>
強制發佈套件資源。你可以在套件的服務提供者中調用 publishes 方法時透過指定標籤來做到這一點。舉例來說，<br/>
先讓我們在服務提供者的 boot 方法內利用標籤來定義兩個資源發佈群組。
```
/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'config');

    $this->publishes([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'migrations');
}
```

接著，套件的使用者就能在執行 vendor:publish 指令時依照指定的標籤來個別發佈這些資源群組。
```
// 發佈設定檔相關資源
php artisan vendor:publish --tag=config
```
