# Laravel 工作佇列

> ~~~
> Laravel佇列為了多種不同的佇列服務提供了統一的API，諸如Beanstalk、Amazon SQS、Redis乃至於
> 關聯式資料庫。佇列能夠讓你延遲一些比較耗時的工作任務，如郵件寄送。這種方式可以大大提升
> 應用程式在處理網路請求的工作效率。
>
> 佇列的配置檔為config/queue，裡頭可以找到各種佇列驅動的連線設定。其中，提供了一個本地用且
> 可用來直接執行任務的同步驅動以及一個用來丟棄佇列任務的Null佇列驅動。
>
> 而Laravel官方也為Redis提供了一個套件Horizon，它是一個Redis的配置系統，擁有美觀的儀表板。
> ~~~

## 連線 VS. 佇列
在使用Laravel佇列前，先了解佇列與連線的區別是很重要的。每一個連線定義都代表著一個佇列服務，而任何<br/>
一個連線都可能擁有多個佇列堆疊來提供不同的工作任務使用。<br/>
每個連線配置區內會有一項queue設定項，其用意是定義該連線預設使用的佇列堆疊。換句話說，若未特別<br/>
定義使用哪個佇列堆疊時，工作任務就會被分發到預設的佇列堆疊區，即queue所設定。
```
// This job is sent to the default queue...
Job::dispatch();

// This job is sent to the "emails" queue...
Job::dispatch()->onQueue('emails');
```
一些簡單的應用或許只需要一個簡單的佇列就好。但當你需要任務優先度或任務分類時，定義多項佇列堆疊<br/>
就會非常有用。尤其，Laravel可以對佇列設定優先順序。例如，將任務分派到優先度為high的佇列後，可以<br/>
使用Artisan指令讓佇列處理器優先處理高優先度的工作任務。
```
php artisan queue:work --queue=high,default
```

## 使用各類佇列驅動的前置條件
### Database
使用database佇列驅動時，你會需要一張資料表來保存工作任務。透過Artisan指令queue:table就可以建立<br/>
對應所需資料表的遷移檔，然後使用migrate指令就可以建立該表。
```
php artisan queue:table

php artisan migrate
```

### Redis
要使用Redis佇列驅動，只需要在config/queue存在Redis的連線配置即可使用。

