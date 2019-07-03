# Laravel Events


> ~~~
> Laravel 提供了一個簡易的觀察者模式實作，允許你訂閱或監聽應用程式中發生的各類事件。定義事件用的類別預設
> 會放在app/Events這個目錄下，而監聽器類別則放在app/Listeners目錄下 。事件為應用程式的各種面向提供了的良好
> 的解偶，因為單一事件能有多個不互相依賴的監聽器。例如，你可能希望每次訂單出貨時就發送 Slack 通知給使用
> 者。此時，你應該做的並不是在訂單處理程式中加入slack通知的邏輯而讓其互相耦合。而是，你要能夠簡單的觸發
>  訂單出貨事件，讓監聽器可以接收並轉換成一個 Slack 通知。
> ~~~

## 註冊事件與監聽器
Laravel 應用程式中引入的 EventServiceProvider 提供一個便捷的地方來註冊應用程式的所有事件監聽器。其中listen<br/>
這個陣列格式的屬性可用來定義事件與其對應的監聽器，其中，事件類別為鍵；監聽器類別為值。當然，你可以可能<br/>
地新增事件到這組陣列來滿足應用程式的需求。下面是新增 OrderShipped 事件的範例。
```
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'App\Events\OrderShipped' => [
        'App\Listeners\SendShipmentNotification',
    ],
];
```

### 產生事件與監聽器
手動逐一地建立事件與監聽器是非常笨重麻煩的，你可以先將事件與監聽器註冊到EventServiceProvider中，再透過<br/>
event:generate這個Artisan指令來建立所有以註冊在EventServiceProvider中的事件以及監聽器。當然，若是已經存在<br/>
的事件和監聽器就不會有任何變動。
```
php artisan event:generate
```

### 手動註冊事件
一般來說，應該將事件註冊到EventServiceProvider 的 $listen 屬性當中。然而，你也可以在EventServiceProvider<br/>
的boot方法中手動使用閉包來註冊一個事件。
```
/**
 * Register any other events for your application.
 *
 * @return void
 */
public function boot()
{
    parent::boot();

    Event::listen('event.name', function ($foo, $bar) {
        //
    });
}
```

#### 使用萬用字元的事件監聽器
你甚至可以使用 * 作為萬用字元參數來註冊監聽器，這可以讓你在同個監聽器上抓到多個事件。萬用字元的監聽器<br/>
將接收到的名稱作為它們的第一個參數，並將整個事件資料的陣列作為第二個參數：
```
Event::listen('event.*', function ($eventName, array $data) {
    //
});
```

PS. 官方說明中，有點讓人糊塗，到底是註冊事件還是註冊監聽器？現在綜合看來比較像是註冊一組事件與監聽器<br/>
，閉包則是做為監聽器的角色。待確認...

## 定義事件
所謂的事件類別就是一個資料容器，其用來儲存事件相關的資料。舉例來說，假設我們創建了一個可接受Eloquent模型<br/>
的訂單出貨事件。就如同下例所示，類別內部並未包含任何邏輯，僅僅包含了一個訂單模型。如果Event物件被 PHP<br/>
的 serialize 函式給序列化了，那麼事件所使用的 SerializesModels trait 將會被用來序列化任意的 Eloquent 模型。
```
<?php

namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use SerializesModels;

    public $order;

    /**
     * Create a new event instance.
     *
     * @param  \App\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```

