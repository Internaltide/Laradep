# Laravel Service Container(服務容器)

> Laravel 透過型別提式自動透過容器取得依賴實例然後注入依賴，<br/>
> 開發者不應該直接於應用內部與服務做直接耦合。<br/>
>
> 藉由服務提供者將依賴註冊並綁定到容器中，應用程式再從容器中解析出需要的實例。<br/>
> 各類服務提供者(**供應商**)透過容器向應用程式提供服務；<br/>
> 應用程式(**消費方**)向容器索取所需要的第三方服務，以支援應用領域的實作；<br/>
> 服務容器(**賣場**)即為第三方服務與應用程式之間的橋梁，管理服務並代為提供服務給消費端。

## Binding
將服務綁定到容器內並非必要，因為框架仍可透過PHP的Reflection機制來取得對應的服務類別。<br/>
只是當注入的依賴為介面時，就必須做明確的綁定，因為實作該介面的服務類別可能有好幾個，<br/>
必須明確宣告綁定方式，容器才知道如何解析。<br/><br/>
綁定通常是透過服務提供者來實作，其內部可以透過$this->app取得服務容器實例並使用bind method<br/>
將類別、介面名稱與實例綁定在一起。

### 簡單綁定
下面例子，因為存在次依賴，所以將容器實例$app傳入閉包，以便於內部進行次依賴的解析<br/>
Method: **bind(name,closure)**
```
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```
### 單例綁定
使用此方法綁定的類別，只會被解析一次。<br/>
當應用再次索取該實例時，容器便直接回傳先前的解析的實例<br/>
Method: **singleton(name,closure)**
```
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
```
### 實例綁定
值皆對已存在的物件進行綁定
Method: **instance(name,object)**
```
$api = new HelpSpot\API(new HttpClient);

$this->app->instance('HelpSpot\API', $api);
```
### 原始值綁定
當建構式除了注入依賴外，還注入其它非類之基本值時(如整數)，即可使用此方式<br/>
Method: **when(name)->need(variable)->give(variable value)**
```
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
```
### 綁定介面到實作
Method: **bind(interface,implementation)**
```
$this->app->bind(
    'App\Contracts\EventPusher',         // Interface
    'App\Services\RedisEventPusher'      // Implemaentation
);
```
### 情境綁定
在先前的綁定機制都是一對一，但當你需要在不同位置綁定同介面的不同實作時，<br/>
前面的方法就不敷使用，必須改用下列方式<br/>
Method: **when(name)->need(name)->give(closure)**
```
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when(VideoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```
### 擴充綁定
提供對已解析服務進行額外的擴充、設定或裝飾<br/>
Method: **extend(service,closure)**
```
$this->app->extend(Service::class, function($service) {
    return new DecoratedService($service);
});
```
### 標記
當需對一系列同類型的類別進行解析時，可以先執行標記再執行一次性解析<br/>

#### 標記方式
Method: **tag(nameArray,tag name)**
```
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```
#### 解析方式
Method: **tagged(tag name)**
```
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

## Resolving
### 使用make methd
```
$api = $this->app->make('HelpSpot\API');
```
### 使用輔助方法resolve
有些地方可能無法取得容器$app，這時可以改用全域輔助方法
```
$api = resolve('HelpSpot\API');
```
### 使用makeWith注入容器無法解析的依賴
```
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```
### 自動注入
透過在建構式使用型別提示，PHP Reflection會自動偵測對應的服務類別，解析後並注入
```
public function __construct(UserRepository $users)
{
    $this->users = $users;
}
```

## Container Events
每一次的容器解析都會觸發事件，所以可透過resolving方法監聽事件並進一步定義觸發後行為
```
$this->app->resolving(function ($object, $app) {
    // 當容器解析任何型別的物件時會被呼叫...
});

$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
    // 當容器解析「HelpSpot\API」型別的物件時會被呼叫...
});
```

## Resolve Container Instance
因為服務容器實作PSR-11容器標準介面，所以在型別提示時可以利用來取得容器實例
```
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```
注意，一般解析時，若發現提示字符並未確實綁定到容器，框架會自行透過PHP Reflection處理後續。<br/>
但倘若使用的式get方法時，則會直接丟出例外。
