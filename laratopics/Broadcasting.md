# Laravel 廣播

> ~~~
> 在許多現代網頁應用程式中，WebSocket 被用來實作即時性與即時更新的使用者介面。當伺服器上某些資料被更新了，
> 系統就會經由WebSocker連線的方式傳送訊息通知客戶端來進行處理。相較於不斷地輪詢來取得後端資料的方式，這提
> 供了更強大、更有效率的方式。
>
> 為了協助大家建構這樣的應用程式，Laravel 讓你可以更簡單的透過WebSocket連線來廣播你的事件。Laravel 允許你在
> 廣播事件的當下，在伺服器端和客戶端的 JavaScript 應用程式之間共用相同的事件名稱。
>
> PS. 在深入探討Laravel廣播系統之前，務必確認已經讀過所有關於 Laravel 事件與監聽器的說明文件。
> ~~~

## 設定
所有關於事件廣播的設定都放在config/broadcasting這個配置檔中，其內則包含了各驅動的範例配置。Laravel提供了<br/>
數種廣播驅動：Pusher、Redis 和用來在本機開發與除錯的 log。除此之外，你還可以使用名為null的驅動，這意味著將<br/>
完全關閉Laravel的廣播功能。

### 廣播的服務提供者
在廣播任何事件之前，必須先到App\Providers\BroadcastServiceProvider進行註冊。在一個全新乾淨的Laravel應用程<br/>
式中，你僅需到config/app配置檔中的providers陣列來取消App\Providers\BroadcastServiceProvider這一行的註解即<br/>
可。這一個服務提供者就可以讓你註冊廣播的授權路由和回呼函式。

### CSRF Token
Laravel Echo 需要存取當前 session 的 CSRF token 。你應該確認 HTML head 標籤裡頭是否含有用來設定 CSRF token<br/>的 meta 標籤。

## 驅動需求
### Pusher
如果要使用Pusher這個驅動來支持廣播系統的運作，必須先使用Composer安裝Pusher的PHP SDK。
```
composer require pusher/pusher-php-server "~3.0"
```

接著，再到config/broadcasting裡頭設定Pusher憑證。設定檔中已經含有範例設定，讓你可以更迅速地定義Pusher<br/>
金鑰、密碼和應用程式 ID。config/broadcasting 裡面的 Pusher 設定也允許你使用 options 來加入 Puhser 支援的額<br/>外功能選項，範例如下：
```
'options' => [
    'cluster' => 'eu',
    'encrypted' => true
],
```

若同時使用 Pushser 和 Laravel Echo 時，你應該在 resources/assets/js/bootstrap.js 檔案中實體化 Echo 實例的<br/>
時候指定 Pusher 為所需的廣播器。
```
import Echo from "laravel-echo"

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key'
});
```

### Redis
如果使用的是 Redis 廣播驅動，請先安裝 Predis 函式庫。
```
composer require predis/predis
```

Redis 廣播驅動會使用Redis的 pub / sub 功能來廣播訊息。然而，你還是需要搭配 WebSocket 以接受來自 Redis 的<br/>
訊息，並將它們廣播到 WebSocket 頻道上。當 Redis 廣播發佈事件後，該事件會被發佈到事件指定的頻道上，裝載<br/>
的資訊則會使用 JSON 格式，其包含了事件名稱、 資料和產生事件 Socket ID 的使用者（如果需要的話）。

### Socket.IO
如果 Redis 廣播要和Socket.IO伺服器進行搭配使用的話，則必須在應用程式當中載入 Socket.IO JavaScript client<br/>
 library，你可以透過npm來安裝該套件。
```
npm install --save socket.io-client
```

接著，你會需要指定socket.io和 host 連線來實例化 Echo。
```
import Echo from "laravel-echo"

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});
```

最後，你需要執行一個相容的Socket.IO伺服器。Laravel 並未包含Socket.IO伺服器的實作，然而你可以使用社群維護<br/>
的Socket.IO伺服器，其放置於Github 的 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) Repository。

### Queue 的先決條件
在廣播事件之前，你會需要設定並執行一個佇列監聽器。所有的事件廣播都會透過佇列任務來完成，這可以讓應用<br/>
程式的回應時間不會受到嚴重的影響。

## 概念描述
Laravel 的廣播系統允許你使用基於驅動的 WebSockets 將後端的 Laravel 事件廣播給前端 JavaScript 應用程式。<br/>
目前，Laravel內建了Pusher跟Redis的驅動，前端只要透過使用 Laravel Echo Javascript 套件，就可以更簡單的處理<br/>
事件。<br/>