## 定義監聽器
接著，讓我們看一下範例事件的監聽器。事件監聽器的handle方法會接受一個事件實例，而使用event:generate 指令<br/>
建立事件時，事件類別的 handle 方法將會自動定義好事件的型別注入。在 handle 方法中，你則可以執行任何必要的<br/>
操作來回應事件的發生。
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        // Access the order using $event->order...
    }
}
```
> ~~~
> 事件監聽器也可以在建構子上注入任何需要的依賴。所有的事件監聽器都會透過 Laravel 服務容器來解析，
> 所以依賴才會被自動注入。
> ~~~

### 停止事件的傳播
有時候你會希望停止某一事件繼續傳播到其它監聽器，你可以透過在監聽器的 handle 方法內返回 false 來達到目的。

## 佇列事件監聽器
若你的監聽器是要處理像是寄送郵件或發出Http請求之類較為耗時的任務，使用隊列監聽器會很有幫助。在使用隊列<br/>
監聽器之前，請先確認已配置好Laravel Queue的使用並在你的環境中啟動佇列監聽器。<br/>
要讓監聽器能夠被加入佇列，你的監聽器類別必須額外實作ShouldQueue這個介面。而透過指令event:generate所產生<br/>
的監聽器，預設就會實作該介面，因此你可以直接使用。
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    //
}
```
當這個監聽器被一個事件觸發調用時，事件分發器會自動使用Laravel的佇列系統將其加入佇列。如果在佇列中執行該<br/>
監聽器任務時並未產生例外，佇列任務就會在完成後自動刪除。

#### 自訂佇列連線與佇列名稱
如果要自定義事件監聽器所使用的佇列連線及佇列儲存體，可以在監聽器類別裡定義 **$connection** 和 **$queue** 兩種屬性。
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * The name of the connection the job should be sent to.
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * The name of the queue the job should be sent to.
     *
     * @var string|null
     */
    public $queue = 'listeners';
}
```

### 手動存取佇列
如果你需要手動存取監聽器其底層之佇列的 delete 和 release 方法，你可以使用Illuminate\Queue\InteractsWithQueue<br/>
在生成的監聽器類別中，則預設就會載入這個Trait以提供這些方法。
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

### 處理失敗的任務
有時候在佇列中的事件監聽器可能會有一些失敗的狀況，如果事件監聽器的嘗試超過了佇列處理器中定義的最大嘗試<br/>
次數，監聽器將會自行調用內部的 failed 方法。failed 方法將會接收事件實例與導致失敗的例外。
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        //
    }

    /**
     * Handle a job failure.
     *
     * @param  \App\Events\OrderShipped  $event
     * @param  \Exception  $exception
     * @return void
     */
    public function failed(OrderShipped $event, $exception)
    {
        //
    }
}
```

## 事件分發
為了對事件進行分發，你可能需要將事件實例傳進**event**這個輔助方法，該方法會自動將事件分發到對應註冊好<br/>
的監聽器上。由於  event 輔助方法是全域性的，所以你可以在應用程式的任何地方調用它。
```
<?php

namespace App\Http\Controllers;

use App\Order;
use App\Events\OrderShipped;
use App\Http\Controllers\Controller;

class OrderController extends Controller
{
    /**
     * Ship the given order.
     *
     * @param  int  $orderId
     * @return Response
     */
    public function ship($orderId)
    {
        $order = Order::findOrFail($orderId);

        // Order shipment logic...

        event(new OrderShipped($order));
    }
}
```
> ~~~
> 測試時，它將有助於斷言某些沒有實際觸發監聽器的事件。Laravel 的內建關於測試用的輔助方法讓它變得很簡單。
> ~~~

## 事件訂閱者

### 撰寫事件訂閱者
事件訂閱者是個可以訂閱多事件的類別，讓你在單一類別中就能定義多個事件的處理器。事件訂閱者必須實作一個<br/>
**subscribe** 方法，該方法將接收一個事件分發器實例，你可以使用該分發器上調用 listen 方法以註冊事件監聽器。
```
<?php

namespace App\Listeners;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function onUserLogin($event) {}

    /**
     * Handle user logout events.
     */
    public function onUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param  \Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@onUserLogin'
        );

        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@onUserLogout'
        );
    }

}
```

### 註冊事件訂閱者
在完成事件訂閱者的定義後，你還得將其註冊到事件分發器中。你可以在EventServiceProvider中使用$subscribe屬性<br/>
來註冊你的訂閱者。以下面為例，我們將新增一個 UserEventSubscriber 訂閱者。
```
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        //
    ];

    /**
     * The subscriber classes to register.
     *
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventSubscriber',
    ];
}
```
