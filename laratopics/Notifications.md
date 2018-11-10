# Laravel Notifications


> ~~~
> 除了支援郵件寄送外，Laravel也提供其它數種不同類型頻道的通知寄送。諸如，SMS (via Nexmo)、Slack。通知也可以
> 被存放在資料庫中，以便直接在網頁中進行顯示。一般而言，通知訊息通常都很短，主要是用來通知使用者應用程式
> 發生了那些事。如果你在寫的是帳單管理的應用程式，或許會需要通過Email或SMS頻道來寄送一個發票支付的通知
> 給你的使用者。
> ~~~

## 創建通知
在 Laravel ，每一種通知頻道都會使用一個類別來定義並且存放到 app/Notifications 目錄下。你可以使用Artisan<br/>
指令 make:notification 來創建。每個剛建立的通知類別，預設都會包含一個via方法及**數種**訊息建構的方法(如<br/>
toMail 或 toDatabase)，可用來將通知轉換成為特定頻道的訊息。
```
php artisan make:notification InvoicePaid
```

## 寄發通知
### 使用Notifiable Trait
通知有兩種寄送方式：使用 Notifiable trait 內的 notify 方法或使用 Notification facade。下面則為 trait 的範例：
```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;
}
```
這個 trait 預設就有被 App\User 模型使用，其包含了一個用於寄送通知的方法 **notify**。該方法會接收一個通知實例。
```
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```
PS. 當然你能在任意模型內使用 Illuminate\Notifications\Notifiable trait，其並不受限只能在 User 模型內使用。

### 使用Notification Facade
另一方式，可以使用Notification Facade的send方法。當你需要將通知發送到多個可通知實體（如用戶集合）時，這會<br/>
非常地有用。為了寄發使用 facade 發送的通知 ，需要將所有可通知實體和通知實例傳遞到 send 方法中。
```
Notification::send($users, new InvoicePaid($invoice));
```

### 指定遞送頻道
每個通知類別都有一個 via 方法用於判別要將通知寄往哪個頻道。換句話說，通知可以被寄送至  mail、database、<br/>
broadcast、nexmo 和 slack 頻道。
> 如果你想要使用其他的遞送頻道，像是 Telegram 或 Pusher，可以參考社群維護的 Laravel Notification Channels 網站。

via 方法接收了一個 $notifiable 實例，該實例即為要向其發送通知的類別實例。你可以使用該實例指定要將訊息傳送<br/>
到何種頻道。
```
/**
 * Get the notification's delivery channels.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

### 通知佇列
寄送通知可能會十分耗時，特別是當頻道是透過外部API來派發通知時。為了加速應用程式的反應時間，必須讓通知<br/>
可以加入到佇列中，你可以透過在通知類別內實作 ShouldQueue 介面和引用 Queueable trait 來達到目的。不過如果你<br/>
是透過指令 make:notification 所建立通的知類別，則該介面與 Trait 預設就已經被引用。
```
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

一旦 ShouldQueue 介面被新增至通知類別內，你仍可以一如往常的使用通知類別。Laravel 會自動偵測通知類別內的<br/>
 ShouldQueue 介面並自動將要遞送的通知加入到佇列中。
 ```
 $user->notify(new InvoicePaid($invoice));
 ```

如果你需要延遲通知的寄送，可以鏈式調用delay方法。
```
$when = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($when));
```

### 按需要通知
有時你需要寄送通知到並非以 "user" 類別儲存的使用者，可能無法藉由前面的方式寄發通知。你可以透過使用<br/>
Notification::route，就能夠在寄送通知前指定臨時的通知路由。
```
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```

## 郵件通知
> ~~~
> Mail頻道的通知寄送與Mail功能一節內的Mailable在功能性上很相似。一般來說會直接使用Mailable，因為
> 是新的設計，使用上與定製的彈性都會比較簡單。但若需要多重的寄送方式，就可以改用通知來實作。
> 不過版本5.3.7開始的Laravel已經將Mailable與通知進行整合，彼此已經可以互相協作。所以你可以直接
> 先實作Mailable，有進一步需求後再透過通知類別來將Mailable寄出。(透過toMail()返回要寄送的Mailable)
> ~~~