### Redis Cluster
若是使用Redis叢集架構，則配置中的queue設定值就必須包含一個Key Hash Tag，才能確保所有Redis Key的資料<br/>
都被存放在同一個Hash Slot中。(Hash Tag會使用{}來包裹)
```
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
```
[什麼是Key Has Tag ?](https://github.com/Internaltide/Laradep/blob/master/laratopics/Queues.md#key-hash-tag-for-redis-cluster)

### 阻塞(Blocking)
當使用Redis佇列時，可以額外配置block_for這個設定，配置後的佇列就會改用阻塞式的方式來取出任務。<br/>
亦即當佇列中沒有任務時，與佇列的連線就會被阻塞而停止輪詢，直到有新任務進入佇列中或達到定義的<br/>
block_for阻塞時間。透過調整block_for的值會遠比持續的詢問Redis是否有新任務有效率的多。
```
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 90,
    'block_for' => 5,
],
```
> ~~~
> 注意!! 阻塞是一個實驗性的新功能，在任務取出的同時，若發生了Redis Server或佇列處理器crashe的狀況，
> 則有小機率造成佇列任務的遺失。
> ~~~


### Others
下列幾個佇列驅動的使用，則須要先行安裝相關的套件
 - Amazon SQS: **aws/aws-sdk-php ~3.0**
 - Beanstalkd: **pda/pheanstalk ~3.0**
 - Redis: **predis/predis ~1.0**

## 建立任務
### 建立任務類別
預設，所有的佇列任務都會存放在app/Jobs目錄下，它會在第一次執行make:job這個建立<br/>
佇列任務的Artisan指令後被創建。而使用指令建設的任務類別預設都會實作Illuminate\Contracts\Queue\ShouldQueue<br/>
這個介面，意味著任務都是設計來進入佇列而非同步執行。<br/>

使用指令建立工作任務
```
php artisan make:job ProcessPodcast
```

### 任務類別結構
初始任務類別的結構很簡單，只包含了一個handle方法。一般而言會在佇列工作器處理該任務時被調用。<br/>
另外，除了建構式外，我們也可以在handle方法透過型別提示來注入依賴。
```
<?php

namespace App\Jobs;

use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }
}
```
任務類中還引用了SerializeModels這個Trait，提供對Eloquent模型進行序列化與反序列化。<br/>
如果你在建構式中注入了Eloquent模型，則只有模型的惟一識別欄位才會被序列化到佇列中。<br/>
接著，在佇列工作器執行該任務時，系統就會自動到資料庫中取出正確的模型。這樣的處理<br/>
方式可以避免掉直接對一個Eloquent模型序列化所會產生的問題。
> ~~~
> 注意!! 放入佇列任務的二進位資料(Ex.圖片...)，必須使用base64_encode進行編碼。否則，任務進入
> 佇列時，可能會無法正確地被序列化為JSON。
> ~~~

## 分發任務
請使用任務類別自帶的dispatch進行分發，傳遞給dispatch的參數就是傳遞給建構式的參數。
```
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast);
    }
}
```

### 延遲分發
如果想要延遲任務的執行，那麼在分發任務時可以改用delay方法處理。下面這個例子中，<br/>
任務派發後須等待10分鐘後才會開始執行。
```
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
    }
}
```
注意!! 若你用的是Amazon的佇列服務，其最大延遲時間只有15分鐘。

### 任務鏈
你可以使用withChain來定義一個按順序執行的任務列表，如果其中一個任務執行失敗，<br/>
則後續的任務都將不會被執行。
```
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```

#### 任務鏈的連線與佇列
如果你想為一個任務鏈定義預設的連線與佇列，可以使用allOnConnection和allOnQueue<br/>
兩種方法來定義任務鏈的預設連線與佇列，但如果其中的任務有明確指定要使用的連線與佇列，<br/>
則仍會使用其任務自身的定義。
```
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');
```

### 自定分發用的佇列與連接
你可以使用onQueue方法將任務分發到不同的佇列實體，並藉以達到分類的目的，甚至優先考量要分配<br/>
多少workers到各種不同的佇列。注意，該方法不會將任務分發到已配置的不同連線，反而是使用單一<br/>
連接。

#### 自訂分發使用的佇列
```
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');
    }
}
```

#### 自訂分發使用的連線
如果是在使用多連線環境下，可以使用onConnection方法來指定任務分發時所使用的連線。
```
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
    }
}
```

#### 鏈式調用，同時指定連線與佇列
```
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

### 定義任務最大嘗試次數及超時時間
#### 指定最大嘗試次數
透過Artisan指令的tries選項指定最大嘗試次數
```
php artisan queue:work --tries=3
```
使用更細粒度的方法，直接在任務類別設定$tries屬性來表示最大嘗試次數。<br/>
此方式的優先度會大於在Artisan指令行上使用選項設置。
```
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```
除了直接指定最大嘗試次數，也可指定任務嘗試的有效時間，在這段時間內該任務<br/>
可無限嘗試直到超時並宣告任務失敗。方式為在任務內部定義retryUntil方法。<br/>
(佇列監聽器中亦可使用retryUntil方法)
```
/**
 * Determine the time at which the job should timeout.
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}
```

#### 指定超時時間
透過Artisan指令的timeout選項指定任務的超時時間。這邊要釐清的是，這裡的超時時間指的是<br/>
任務可執行的有效時間；前面那一項則是可持續嘗試將任務啟動的時間。
```
php artisan queue:work --timeout=30
```
同樣地，亦可使用更細粒度的方法，在任務類別設定$timeout屬性來表示任務運行超時時間。<br/>
其優先度亦大於在Artisan指令行上使用選項設置的方式。
```
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;
}
```

### 存取頻率限制
注意!! 此項特徵需要在Redis的環境下始能運作。<br/>
所以當你的應用可以與Redis進行溝通時，你就可以藉由時間或併發來限制佇列運作。<br/>
當佇列任務調用到有速率限制的API時，這個特性就會很有幫助。因為你可以使用throttle方法<br/>
限制任務的執行頻率(如每60秒只可執行10次)，從而降低API的調用。<br/>
另外，在未獲得鎖(即超過了限制頻率)的情況下，一般你應該做的就是將任務重新放回佇列中，<br/>
以便稍後可以重試。
```
// 每60秒只能執行10次
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // 限制頻率內，可獲得鎖
    // 任務邏輯...
}, function () {
    // 超過限制頻率，無法獲得鎖
    return $this->release(10);
});
```
PS. throttle方法接受的參數需是任務類別中可唯一識別的任意字串。例如，任務類別名或<br/>
任務操作模型的ID值。<br/>

另一種方式，可以限制任務的最大執行數量，當佇列任務正在處理一個只能一次被一個程序存取<br/>
的資源時，透過funnel方法就可以限制特定任務一次只能使用一個worker。
```
Redis::funnel('key')->limit(1)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```
PS. 使用頻率限制時，可能會難以確認任務執行的嘗試次數。此時將頻率限制改為合併使用<br/>
指定任務執行嘗試的有效時間(Time Based Attempts)會比較有用。

### 錯誤處理
當任務執行過程中因出現異常而拋出了例外，任務會自動被丟回佇列以便馬上重新嘗試執行，<br/>
直到達到最大重新嘗試次數，才宣告任務失敗。


## 運行佇列處理器
Laravel使用佇列處理器來把工作任務推送進佇列中，並提供指令queue:work來啟動一個處理器。<br/>
該指令命令運行後，除非手動進行中斷或關閉終端機，否則就會持續執行。
```
php artisan queue:work
```
> 若要改用背景執行，就得使用 Linux Supervisor這類的進程管理器，以確保佇列處理器不會中斷。

PS. 要記住，佇列處理器是一種常駐的程序，並會在記憶體中儲存其啟動狀態。也因為如此，<br/>
       已啟動的佇列處理器並不會知曉代碼的變更，所以在應用程式佈署階段都必須重啟你的<br/>
       佇列處理器。<br/>

#### 執行單一任務
使用選項--once可以設定讓處理器只從佇列取出一項任務來執行。
```
php artisan queue:work --once
```

#### 指定使用連線
```
php artisan queue:work redis
```

#### 指定處理器只連接執行某個佇列實體
```
// 啟動處理器運行emails這個佇列任務，並使用redis連線
php artisan queue:work redis --queue=emails
```

#### 資源注意事項
常駐型處理器並不會在運行任務後進行框架重啟，所以，對於一些占用資源過高的任務進行<br/>
完成後釋放資源的動作。例如，使用GD處理圖片完成後，就應該使用imagedestroy來釋放記憶體。

### 佇列優先層級
有時候你可能會想要控制佇列的執行優先度。例如，在config/queue中將特定連線的預設佇列更改<br/>
為低優先度的low佇列。以下方式表示將任務分發到高優先度的high佇列。
```
dispatch((new Job)->onQueue('high'));
```
同時定義處理low與high兩個佇列，並指定high執行完才執行low
```
php artisan queue:work --queue=high,low
```

### 佇列處理器與佈署
重啟佇列處理器，該行為通常被使用在應用程式佈署流程中。目的常常是為了反映常駐處理器<br/>
在佇列相關代碼的變更。該重啟指令並非立即強制中斷，它會讓各處理器將處理中的任務執行完畢<br/>
才中止，所以不會有丟失任務的風險。<br/>
```
php artisan queue:restart
```
PS. 執行重啟時，原處理器程序會被kill掉。所以，你仍需要使用進程管理器來自動重啟處理器。<br/>
> 注意!! 重啟訊號會使用快取來儲存，所以要使用此特性前，須確保配置好快取驅動。

### 任務過期與超時
#### 任務過期
透過在配置檔裡每個連線配置的retry_after定義重新嘗試執行任務前所必須等待的時間秒數。<br/>
例如，retry_after被設定成90，則任務執行90秒後沒被刪除時，就會重新放回佇列中。<br/>
也因為這樣，通常你應該使用任務可能執行的最長時間來做為其配置值。<br/>
> 注意!! Amazon SQS佇列驅動並不支援retry_after配置，你必須改用AWS控制台來設定任務[的Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)

#### 佇列處理器超時
即前述的queue:work --timeout，其指的是任務可執行的最長時間，當超過定義值就會被刪除。<br/>
看起來與retry_after相當類似，兩者都定義了一個時間門檻值，只是超過門檻值後的處理方式<br/>
卻完全不相同。<br/>

timeout特性是將執行該任務的子進程殺掉，retry_after則是將任務重新丟回排程。因此，<br/>
當出現了任務執行時間超過了所設定的retry_after後，就會產生任務被重複執行的可能性。<br/>
認清retry_after依舊只是個概估值，並無法確保每一次任務執行都一定不會超過設定值。所以，<br/>
若需要保證任務只會成功被執行一次，通常要搭配一個小於retry_after幾秒鐘的timeout設定，確保<br/>
當出現任務執行超時前可以事先被刪除而免於重新回到佇列中。<br/>
( 只用一個timeout不就能確保任務只被成功執行一次嗎? 為何一定要併用? retry_after是強制需要的?<br/>
  故意設計成預設超時就丟回佇列重跑的行為，然後在需要確保只能執行一次的任務場合再合併使用<br/>
  timeout? )<br/>

  #### 佇列處理器睡眠時間
  預設只要佇列中含有有效任務時，處理器就會持續地取出任務來執行。當你設定了處理器的睡眠<br/>
  時間後，處理器會在佇列中沒有任務時開始睡眠，直到設定的時間醒來。睡眠途中若有新任務<br/>
  進入佇列也不會喚醒處理器。你可以使用sleep這個選項來設定佇列處理器的睡眠時間。<br/>
  ```
  php artisan queue:work --sleep=3
  ```

## 管理員設定

## 處理失敗的任務
### 清除失敗任務
### 任務失敗事件
### 任務失敗重試

## 任務事件


>
> ## Key Hash Tag For Redis Cluster
>
> Redis的叢集架構支援一次調用多個Key的資料操作(Multi-Key)，只是Redis叢集卻無法跨節點操作。所以，
> 為了Multi-Key在叢集上也能正常運作，Redis叢集實現了Key Hash Tag的概念，簡單說就是讓每個KeyName都
> 可以再額外包含一個Tag，該Tag會用{}來包裹，其可用來強制某些Keys的資料都可以被儲存於同一個節點上。
>
> 而其機制如下：
> 當KeyName內含Hash Tag時，就會使用該Tag來計算資料的儲存位置；反之，則使用Key來求得儲存位置。因此，
> 對於foo、{foo}.teacher、{foo}.student、{shh}.roommate、shh這五個Key Name來說，前三筆資料會存放在同一個
> 節點，後兩筆資料放在同一個節點。
>
> ~~~
>