事件是經由頻道進行廣播的，而頻道可以被指定為公開或私人。對於公開頻道來說，任意一個應用程式的訪客都無須<br/>
經過驗證及授權就可以訂閱該頻道。而若想要訂閱私人頻道，使用者就必須認證身份與通過授權才可以訂閱該頻道。

### 使用一個範例應用程式
在深入廣播事件的每個元件前，讓我們用電子商務作個簡易的範例。假設有個頁面可以讓使用者查看訂單的出貨狀況，<br/>
因此讓我們假設在應用程式處理更新出貨進度時，會觸發 **ShippingStatusUpdated** 的事件。
```
event(new ShippingStatusUpdated($update));
```

#### ShouldBroadcast 介面
當使用者正在查看其中一個訂單時，我們並不想讓他們透過重新整理頁面的方式來檢視狀態更新與否。取而代之的是，<br/>
我們要在建立訂單時透過廣播來更新狀態。所以，我們需要在 ShippingStatusUpdated 事件中實作 ShouldBroadcast 介面。<br/>
這會讓 Laravel 在事件被觸發時，廣播這個事件。
```
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ShippingStatusUpdated implements ShouldBroadcast
{
    /**
     * Information about the shipping status update.
     *
     * @var string
     */
    public $update;
}
```

ShouldBroadcast介面的實作需要我們定義一個broadcastOn的方法，該方法負責回傳事件應該廣播的頻道。指令產生的<br/>
事件類別上預設就定義了這個方法，我們只需要填寫它的細節即可。我們僅需讓訂單建立者能夠查看狀態更新就好。<br/>
因此，我們會在與訂單相關的私人頻道上廣播該事件。
```
/**
 * Get the channels the event should broadcast on.
 *
 * @return \Illuminate\Broadcasting\Channel|array
 */
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```

#### 頻道認證
記住，在私有頻道上，使用者必須先經過授權才能去監聽。我們可以在routes/channels這個檔案中進行頻道授權規則<br/>
的定義。在下面這個例子中，我們必須對所有想監聽 **order.1** 這個頻道的使用者進行身分驗證，確認是否為訂單<br/>
的擁有者。
```
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```
例子中使用了channel這個方法，該方法接受兩個參數頻道名稱與回調函數。其中，回調函數是用來檢查使用者是否<br/>
被授權監聽指定的頻道。<br/>

所有的授權回調函數都會將當下已認證使用者實例做為其第一個參數以及任何額外的 wildcard 通配符參數作其後續的<br/>
參數。在第一參數部分也使用了 **{orderId}** 做為佔位符，代表著頻道名稱中的ID部分的通配符。

#### 監聽廣播事件
最後，只剩下在Javascript用戶端監聽事件，而我們可以使用Laravel Echo來達成目的。首先，我們會使用private這個<br/>
方法訂閱私有頻道，接著再用listen方法監聽ShippingStatusUpdated這個廣播事件。而預設上，所有事件的公開屬性都<br/>
會被載入到廣播事件中。
```
Echo.private(`order.${orderId}`)
    .listen('ShippingStatusUpdated', (e) => {
        console.log(e.update);
    });
```

## 定義廣播事件
為了指示 Laravel 廣播某特定事件，則該事件類別上就需實作 Illuminate\Contracts\Broadcasting\ShouldBroadcast<br/>
介面。而該介面需要你實作一個broadcastOn方法，該方法會返回事件要廣播的單一頻道實例或含多頻道實例的<br/>
陣列。這些頻道實例應該為一般頻道、私有頻道或者 Presence 頻道。其中，一般頻道 Channel 代表的是任何使用<br/>
者都可以訂閱的公共頻道，而私有頻道和 Presence 頻道則代表需要經過授權的私人頻道。
```
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('user.'.$this->user->id);
    }
}
```
接著，你只需要和平常一樣觸發事件。一旦事件被觸發，佇列任務會自動透過你指定的廣播驅動來廣播該事件。

### 廣播名稱
預設，Laravel會使用事件類別的名稱來做為廣播的名稱，然而你可以於事件類別中定義broadcastAs方法來客製<br/>
廣播名稱   。
```
/**
 * The event's broadcast name.
 *
 * @return string
 */
public function broadcastAs()
{
    return 'server.created';
}
```

一但你使用 broadcastAs 方法自訂廣播名稱，記得在註冊監聽器時加上前綴符號 **"."** 。這會讓 Laravel Echo 不<br/>
要把應用程式的命名空間加入到事件中。
```
.listen('.server.created', function (e) {
    ....
});
```

