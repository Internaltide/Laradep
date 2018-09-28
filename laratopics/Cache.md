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
### 獲取單一快取實例
Laravel是透過 **Illuminate\Contracts\Cache\Factory** 與 **Illuminate\Contracts\Cache\Repository** 來提供<br/>
存取快取服務的。Factory契約用來實現存取各類快取驅動；Repository則是典型的快取驅動實作，會隨著快取設定<br/>
檔而變化。

當然，你也可能需要使用到Cache Facade，該Facade提供了方便且簡潔的方法去存取已按契約實作的底層類別。

### 存取多個快取儲存
透過store方法可以存取多個快取儲存體，傳遞到store方法內的參數必須是config/cache設定檔中stores陣列裡的其中之一。
```
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 10);
```

### 從快取中取得資料
get方法是用來從快取儲存體裡取得特定快取資料的，如果資料不存在，則會回傳null。當然，如果您希望<br/>
有預設值，可以透過方法的第二參數，指定當資料不存在時要回傳的預設值。
```
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```
指定預設值時，除了實際的值外，也可以傳遞第一個閉包，而閉包裡頭的Return就會成為資料不存在時的<br/>
預設值。如此一來，就可以從資料庫或其他外部服務來更彈性的獲取預設值。
```
$value = Cache::get('key', function () {
    return DB::table(...)->get();
});
```

#### 檢查快取資料是否存在
使用has方法來檢查資料是否存在於快取，如果值為null或false則返回false
```
if (Cache::has('key')) {
    //
}
```

#### 快取資料的遞增遞減
使用方法increment或decrement可以調整快取中的整數型資料，預設一次調整值為1，但接受<br/>
第二參數來指定每次調整的量
```
// 增加1
Cache::increment('key');

// 增加$amount
Cache::increment('key', $amount);

// 減少1
Cache::decrement('key');

// 減少$amount
Cache::decrement('key', $amount);
```

#### 取用及儲存
從快取中取用資料時，若資料不存在則可以儲存一個預設值到快取中。例如，從快取獲取用戶資料時，<br/>
如果資料不存在就可以透過閉包到資料庫撈出資料，接著回傳並儲存進快取中。
method:**remember**
```
// 存入快取時有過期時間
$value = Cache::remember('users', $minutes, function () {
    return DB::table('users')->get();
});
```
method:**rememberForever**
```
// 存入快取時為永久性
$value = Cache::rememberForever('users', function() {
    return DB::table('users')->get();
});
```

#### 取用及刪除
從快取中取用完資料後把資料從快取中移除，若資料不存在時則回傳null。
```
$value = Cache::pull('key');
```

### 把資料存入快取中
使用Cache Facade的put方法將資料儲存進快取並指定有效時間
```
Cache::put('key', 'value', $minutes);
```
除了指定有效時間外，也可以傳遞DateTime實例來直接指定一個過期時間點
```
$expiresAt = now()->addMinutes(10);

Cache::put('key', 'value', $expiresAt);
```

#### 只有資料不存在時才存入快取
使用Cache Facade的add方法也可將資料儲存進快取，但只有再資料不存在時才會執行。<br/>
當資料確實加入快取時回傳true，反之回傳false。
```
Cache::add('key', 'value', $minutes);
```

#### 將資料永久加入快取
使用forever方法將資料永久加入快取中，但也因無過期時間，所以必須使用forget方法才能從快取中移除。
```
Cache::forever('key', 'value');
```

### 將資料移出快取
你可以使用forget將資料從快取中移除
```
Cache::forget('key');
```

#### 清空快取
使用flush方法清空所有快取資料。要注意該方法並不會考慮快取前綴，就是完全清空，所以使用時要注意。
```
Cache::flush();
```

### 原子鎖
> 要使用這個特性前，你的快取驅動必須是Memcached或Redis。除此之外，所有的主機還必須與同一台<br/>
> 中央快取伺服器做溝通。<br/>

透過原子鎖，可以不用再擔心操作的兢爭。舉例來說，Laravel Forge就使用了原子鎖來確保一個時間點下，只能<br/>
存在一個被執行的遠端任務。你可以使用Cache::lock來創建及管理鎖。當特定請求獲得該鎖就可以開始其邏輯運作，<br/>
其他請求則必須等到鎖釋放變成有效狀態。
```
if (Cache::lock('foo', 10)->get()) {
    // Lock acquired for 10 seconds...

    Cache::lock('foo')->release();
}
```
get方法也接受閉包，再閉包被調用執行後，Laravel就會自動釋放鎖
```
Cache::lock('foo')->get(function () {
    // Lock acquired indefinitely and automatically released...
});
```
當發出請求卻發現鎖尚未呈現有效狀態時(被其他請求使用中)，可使用block方法指示Laravel等待鎖被釋放<br/>
並變成有效。
```
if (Cache::lock('foo', 10)->block()) {
    // Lock acquired after waiting...
}
```
預設，block方法會無限期等待，直到鎖變成有效，若不想要這樣的行為，可以改用blockFor方法，<br/>
該法可以指定等待秒數。時間內，若鎖還是不能變成有效，則拋出Illuminate\Contracts\Cache\LockTimeoutException<br/>
例外。
```
if (Cache::lock('foo', 10)->blockFor(5)) {
    // Lock acquired after waiting maximum of 5 seconds...
}

Cache::lock('foo', 10)->blockFor(5, function () {
    // Lock acquired after waiting maximum of 5 seconds...
});
```

### 快取輔助方法
除了Facade跟Contract外，Laravel亦為了快取提供了等價的全域輔助方法來從快取中取的資料或將資料存入快取。<br/>

當接收的是一個字串參數時，回傳鍵值為該字串值的快取資料
```
$value = cache('key');
```
當傳遞的是一組Key-Value跟過期時間時，輔助函數會將資料存入快取並指定有效時間
```
cache(['key' => 'value'], $minutes);

cache(['key' => 'value'], now()->addSeconds(10));
```
PS. 在測試時使用Helper Function cache時，可以使用Cache::shouldReceive，就好像在測試Facade一樣。

## 快取標籤
## 擴充自定的快取驅動
## 快取事件