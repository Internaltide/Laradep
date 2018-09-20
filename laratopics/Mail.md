# Laravel Mail

> Laravel 挑選了 SwiftMailer 做為它底層的郵件服務基礎實作類別，除了基本的SMTP遠端連接、<br/>
> PHP Mail Function、Sendmail的本地連接外，還可以直接串接目前一些常見的線上郵件寄送服務，<br/>
> 諸如，MailGun、SparkPost、Amazon SES。讓我們可以快速為應用程式提供郵件寄送的功能。<br/>

## 先決條件
若是選用雲端的郵件服務做為應用的解決方案，通常是基於API形式驅動的，需要額外的函示庫來提供，<br/>
一般安裝 Guzzle HTTP 函式庫是必要的。
```
composer require guzzlehttp/guzzle
```
或者，在composer.json加入下列一行後，再使用composer update進行更新
```
"guzzlehttp/guzzle": "~5.3|~6.0"
```

### MailGun Driver
由Guzzle Http來提供驅動，並於config/mail與config/services內部進行配置
```
// 改用mailgun driver，預設是smtp
'driver' => env('MAIL_DRIVER', 'mailgun'),
```
```
// 設定其在config/services的專屬配置
'mailgun' => [
    'domain' => 'your-mailgun-domain',
    'secret' => 'your-mailgun-key',
],
```

### SparkPost Driver
由Guzzle Http來提供驅動，並於config/mail與config/services內部進行配置
```
// 改用sparkpost driver，預設是smtp
'driver' => env('MAIL_DRIVER', 'sparkpost'),
```
```
// 設定其在config/services的專屬配置
'sparkpost' => [
    'secret' => 'your-sparkpost-key',

    // 如果需要的話，還可以設定使用的API端點
    'options' => [
        'endpoint' => 'https://api.eu.sparkpost.com/api/v1/transmissions',
    ],
],
```

### SES Driver
需使用官方提供的SDK，在composer.json的require區段加入以下一行後，再使用composer update進行更新
```
"aws/aws-sdk-php": "~3.0"
```
接著，於config/mail與config/services內部進行配置
```
// 改用ses driver，預設是smtp
'driver' => env('MAIL_DRIVER', 'ses'),
```
```
// 設定其在config/services的專屬配置
'ses' => [
    'key' => 'your-ses-key',
    'secret' => 'your-ses-secret',
    'region' => 'ses-region',  // e.g. us-east-1
],
```

## 建立 Mailables
Laravel框架中，每一種寄送的郵件類型都對應一個mailable類型的class。這些Mailable Class檔案<br/>
都會被放置在app/Mail目錄下面，如果不見該目錄，則再執行make:mail的artisan指令後就會自動<br/>
建立了。該指令即用來建立初始的Mailable Class。
```
php artisan make:mail OrderShipped
```

## 編輯 Mailables
Mailable Class的所有初始設定都需在**build**方法內完成。在該方法內可以再用以下之內部方法<br/>
設定郵件寄送前所需要的資訊。
 - form
 - subject
 - view
 - attach

 ### 設定寄件者
 #### 於build方法內使用from內部方法
 ```
 /**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->from('example@example.com')
                ->view('emails.orders.shipped');
}
 ```
 #### 設定全域寄件者
在config/mail中所設定的from就是全域設定，當Mailable中沒有指定from時，就會使用設定檔的from
 ```
 'from' => ['address' => 'example@example.com', 'name' => 'App Name'],
 ```

 ### 設定郵件視圖

 #### Html郵件格式
 典型的，Laravel仍然是使用Blade來渲染郵件的呈現。我們可以在Mailable類別的build內部使用
 view來設定使用的視圖。
 ```
 /**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped');
}
 ```
 > 一般典型作法，我們會在resources/views下建立emails目錄來存放這些郵件視圖，<br/>
 > 然而，這並非強制，我們仍可以在resource/views下的任意位置置放這些視圖<br/>

 #### 純文字郵件格式
若您的郵件是屬於純文字格式，可以改用text方法來指定使用的模板。<br/>
當然，你還可以同時指定Html與PlainText兩種格式。
```
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->text('emails.orders.shipped_plain');
}
```
PS. 同時指定的好處是? 在寄送時可以選擇? 待測試...

### 視圖資料
#### 透過public屬性
Mailable物件的public屬性被設計成可做為視圖資料。因此，透過將資料傳入<br/>
建構式中，再設定給Mailable物件的public屬性。
```
class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * The order instance.
     *
     * @var Order
     */
    public $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    public function build()
    {
        return $this->view('emails.orders.shipped');
    }
}
```
一經設定為public屬性後，就可以在視圖中使用該資料
```
<div>
    Price: {{ $order->price }}
</div>
```