### 格式化郵件訊息
如果是支援郵件寄送的通知，你應該在該類別中定義好toMail方法。該方法接受$notifiable實例並返回一個<br/>
Illuminate\Notifications\Messages\MailMessage實例。郵件訊息可能包含多行文字及 "call to action"連結 等，讓<br/>我們看看一個 toMail 方法的範例：
```
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```
在上面這個範例中，我們定義了一個問候語、一行文字、一個call to action類型的連結及另一行文字。這些方法都是
MailMessage物件所提供，可以方便我們格式化要傳送的郵件內文格式。而郵件頻道稍後會將訊息元件轉換成可讀且
為響應式的 HTML 文本。以下就是透過郵件頻道建立的郵件範例：
![Alt text](https://laravel.com/assets/img/notification-example.png "郵件頻道建立的郵件範例")

#### 其他通知格式選項
除了在通知類別內使用line方法一行行的定義文字，你也可以改用 view 方法指定自訂模板用來渲染通知信件。
```
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```
toMail方法也可以改返回一個Mailable的物件，這讓原本定義好的Mailable也可以透過通知的郵件頻道傳送。
```
use App\Mail\InvoicePaid as Mailable;

/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return Mailable
 */
public function toMail($notifiable)
{
    return (new Mailable($this->invoice))->to($this->user->email);
}
```

#### 錯誤訊息
有些通知是用來提醒使用者某些錯誤發生，像是付款失敗。你可以在建立通知訊息時利用 error 方法指出信件是與相關<br/>
錯誤有關係的，內文就會改以紅色字體來呈現。
```
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Message
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('Notification Subject')
                ->line('...');
}
```

### 自訂收件者
當通知是透過mail頻道來發送時，通知系統會自動從可通知實體中找尋email屬性。你也可以透過在實體內定義名為<br/>
routeNotificationForMail 的方法用來自訂寄送的 email 位置。
```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForMail($notification)
    {
        return $this->email_address;
    }
}
```

### 客製郵件主題
預設郵件標題就是通知類別的名稱並經格式化後的字串。例如，通知類別名稱為 InvoicePaid，則該信件的預設標題就<br/>
會是Invoice Paid。所以，如果想要指定一個明確的郵件標題，可以在建立訊息時調用 subject 方法。
```
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('Notification Subject')
                ->line('...');
}
```

### 客製郵件模板
你可以透過發佈通知套件的資源來修改信件通知使用的 HTML 和純文字模板。在使用指令發佈套件後，通知<br/>
模板就會被放到 resources/views/vendor/notifications 這個目錄下。
```
php artisan vendor:publish --tag=laravel-notifications
```

## Markdown郵件通知
使用 Markdown 格式的郵件通知可享有利用預先構建郵件通知模板的好處，可以更加彈性地撰寫較長的自訂郵件內文。<br/>
由於訊息採用 Markdown 格式撰寫，Laravel 能協助你將訊息渲染成美觀且具響應式的 HTML 模板，同時也能自動的<br/>
產生相應的純文字格式。

### 產生訊息
你可以使用 Artisan 的 make:notification 指令並附加 --markdown 選項，即可產生通知類別的同時建立對應的 Markdown<br/>
通知模板。
```
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```
就像其它Mail通知一樣，使用Markdown模板的郵件通知也需要定義toMail方法。然而，我們不再需要使用諸如line<br/>
和action方法，而是使用markdown方法指定使用的Markdown模板名稱。
```
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

### 撰寫訊息
Markdown 郵件通知搭配使用了 Blade 元件及 Markdown 標記語言，這使你可以更輕鬆建構郵件通知。同時還能使用<br/>
Laravel 內建的方法。
```
@component('mail::message')
# Invoice Paid

Your invoice has been paid!

@component('mail::button', ['url' => $url])
View Invoice
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

### 常用元件
#### 按鈕元件
該元件會建構一個置中的按鈕連結，並接受url與color兩種參數來定義按鈕的樣式。其中，顏色部分支援紅、綠、藍三種<br/>
顏色。
```
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Invoice
@endcomponent
```

#### 面板元件
該元件用來渲染一個文字區塊，區塊會跟Markdown郵件通知有著不同的背景顏色。這個區塊讓你用來在郵件中建立一塊<br/>
更吸引人注意的文字區塊。
```
@component('mail::panel')
This is the panel content.
@endcomponent
```

#### 表格元件
該元件用以將Markdown表格轉換成Html表格。該元件直接受一個Markdown表格的內容，並且支持使用預設的 Markdown<br/>
表格對齊符號來對齊表格欄位。
```
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

### 客製化元件
你可以將內建的Markdown元件匯出到應用程式並予以客製修改。若你要再發佈該元件，可以使用 Artisan 的 vendor:publish<br/>
指令並附加標籤 laravel-mail 以發佈你的客製化元件。
```
php artisan vendor:publish --tag=laravel-mail
```
上述指令會將Markdown郵件元件發佈到resources/views/vendor/mail這個目錄下，該目錄下會包含了html與markdown<br/>
兩個子目錄。各子目錄則包括個別元件對應的獨立樣式，你可以依照你的需求任意的客製這些元件。

#### 客製CSS
在匯出元件後，resources/views/vendor/mail/html/themes這個目錄就會含有一個預設的樣式定義default.css，你可以<br/>
自訂這個檔案內的 CSS 樣式，該樣式會自動的套用在 Markdown 通知內的 HTML 中。
> ~~~
> 如果你想為 Markdown 元件建立全新的樣式，你可以輕易的在 html/themes 目錄內撰寫新的 CSS 檔案，並且更改
> mail 設定檔案內的 theme 選項。
> ~~~


## 資料庫通知
### 先決條件
database通知頻道會將訊息儲存在資料庫的表格中。該資料表包含了像是通知類型及JSON資料等用來描述通知的資訊。<br/>
也因此你可以透過資料表查詢以在使用者應用程式介面中顯示通知。但在你這麼做之前，你必須先建立好這些資料表格。<br/>
你可以透過Artisan的notifications:table這個指令來建立該表的遷移檔。
```
php artisan notifications:table

php artisan migrate
```

### 格式化資料庫通知
若是使用database這個通知頻道，你應該要在該通知類別中定義 toDatabase 及 toArray 兩種方法。這些方法可接受<br/>
$notifiable實體並會返回一個 PHP 陣列，而該陣列將被編碼成 JSON 格式接著存放在 notifications 資料表中的 data<br/>欄位中。下面是一個toArray的例子。
```
/**
 * Get the array representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

#### toDatabase VS toArray
toArray 方法同時也會被 broadcast 廣播頻道用來判斷要推播哪些資料到Javascript用戶端。若需要有兩種不同的陣列<br/>來支援 database 和 broadcast 兩種頻道的訊息格式化，你必須改定義一個 toDatabase 方法來支援資料庫通知的格式<br/>
化，而不是 toArray 方法。

### 存取通知
一旦通知訊息是存放在於資料庫中，你會需要一個方便的方法從可通知實例來存取它們。 Illuminate\Notifications\Notifiable<br/>
Trait 預設就被包含在 Laravel 的 App\User 模型，其包含了一個 notifications 的 Eloquent 關聯模型，可藉此回傳<br/>通知實例。為了獲取這些通知，你可以像其它 Eloquent關聯一樣地存取它們。預設，通知則會用 created_at 時間戳記<br/>
來進行排序。
```
$user = App\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```
如果你想取得是未讀取的通知，可以改用**unreadNotifications**這關聯模型，同樣地該模型取得的通知實例是以<br/>
created_at 時間戳記來進行排序。
```
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

### 標示通知為已讀
通常你會希望將使用者看過的通知訊息標記成已讀，Illuminate\Notifications\Notifiable Trait 提供了一個<br/>
markAsRead 方法可以用來更新資料表內通知紀錄的read_at欄位。
```
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```
然而，除了以迴圈的方式去標記每則通知為已讀，還可以直接在通知集合物件上直接調用markAsRead 方法予以<br/>
標記這些通知。
```
$user->unreadNotifications->markAsRead();
```

你可能也需要使用Mass Update的方式來標記所有通知為已讀而無需從資料庫中取得這些資訊實例後才進行操作。
```
$user = App\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```
當然，你也能透過 delete 方法將這些通知從資料庫中完整的刪除。
```
$user->notifications()->delete()
```

## 廣播通知
### 先決條件
在開始廣播通知前，你需要配置好Laravel提供的事件廣播服務，事件廣播提供了一個方法讓你能夠從JavaScript客戶端<br/>
與伺服器端的觸發事件進行互動。

### 格式化廣播通知
經廣播頻道的通知方式使用的是 Laravel 的事件廣播服務，能夠讓你的 JavaScript 用戶端可以即時的捕捉到通知<br/>
訊息。如果你的通知支援廣播通知，就需要在通知類別內部定義一個toBroadcast方法，該方法同樣接受一個$notifiable<br/>
實例，然後會返回一個BroadcastMessage實例。回傳的資料會使用 JSON 格式傳遞到 JavaScript 用戶端。讓我們來看<br/>
一個使用  toBroadcast 方法的範例。
```
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return BroadcastMessage
 */
public function toBroadcast($notifiable)
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

#### 廣播佇列設定
所有的廣播訊息都會直接採用佇列的方式進行廣播。如果你想要設定進行廣播操作時使用的佇列連線或佇列名稱，<br/>
可以使用 BroadcastMessage 的 onConnection 和 onQueue 方法。
```
return (new BroadcastMessage($data))
                ->onConnection('sqs')
                ->onQueue('broadcasts');
```
> ~~~
> 除了指定的訊息之外，廣播通知也會使用一個  type 欄位來儲存通知的類別名稱。
> ~~~

### 監聽通知
通知會透過以 **{notifiable}.{id}** 這個慣用的表達方式在私人頻道上進行廣播。所以，如果你想寄送通知到ID為1的<br/>
使用者身上，該通知就會使用 **App.User.1** 這個私人頻道進行傳遞。當使用了 **Laravel Echo**，你可以更輕鬆的<br/> 利用輔助方法 notifications 來進行該頻道的監聽。
```
Echo.private('App.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

#### 自訂通知頻道
如果想自訂頻道讓可通知實體能在上頭接收廣播通知，請在可通知實體內定義一個 receivesBroadcastNotificationsOn<br/>
的方法。
```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     *
     * @return string
     */
    public function receivesBroadcastNotificationsOn()
    {
        return 'users.'.$this->id;
    }
}
```

## 簡訊通知
### 先決條件
在Laravel使用SMS簡訊通知是透過Nexmo來提供支援。在透過 Nexmo 傳送通知前，你必須先用 Composer 安裝<br/>
nexmo/client 套件並在config/services中加上對應的設定。範例設定如下：
```
'nexmo' => [
    'key' => env('NEXMO_KEY'),
    'secret' => env('NEXMO_SECRET'),
    'sms_from' => '15556666666',
],
```
其中，sms_from 選項是用來設定SMS 訊息是由哪個電話號碼進行傳送。你需要在 Nexmo 的主控台介面中來產生這樣<br/>
一組電話號碼。

### 格式化SMS簡訊通知
若通知是利用 SMS 方式進行傳送，你需要在通知類別中定義 toNexmo 方法。該方法也接受一個 $notifiable 實體並且<br/>
會回傳一個 Illuminate\Notifications\Messages\NexmoMessage 實例。
```
/**
 * Get the Nexmo / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content');
}
```

#### Unicode 萬國碼內容
如果簡訊訊息中包含了Unicode的字元，則必須在建構NexmoMessage實例時調用unicode方法
```
/**
 * Get the Nexmo / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your unicode message')
                ->unicode();
}
```

### 自訂 "FROM" 號碼
如果寄送簡訊通知時需要使用有別設config/services配置以外的電話號碼，你必須在NexmoMessage實例上調用from方法<br/>
予以指定。
```
/**
 * Get the Nexmo / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content')
                ->from('15554443333');
}
```

### 路由轉送SMS簡訊通知
當使用 nexmo 頻道寄送通知時，預設通知系統是先找尋可通知實體中的phone_number屬性，如果想要自訂可通知<br/>
實體獨立使用的電號號碼，可在可通知實體中定義routeNotification方法。
```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Nexmo channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForNexmo($notification)
    {
        return $this->phone;
    }
}
```

## Slack通知
### 先決條件
在開始使用Slack傳送通知時，必須先使用Composer來安裝Guzzle HTTP library
```
composer require guzzlehttp/guzzle
```

你還會需要為你的Slack團隊設置一個"Incoming Webhook"的整合。這項整合會提供你一個可能用來路由Slack通知的URL。


### 格式化Slack通知
若要讓通知訊息支援由 Slack 傳遞，你可以在你的通知類別中定義一個 toSlack 方法。該方法接受一個可通知實體然後<br/>
返回一個Illuminate\Notifications\Messages\SlackMessage實例。Slack訊息可包含的內容除純文字之外還可加上附件<br/>
，一種含有額外格式化的文字或陣列數據。
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->content('One of your invoices has been paid!');
}
```
上述範例中僅僅是傳送一行文字給Slack，其樣式看起來就像下面所示：<br/>
![Slack訊息通知範例一](https://laravel.com/assets/img/basic-slack-notification.png "Slack訊息通知範例一")

#### 自定傳送者及接收者
你可以使用 **from** 及 **to** 兩種方法來定義傳送者與接收者。from方法可以接受使用者名稱或emoji表情符號；<br/>
to方法則接受頻道或使用者名稱。
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Ghost', ':ghost:')
                ->to('#other')
                ->content('This will be sent to #other');
}
```
你也可以使用一張圖片作為訊息的 Logo 而不是使用 emoji 符號：
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Laravel')
                ->image('https://laravel.com/favicon.png')
                ->content('This will display the Laravel logo next to the message');
}
```

### Slack 附件類型
#### 格式化文字
你也可以在Slack訊息上加上附件，比起簡單文字訊息，附件可以提供更為豐富的格式化選擇。下面範例中，我們會<br/>
寄送一個有關於應用程式發生例外的錯誤通知，裡頭包含了一個關於例外發生原因的詳細資訊連結。
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was not found.');
                });
}
```
上述範例所產生的Slack訊息會像是下面這張圖一樣：<br/>
![Slack 訊息範例二](https://laravel.com/assets/img/basic-slack-attachment.png "Slack 訊息範例二")

#### 陣列數據
附件同時允許你可以指定一個要傳遞給使用者的陣列數據，這些數據會使用表格式的方式呈現，以其更容易閱讀
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/invoices/'.$this->invoice->id);

    return (new SlackMessage)
                ->success()
                ->content('One of your invoices has been paid!')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Invoice 1322', $url)
                               ->fields([
                                    'Title' => 'Server Expenses',
                                    'Amount' => '$1,234',
                                    'Via' => 'American Express',
                                    'Was Overdue' => ':-1:',
                                ]);
                });
}
```
下列為上述範例的Slack訊息樣式圖片：<br/>
![Slack 訊息範例三](https://laravel.com/assets/img/slack-fields-attachment.png "Slack 訊息範例三")

#### Markdown 附件內容
如果在附件欄位中包含Markdown的格式內容，可以使用markdown這個方法來指示Slack解析Markdown並顯示其<br/>
代表的內容。該方法可接受的值有 pretext、 text、 和 / 或 fields，更多關於Slack附件的可用格式請參見[Slack API documentation](https://api.slack.com/docs/message-formatting#message_formatting)<br/>
```
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was *not found*.')
                               ->markdown(['text']);
                });
}
```

### 路由轉送Slack訊息
要將Slack通知路由到適當的位置，你必須在你的可通知實體內定義一個routeNotificationForSlack方法。該法應該<br/>
返回一個通知應該要發送至的webhook URL。而Webhook URLs 則可透過在你的 Slack team 內新增一個<br/>
"Incoming Webhook" 服務：
```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForSlack($notification)
    {
        return $this->slack_webhook_url;
    }
}
```

## 通知事件
當通知被寄送以後，Illuminate\Notifications\Events\NotificationSent這個事件就會被觸發，其包含了一個可通知實體
及通知實例。你可以在 EventServiceProvider 中為該事件註冊監聽器。
```
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Notifications\Events\NotificationSent' => [
        'App\Listeners\LogNotification',
    ],
];
```

在你的 EventServiceProvider 註冊監聽器後，接著可用 event:generate Artisan 指令快速產生監聽器類別。<br/>

而在監聽器內，你可以透過存取事件的notifiable、notification 和 channel屬性，以獲取更多通知接收對象或通知本身的資訊。
```
/**
 * Handle the event.
 *
 * @param  NotificationSent  $event
 * @return void
 */
public function handle(NotificationSent $event)
{
    // $event->channel
    // $event->notifiable
    // $event->notification
}
```

## 客製頻道
Laravel 自帶了幾種通知型頻道，但如果你想要自行定義經由其它頻道傳送通知的驅動，可以定義一個包含了send方法<br/>
的類別。該方法需要接收可通知實體與通知兩種參數。
```
<?php

namespace App\Channels;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     *
     * @param  mixed  $notifiable
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return void
     */
    public function send($notifiable, Notification $notification)
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```
一旦你定義好你的通知頻道類別，你就可以輕鬆透過任意通知實例的 via 方法來回傳你的類別名稱。
```
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use App\Channels\VoiceChannel;
use App\Channels\Messages\VoiceMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     *
     * @param  mixed  $notifiable
     * @return array|string
     */
    public function via($notifiable)
    {
        return [VoiceChannel::class];
    }

    /**
     * Get the voice representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return VoiceMessage
     */
    public function toVoice($notifiable)
    {
        // ...
    }
}
```