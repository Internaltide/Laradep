# Laravel Notifications


> ~~~
> 除了支援郵件寄送外，Laravel也提供了其它數種不同頻道的寄送通知。諸如，SMS (via Nexmo)、Slack。通知也可以
> 被存放在資料庫中，以便在直接在網頁中顯示。一般而言，通知訊息通常都很短，主要用來通知你的使用者應用
> 應用程式發生了那些事。如果你在寫的是帳單管理的應用程式，或許會需要通過Email或SMS頻道來寄送一個發票支付
> 的通知給使用者。
> ~~~

## 創建通知
在 Laravel ，每種通知頻道都會使用一個類別來定義並且存放在app/Notifications目錄下。你可以make:notification<br/>
使用這個Artisan指令來建立。每個剛建立的通知類別預設都會包含一個via方法及數種訊息建構的方法(如toMail 或<br/>
 toDatabase)，用來將通知轉換成為特定頻道優化的訊息。
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
每個通知類別都有一個 via 方法用於判別要將通知寄送哪個頻道。換句話說，通知可以被寄送至  mail、database、<br/>
broadcast、nexmo 和 slack 頻道。
> 如果你想要使用其他的遞送頻道，像是 Telegram 或 Pusher，可以參考社群維護的 Laravel Notification Channels 網站。

via 方法接收了一個 $notifiable 實例，該實例屬於寄送通知的類別。你可以使用該實例指定要用於接收訊息<br/>
的頻道。
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
可以加入到佇列中，你可以透過在通知類別內實作 ShouldQueue 介面和引用 Queueable trait 來達到目的。如果你是透<br/>
過指令make:notification來建立通知類別，則該介面與Trait預設就已經被引用了。
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

一旦 ShouldQueue 介面被新增至通知類別內，你仍可以一如往常的呼叫通知類別。Laravel 會自動的偵測類別內的<br/>
 ShouldQueue 介面並自動將要遞送的通知加入到佇列。
 ```
 $user->notify(new InvoicePaid($invoice));
 ```

如果你需要延遲通知的寄送，可以鏈式調用delay方法。
```
$when = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($when));
```

### 按需要通知
有時你需要寄送通知到未以 "user" 類別儲存的使用者，所以無法藉由前面的方式寄發通知。你可以透過使用<br/>
Notification::route，就能夠在寄送通知前指定臨時的通知路由。
```
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```

## 郵件通知
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
除了在通知類別內使用line方法一行行的定義文字，你也可以改用 view 方法指定一個自訂的模板用來渲染通知信件。
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
toMail方法也可以改返回一個Mailable的物件
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

### 自訂收件者
### 客製郵件主題
### 客製郵件模板


## Markdown郵件通知
## 資料庫通知
## 廣播通知
## 簡訊通知
## Slack通知

## 通知事件
## 客製頻道