#### 透過with方法
當你需要傳遞特定格式的資料到視圖時，就可以善用with方法。一樣地，將資料傳進建構式中，<br/>
然後設定給類別屬性，但該屬性不能是public以避免自動生效為試圖資料。之後，對資料進行<br/>
處理後再透過with方法將資料帶進視圖中。
```
class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * The order instance.
     *
     * @var Order
     */
    protected $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->with([
                        'orderName' => $this->order->name,
                        'orderPrice' => $this->order->price,
                    ]);
    }
}
```
一經設定後，就可以在視圖中用不同的方式合併該資料
```
<div>
    Price: {{ $orderPrice }}
</div>
```

### 郵件附件

#### 一般附件
**附加附件**<br/>
在build方法中使用attach來附加附件到郵件中，該方法接受附件檔案的完整路徑做為第一參數。<br/>
```
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file');
}
```
**指定附件MIME Type與顯示名稱**<br/>
該方法藉由第二參數來指定附件的顯示名稱與Mime Type。<br/>
```
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file', [
                    'as' => 'name.pdf',
                    'mime' => 'application/pdf',
                ]);
}
```

#### 原始數據附件(Raw Data Attachment)
使用attachData方法附加RawData類型的附件，例如建立的PDF的資料還是以RawData存在記憶體時，<br/>
就可採用attachData方法來將記憶體中的資料附加到郵件做為附件。該方法接受RawData做為第一參數，<br/>
附件名稱做為第二參數，可選的第三參數則用來指定附件的MIME Type等屬性。<br/>
```
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attachData($this->pdf, 'name.pdf', [
                    'mime' => 'application/pdf',
                ]);
}
```

### 內嵌圖片
直接於信件內文安插圖片會讓郵件本體顯得很笨重，影響郵件開啟速度。但隨然如此，Laravel還是提供了<br/>
便捷的方法來處理這類的任務並自動取得該圖檔的Cid(Content-Id)。使用方式為在Blade視圖中透過調用$message<br/>
物件的embed方法來嵌入圖檔。
```
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToFile) }}">
</body>
```
PS. 在郵件視圖中，$message訊息物件會自動產生，所以無需刻意手動傳遞。唯一要注意的是$message<br/>
       不可以用在markdown message中。<br/>

### 內嵌原始數據附件
Blade視圖中，透過$message物件調用embedData方法
```
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, $name) }}">
</body>
```

### 自訂SwiftMailer訊息
在Mailable透過withSwiftMessage註冊一個回調方法，該回調方法會在郵件被寄送出去前被調用，<br/>
而傳入的方法的參數即SwiftMailer Message實例。故在回調方法內，我們可以重新調整發送的訊息。
```
public function build()
    {
        $this->view('emails.orders.shipped');

        $this->withSwiftMessage(function ($message) {
            $message->getHeaders()
                    ->addTextHeader('Custom-Header', 'HeaderValue');
        });
    }
```

## Markdown 格式的 Mailables
Artisan指令建立的初始Mailable預設並不包含Markdown模板。若需要Markdown模板，可以為指令附加<br/>
--markdown選項。
```
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```
接者，Mailable的build方法內，改用markdow方法取代原本的view，並載入指令生成的Markdown模板。
```
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->from('example@example.com')
                ->markdown('emails.orders.shipped');
}
```

### 編寫Markdown格式的郵件
Markdown Mailable同時結合Blade跟Markdown兩種語法，同時也可以在內部使用Laravel預製的Markdown元件
```
@component('mail::message')
# Order Shipped

Your order has been shipped!

@component('mail::button', ['url' => $url])
View Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```
> 在撰寫Markdown電子郵件時，請勿使用多餘的縮進。<br/>
> Markdown解析器將縮進內容呈現為代碼塊。

#### 按鈕元件
該元件會建立一個置中的連結按鈕，可接受url跟color兩個參數，支援的顏色有blue、green、red
```
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Order
@endcomponent
```

#### 面板元件
該元件會建立一個顏色略不同於背景的文字屏幕，予以提供一個引人注意的提醒區塊
```
@component('mail::panel')
This is the panel content.
@endcomponent
```

#### 表格元件
該元件會提供將Markdown Table轉換為Html Table的功能
```
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

### 自訂Markdown元件
#### 匯出
匯出Laravel的Markdown郵件元件到應用程式中
```
php artisan vendor:publish --tag=laravel-mail
```
#### 客製內容
匯出的元件，預設會放在resources/views/vendor/mail目錄下。其內又有html跟markdown兩個子目錄，<br/>
分別置放各自html格式與markdown plain-text格式的文件。然後就可以針對這些文件進行客製。<br/>
#### 客製CSS
匯出元件後，resources/views/vendor/mail/html/themes目錄下就會出現一個default.css的文件。<br/>
你可以在該文件內進行CSS的客製，自訂的樣式會自動被嵌入郵件。

### 全新替換Markdown元件樣式
如果你要的是完全地替換掉Markdown元件的樣式，必需到html/themes目錄下撰寫一個全新的CSS，<br/>
並到config/mail設定檔中替換theme選項的設定值。

## 寄送郵件
要將郵件寄出前，我們必須先用Mail Facade的方法to來設定郵件的收件人，該法可接受單個mail address、<br/>
user實例或user物件集合。當傳遞的是user物件或其集合時，Laravel Mailer會自動使用物件的email跟name屬性<br/>
做為寄件人資訊。最後在調用Mail Facade的方法send來將郵件寄出
```
<?php

