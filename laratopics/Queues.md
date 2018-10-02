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
[什麼是Key Has Tag ?](https://github.com/Internaltide/Laradep/blob/master/laratopics/Queues.md#Key%20Hash%20Tag%20For%20Redis%20Cluster)

### 阻塞(Blocking)
當使用Redis佇列時，可以額外配置block_for這個設定，配置後的佇列就會改用阻塞式的方式來取出任務。
亦即當佇列中沒有任務時，與佇列的連線就會被阻塞而停止輪詢，直到有新任務進入佇列中或達到定義的
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
### 類別結構

## 分發任務
### 延遲分發
### 任務鏈
### 客製佇列與連接
### 定義任務最大嘗試次數及超時時間
### 頻率限制
### 錯誤處理

## 運行佇列處理器
### 佇列優先層級
### 佇列處理器與部屬
### 任務過期與超時

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