### 廣播資料
當事件廣播時，事件類別的所有public屬性都會被序列化並成為廣播時的資料。這允許你的Javascript可以存取到<br/>
這些事件類別的公開屬性。舉例來說，當你的事件類別中含有一個為Eloquent模型的公開屬性$user，那麼事件廣播<br/>
的資料會像下面這樣：
```
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

然而，若想要對廣播資料有更細粒度的控制能力，你可以在事件類別中再定義一個 broadcastWith 方法，該方法可以<br/>
讓你以陣列的格式返回你想要的廣播資料。
```
/**
 * Get the data to broadcast.
 *
 * @return array
 */
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```

### 廣播佇列
預設，所有廣播事件都會放在queue.php中定義的預設佇列連線及預設佇列儲存實體。不過，你可以在事件類別中<br/>
定義一個broadcastQueue屬性來自訂要使用的佇列名稱。
```
/**
 * The name of the queue on which to place the event.
 *
 * @var string
 */
public $broadcastQueue = 'your-queue-name';
```

另外，當廣播事件時想要使用名稱為sync的佇列而非預設佇列時，可以改實作介面 ShouldBroadcastNow 而<br/>
非 ShouldBroadcast。
```
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```

### 廣播條件
有時候你僅想要在特定條件為真時才進行廣播，你可以透過在事件類別內新增broadcastWhen方法來定義你想要<br/>
的條件。
```
/**
 * Determine if this event should broadcast.
 *
 * @return bool
 */
public function broadcastWhen()
{
    return $this->value > 100;
}
```

## 頻道授權
私人頻道只允許你授權過的使用者才可以監聽頻道。整個過程是透過發送一個含指定頻道名稱資料的 HTTP 請求<br/>
到 Laravel 應用程式，接著判斷該使用者是否可以監聽該頻道。而當你使用 Laravel Echo 時，將會自行發送這樣的<br/>
HTTP 請求到要訂閱的私人頻道。當然，也就須要有個合適的路由來回應這些請求。

### 定義授權路由
在 Laravel 應用程式內的 BroadcastServiceProvider 中，你會看到其調用了 Broadcast::routes 方法，該方法就是<br/>
用來註冊處理授權請求的/broadcasting/auth 路由。
。
```
Broadcast::routes();
```

Broadcast::routes 方法會將它定義的路由放在 web 中介層群組中，然而，如果你想要自定指定的屬性，你可以傳遞<br/>
一組路由屬性的陣列到這個方法中。
```
Broadcast::routes($attributes);
```

### 定義授權回調函數
接著，我們需要定義頻道授權的邏輯代碼，而這些可以在routes/channels.php 裡頭直接完成。在該檔案中，可以使用<br/>
Broadcast::channel這個方法來註冊你的授權回調函數。
```
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```
其中，channel方法接受兩個參數，一個是頻道名稱，另一個則是返回 true 或 false，用來判斷使用者是否被授權監聽<br/>
頻道的回調函數。另外，所有授權回調都會將當前已認證使用者作為其第一參數，並使用任意其他的通配符參數作為<br/>
後續參數。在範例中，還用了佔位符{orderId} 來表示頻道名稱的ID 。

#### 授權回調模型綁定
就像 HTTP 的路由模型綁定一樣，頻道路由也可以使用隱式或顯示的路由模型綁定。舉例來說，除了接收字串、數字<br/>
格式的 Order ID，你也可以請求實際的 Order 模型實例。
```
use App\Order;

Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});
```

### 定義頻道類別
如果你的應用程式使用了很多種廣播頻道，你的routes/channels.php可能會變得很臃腫。此時，你可以改用頻道類別<br/>
而非使用閉包來定義頻道的授權。你可以使用Artisan的make:channel指令來創建頻道類別，創建的檔案則會被放在<br/>
App/Broadcasting這個目錄下。
```
php artisan make:channel OrderChannel
```

接著，在config/channel中註冊你的頻道類別
```
use App\Broadcasting\OrderChannel;

Broadcast::channel('order.{order}', OrderChannel::class);
```

最後，在你的頻道類別中的**join**方法時做你的頻道授權邏輯，而這一切就向你在閉包中實作一樣。
```
<?php

namespace App\Broadcasting;

use App\User;
use App\Order;

class OrderChannel
{
    /**
     * Create a new channel instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Authenticate the user's access to the channel.
     *
     * @param  \App\User  $user
     * @param  \App\Order  $order
     * @return array|bool
     */
    public function join(User $user, Order $order)
    {
        return $user->id === $order->user_id;
    }
}
```
> ~~~
> 同樣地，你一樣可以在頻道類別的建構式中透過型別提示來注入依賴，服務容器會自動解析。
> ~~~

## 廣播事件
## 接收廣播
## Presence頻道
## 客戶端事件
## 通知