namespace App\Http\Controllers;

use App\Order;
use App\Mail\OrderShipped;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;
use App\Http\Controllers\Controller;

class OrderController extends Controller
{
    /**
     * Ship the given order.
     *
     * @param  Request  $request
     * @param  int  $orderId
     * @return Response
     */
    public function ship(Request $request, $orderId)
    {
        $order = Order::findOrFail($orderId);

        // Ship order...

        Mail::to($request->user())->send(new OrderShipped($order));
    }
}
```
當然，我們還可以定義其他收件人類型，如副本BC與密件副本BCC
```
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

## 渲染Mailable
有時候我們只想捕抓出郵件的內容而無須寄送。此時，可以調用mailable的render方法，<br/>
該方法會以字串型態回傳該郵件的內容。
```
$invoice = App\Invoice::find(1);

return (new App\Mail\InvoicePaid($invoice))->render();
```

## 在瀏覽器預覽郵件
Laravel提供了一個非常方便的方法，只要在路由的閉包中回傳Mailable實例，就可以直接在<br/>
瀏覽器上顯示出郵件的內容與樣貌，無須做任何額外的處理。
```
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

## 將郵件加入佇列
### 加入佇列
和寄送郵件的方式雷同，只需將原本調用的send方法改成queue，如此郵件就會被加入倒佇列等待寄送。<br/>
這樣的方式可以避免因直接寄送造成網頁反應時間大幅加長。
```
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```
### 延遲的訊息佇列
透過調用later方法，可以將郵件加入佇列，並且還可以自行指定延遲時間。郵件將不再是根據佇列<br/>
排隊的結果而是改採指定的延遲時間，一但超過了延遲時間就會寄發出去。
```
// 方法later的第一參數$when接受DateTime實例
$when = now()->addMinutes(10);

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later($when, new OrderShipped($order));
```
### 加入特定佇列
所有通過make:mail產生的Mailable實例都會使用Illuminate\Bus\Queueable這個Trait。所以，可以<br/>
利用onQueue及onConnection先指定好要加入佇列的連線與名稱後，再調用queue或later。
```
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

### 預設就加入佇列
若存在預設行為就是要加入佇列的Mailable，那可以讓那些Mailable去實作ShouldQueue契約，<br/>
如此一來，即使指是調用send方法來寄送郵件，Laravel仍然會將郵件加入佇列去。
```
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

## 本機端的開發的郵件寄送
有時應用程式仍處於開發階段，我們並不想讓郵件真的寄往實際郵件位址，進而影響使用者。<br/>
Laravel提供了幾個方式來處理這類的狀況，你可以不用動原本程式的寄送邏輯又可以擁有變相的處理行為。
### 日誌驅動
透過設定檔配置，讓Laravel將郵件內容改寫到log file裡頭去，取代真實的寄送。

### 通用收件者
透過在config/mail設定通用的收件者，讓所有郵件寄到通用收件者而不是實際收件者
```
'to' => [
    'address' => 'example@example.com',
    'name' => 'Example'
],
```
PS. 預設to區塊並不存在，所以上線前要拿掉? 待測試...

### 郵件陷阱(Mailtrap)
使用Mailtrap這個網路服務，改用SMTP驅動連線到Mailtrap的主機。寄送出去的Email<br/>
並不會真的寄到收件者哪邊去，而是被Mailtrap捕抓下來。
```
MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=Your Mailtrap Username
MAIL_PASSWORD=Your Mailtrap Password
MAIL_ENCRYPTION=Your Mailtrap Encryption(ex. tls))
```

## 事件
Laravel為郵件寄送提供了兩種事件觸發行為，而該事件僅會在郵件實際寄送時被觸發，<br/>
加入佇列階段並不會觸發。<br/>
 - MessageSending - 郵件寄送前被觸發<br/>
 - MessageSent - 郵件寄送後被觸發<br/>
可以在EventServiceProvider內進行事件監聽的註冊行為
```
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Mail\Events\MessageSending' => [
        'App\Listeners\LogSendingMessage',
    ],
    'Illuminate\Mail\Events\MessageSent' => [
        'App\Listeners\LogSentMessage',
    ],
];
```

## 補充
**Cid**
> 一封郵件由Headers和郵件本體組成，Header中包括了各種屬性，其中就有Cid，即Content-Id。<br/>
>  該Cid為內嵌資源的唯一辨識碼，供HTML格式正文可利用該id來取得並引用內嵌資源。<br/>
> 舉例，一個Content-ID: it315logo_gif即在HTML文本對應&lt;img src="it315logo_gif"&gt;