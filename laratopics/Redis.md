# Redis

## 簡介
Redis 是一個開源、優異的 key-value 儲存系統，而由於其儲存鍵可以使用諸如 string、hashes、lists、sets 及<br/>
sorted sets，所以常又被稱為資料結構伺服器。<br/>

不過，在 Laravel 使用 Redis 前，必須先使用 Composer 安裝 predis/predis 套件：
```
composer require predis/predis
```
另一個選擇是使用PECL來安裝 PhpRedis 套件。只是這個擴充套件的安裝過程較為複雜，但對於依賴 Redis 較重的<br/>
應用程式來說，PhpRedis 或許較可能滿足較佳的效能需求。

## 配置
關於Redis的配置會放在config/database.php這個設定檔案中。在其中你會看到一個鍵為 redis 的陣列，裡頭包含了<br/>
應用程式使用 Redis 伺服器的相關設定項。
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
預設的伺服器設定應該是足以滿足開發環境所需。不過你仍可以按照自己的需求來變更配置方式，設定檔中的每一個<br/>
被定義的 Redis 伺服器都必須有一個 name、host、port。

### 叢集配置
如果你的應用程式使用了 Redis 叢集，你可以在 Redis 設定檔案設定內的 clusters 選項設定 cluster 相關的資訊。
```
'redis' => [

    'client' => 'predis',

    'clusters' => [
        'default' => [
            [
                'host' => env('REDIS_HOST', 'localhost'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', 6379),
                'database' => 0,
            ],
        ],
    ],

],
```
預設下，叢集會跨越你的所有節點進行 client-side sharding，允許你匯集節點並建立一個大量可用的記憶體。然而，<br/>
要注意的是 client-side sharding 並不會處理故障轉移，所以 Redis 叢集主要適用於對某個主要資料儲存進行資料的<br/>
快取。( sharding 類似一種資料切割的概念 )<br/>
如果你想要使用原生的 Redis 叢集，可以在設定檔內使用 options 選項進行相關的設定。
```
'redis' => [

    'client' => 'predis',

    'options' => [
        'cluster' => 'redis',
    ],

    'clusters' => [
        // ...
    ],

],
```

### Predis
除了host、port、database 以及 password 這些選項外，Predis 還支援額外的[連線參數](https://github.com/nrk/predis/wiki/Connection-Parameters)。要使用這些額外的設定選項，<br/>
你可以在 config/database.php 直接新增這些參數至你的 Redis 伺服器設定區塊中。
```
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_write_timeout' => 60,
],
```

### PhpRedis
> 注意!! 如果是使用 PECL 安裝的 PhpRedis 擴充套件，你需在 config/app.php 設定檔內重新命名 Redis 的別名。

為了切換成使用 PhpRedis 擴充套件，你需要在 Redis 設定檔將 client 選項更改為 phpredis。這個選項可以在 <br/>
config/database.php 設定檔中找到。
```
'redis' => [

    'client' => 'phpredis',

    // Rest of Redis configuration...
],
```
而除了host、port、database 以及 password 這些選項外，PhpRedis 還支援 persistent, prefix,  read_timeout 及<br/>
timeout 等連線參數。
```
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_timeout' => 60,
],
```

## 與 Redis 互動
你可以透過 Redis Facade 所提供的數種方法來與Redis進行溝通互動。Redis Facade 還支援動態方法，亦即<br/>
呼叫任何 facade 內的 Redis 指令都會直接傳遞至 Redis。在下面的範例中，我們將透過呼叫 Redis Facade 的<br/>
Get 方法來調用Redis 的 get 指令。
```
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redis;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Redis::get('user:profile:'.$id);

        return view('user.profile', ['user' => $user]);
    }
}
```
當然，如同先前所提，你可以藉由 Redis Facade 來執行任何的 Redis 指令。Laravel 使用了魔術方法來傳遞這些<br/>
指令到Redis 伺服器。所以只需要傳遞原生 Redis 指令期望所需的參數。
```
Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

你還可以使用 command 方法來傳遞要調用的 Redis 指令，這個方法接受指令名稱作為第一個個參數，同時將參數值<br/>
使用陣列包裝後作為第二個參數。
```
$values = Redis::command('lrange', ['name', 5, 10]);
```

### 使用多個 Redis 連線
你可以透過 Redis::connection 方法來取得 Redis 實例，而且會返回預設配置的 Redis 實例。
```
$redis = Redis::connection();
```

然而，你可以透過傳遞連線或叢集名稱來取得特定配置的 Redis 實例。
```
$redis = Redis::connection('my-connection');
```

### 管道指令
如果需要在一次的操作就向伺服器發送多個指令，可以使用管道的方式。pipeline 方法接受一個可接收 Redis 實例<br/>
的閉包。你可以將所有的指令發布到該 Redis 實例，他們將在該次操作下一次執行完畢。
```
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

## Redis 的 Pub/Sub
Laravel 提供了一個方便的介面操作 Redis 的 publish 及 subscribe 指令。這些 Redis 指令允許你在指定的頻道上<br/>
監聽訊息。你可以從其他應用程式甚至使用其他程式語言來推播訊息到該頻道，讓你更方便地在應用程式與進程<br/>
之間溝通。<br/>

首先，我們先用 subscribe 方法設定頻道的監聽。而因為呼叫 subscribe 方法是一個持續執行的進程，我們將在<br/>
Artisan指令類別內部來定義呼叫這個方法。
```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Subscribe to a Redis channel';

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        Redis::subscribe(['test-channel'], function ($message) {
            echo $message;
        });
    }
}
```

現在，我們就可以使用 publish 方法在頻道上開始推播訊息
```
Route::get('publish', function () {
    // Route logic...

    Redis::publish('test-channel', json_encode(['foo' => 'bar']));
});
```

### 通配字元訂閱
使用了 psubscribe 方法就可以用通配字符來定義要監聽的頻道，藉此就能方便的訂閱到全部的頻道。其中，頻道<br/>
名稱 $channel 則作為第二參數傳遞給閉包回調函數。
```
Redis::psubscribe(['*'], function ($message, $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function ($message, $channel) {
    echo $message;
});
```
