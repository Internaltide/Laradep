# Laravel Logging


> ~~~
> 為了讓你更了解應用程式發生了那些事情，Laravel提供了一個日誌服務，讓你可以將日誌訊息
> 寫入到檔案、系統Log，甚至於傳遞到Slack來通知整個專案團隊。在底層，Laravel是使用了
> Monolog 這個函式庫來支援多種不同類型的日誌處理器。在Laravel為這些日誌驅動進行配置
> 是很簡單的，框架允許你混搭各種日誌處理器以客製自己的日誌處理邏輯。
> ~~~

## 配置
應用程式的日誌系統配置都放在config/logging設定檔裡面。在裡頭，你可以藉由不同選項的配置來<br/>
定義自己的日誌頻道。預設，當在寫入日誌訊息時，Laravel是使用 **stack** 這個頻道，而該頻道<br/>
被用來將多個日誌頻道匯聚成單一頻道，更多關於堆疊建立的資訊都會在後面陸續說明。

### 設定頻道名稱
預設的頻道名稱會使用與當前環境匹配的名字，如production或local。若要變更這個名稱，必須在頻道<br/>
配置區塊內額外添加 **name** 這個設定項。
```
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

### 可用的日誌驅動
| 名稱 | 描述 |
|--------|-------|
| stack | 可匯聚多頻道的日誌頻道 |
| single | 單一檔案、路徑的日誌頻道 |
| daily | 一個基於檔案的日循環之日誌頻道 |
| slack | Slack 日誌頻道 |
| syslog | 系統日誌之日誌頻道 |
| errorlog | 錯誤日誌之日誌頻道 |
| monolog | Monolog日誌驅動，可支援任意的原生Monolog內建處理程序 |
| custom | 調用指定工廠以建立所需的頻道驅動 |

### 配置Slack頻道
Slack日誌頻道的必要設定項目為 **url**，該選項設定值必須與slack所設定的incoming webhook<br/>
一致。所謂的incoming webhook就是用來將外部訊息傳遞進Slack的一種方法。詳細可見[官方說明](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)

### 建立日誌堆疊
如同前面所述，stack驅動允許將多個頻道匯聚成單一頻道。為了描繪如何使用日誌堆疊，讓我們看一下可能在線上環境出現的範例配置設定。<br/>
```
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => 'debug',
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => 'Laravel Log',
        'emoji' => ':boom:',
        'level' => 'critical',
    ],
],
```
讓我們來剖析這一份配置，首先，該配置透過channels設定項匯聚了其他兩種頻道syslog和slack。<br/>
所以，記錄訊息時，兩個頻道都有機會記錄到該訊息。<br/>
(會說是有機會的原因是後面所提及的層級因素，頻道是否記錄訊息需得透過訊息本身與頻道設定的層級
 比較才能決定)

 > ~~~
>原生Monolog是單個日誌服務(Logger)會有個唯一的頻道名稱且可擁有多個日誌處理器(Handler)並以堆疊
> 的資料格式來儲存。在Laravel的設計中卻似乎是每個配置的頻道都是一個Logger，而且都只配置一個處
> 理器，只有stack頻道才是一個真正擁有處理器堆疊的日誌頻道。
> ~~~

### 日誌層級
特別注意到上面範例中，syslog 與 slack配置區塊中的level設定項，該設定項用來決定記錄訊息時<br/>
的最低層級。Monolog為Laravel提供了符合[RFC 5424 規格](https://tools.ietf.org/html/rfc5424)的日誌層級，依序由高到低：<br/>
**emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, 及 **debug**。<br/>

想像一下我們使用了Log Facade 的 debug方法來記錄訊息，配合上面的配置範例，因為層級的關係，<br/>
該debug訊息只會傳遞到syslog這個頻道來記錄。
```
Log::debug('An informational message.');
```

若改成記錄一條emergency層級的訊息，則兩個頻道都會進行記錄。
```
Log::emergency('The system is down!');
```

## 日誌訊息寫入
依前例所述，我們知道可以透過Log Facade處理日誌的寫入，根據RFC 5424的標準，框架日誌處理<br/>
器也提供了相對應的八種記錄方法。
```
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```
因此，可以調用這些方法來記錄對應層級的訊息，該訊息會被寫入到config/logging設定檔的預設頻道中。
```
<?php

