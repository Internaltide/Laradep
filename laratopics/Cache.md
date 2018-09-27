# Laravel Cache


## 配置方式
config/cache為Laravel快取的配置檔，在其中可以定義預設要使用的快取驅動。預設是使用檔案快取驅動，<br/>
會將序列化後的快取物件儲存到文件中，只是在較大的應用程式中，還是建議將快取儲存到Laravel也支援的<br/>
一些流行快取系統上，如memcached跟redis。您也可以同時配置使用多個緩存系統。

### 快取前提條件
#### Database
當使用的快取驅動是database時，需要建立一張表來儲存快取資料，下面是資料結構的範例
```
Schema::create('cache', function ($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});
```
也可以直接透過Artisan指令建立快取資料表
```
php artisan cache:table
```
#### Memcached
使用memcached時，你的PHP必須裝有Memcached PECL package的延伸函式庫。<br/>
你可以在config/cache中定義所有的memached服務主機。
```
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],
```
其中，host配置項也可以使用Memcached的UNIX Socket路徑。若使用socket這類的溝通方式，則需要將port這一項
設置為0
```
'memcached' => [
    [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
        'weight' => 100
    ],
],
```

#### Redis
在使用Redis驅動以前，需先透過Composer安裝predis/predis package (~1.0)這個套件<br/>
或者，在PHP裡安裝PhpRedis PECL package的延伸函式庫。<br/>
**配置範例**
```
'redis' => [

    'client' => 'predis',

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
    ],

],
```
PS. 每一組Redis配置，都至少需要名稱、host及port。(更多的配置方式詳見[Redis 說明文件](https://github.com/Internaltide/Laradep/blob/master/laratopics/Redis.md))

## 使用方式
### 獲取快取實例
Laravel是透過 **Illuminate\Contracts\Cache\Factory** 與 **Illuminate\Contracts\Cache\Repository** 來提供<br/>
存取快取服務的。Factory契約用來實現存取各類快取驅動；Repository則是典型的快取驅動實作，會隨著快取設定<br/>
檔而變化。

當然，你也可能需要使用到Cache Facade，該Facade提供了方便且簡潔的方法去存取已按契約實作的底層類別。

### 存取多個快取實作
透過store方法可以存取多個快取儲存體，傳遞到store方法內的參數必須是config/cache設定檔中stores陣列裡的其中之一。
```
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 10);
```

### 快取操作
### 從快取中取得資料
get方法是用來從快取儲存體裡取得特定快取資料的，如果資料不存在，則會回傳null。當然，如果您希望<br/>
有預設值，可以透過方法的第二參數，指定當資料不存在時要回傳的預設值。
```
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```
指定預設值時，除了實際的值外，也可以傳遞第一個閉包，而閉包裡頭的Return就會成為資料不存在時的<br/>
預設值。如此一來，就可以從資料庫活其他外部服務來更彈性的獲取預設值。
```
$value = Cache::get('key', function () {
    return DB::table(...)->get();
});
```
### 檢查快取資料是否存在
### 快取資料的遞增遞減
### 取用及儲存
### 取用及刪除

## 快取標籤
## 擴充自定的快取驅動
## 快取事件