namespace App\Http\Controllers;

use App\User;
use Illuminate\Support\Facades\Log;
use App\Http\Controllers\Controller;

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
        Log::info('Showing user profile for user: '.$id);

        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

### 上下文資料
你可以利用陣列將上下文資料傳遞給記錄訊息的方法，該資料會被格式化並且與訊息一併顯示。<br/>
這即是一種為日誌訊息添加額外資訊的方式。
```
Log::info('User failed to login.', ['id' => $user->id]);
```

## 寫入特定日誌頻道
有時候你想讓訊息記錄到配置檔預設頻道以外的頻道時，你可以先調用Log Facade的channel 方法<br/>
來切換至任何一個有定義在config/logging中的頻道，接著調用你要的層級記錄方法。
```
Log::channel('slack')->info('Something happened!');
```

若你想建立一個由數個日誌頻道組成的日誌堆疊(log stack)，可以改用stack方法來指定。
```
Log::stack(['single', 'slack'])->info('Something happened!');
```

## Monolog日誌處理客製
### 為頻道客製其底層Monolog的行為
有時為取得現有頻道的完全控制，你會想為頻道進行更進一步的配置方式，以控制更細部的日誌行為。<br/>
例如，你想為某個日誌處理器配置一個Monolog FormatterInterface介面的實現。首先，你需要在配置<br/>
區塊內使用**tap**選項來包含一系列類別，一系列在Monolog實例創建後，可被用來客製Monolog實例<br/>
的類別( 這邊指的主要應該是Monolog的格式器Formatter與加工器Processor )。
```
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```

一但你在頻道中設置了tap選項，你就得著手定義用來客製Monolog實例的自訂類別，這個類別通常僅<br/>
需要一個可以接受 **Illuminate\Log\Logger** 實例的 **__invoke** 方法，該實例會被用來代理所有<br/>
底層Monolog實例方法的調用。
```
<?php

namespace App\Logging;

class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     *
     * @param  \Illuminate\Log\Logger  $logger
     * @return void
     */
    public function __invoke($logger)
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(...);
        }
    }
}
```
> 所有tap設定項內的類別都可以透過容器解析並在建構式依型別提示來注入依賴

## 創建Monolog型的日誌處理頻道
原生Monolog擁有多種可用的日誌處理器，一般狀況下，你需要的可能只是個僅僅擁有單一處理器的<br/>
日誌服務。此時，你可以使用monolog這個驅動來建立該日誌頻道。當你使用了monolog驅動後，就可以<br/>
使用handler這個設定項來指示要初始化的處理器類別。另外，若處理器建構式的參數傳遞則可以使用<br/>
handler_with來定義。
```
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'handler_with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

### Monolog 格式器
當使用了monolog驅動後，預設會套用 **LineFormatter** 這個格式器，然而，若要使用客製的日誌<br/>
處理格式器，則需使用formatter、formatter_with兩個設定項來定義。
```
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

如果你使用的日誌處理器可以自我提供格式化處理，則可以將formatter配置項設定成 **default**
```
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

## 透過工廠建立日誌頻道
如果你想要完全控制Monolog的配置與初始化進而建立一個客製的頻道，你應該使用custom這個驅動。<br/>
然後使用via這個設定項來定義被用來創建Monolog實例的工廠類別。
```
'channels' => [
    'custom' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```
一但定義了使用custom驅動，就得定義用來創建Monolog實例的類別，該類別僅需要一個用來返回<br/>
Monolog實例的方法 **__invoke**。
```
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * Create a custom Monolog instance.
     *
     * @param  array  $config
     * @return \Monolog\Logger
     */
    public function __invoke(array $config)
    {
        return new Logger(...);
    }
}
```
