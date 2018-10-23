# Laravel Logging


> ~~~
> 為了讓你更了解應用程式發生了那些事情，Laravel提供了一個日誌服務，讓你可以將日誌訊息
> 寫入到檔案、系統Log，甚至於傳遞到Slack來通知整個專案團隊。在底層，Laravel是使用了
> Monolog 這個函式庫來提供多種不同類型的日誌處理行為。在Laravel為這些日誌驅動進行配置
> 是很簡單的，也允許混搭各種選項以客製自己的日誌處理邏輯。
> ~~~

## 配置
應用程式的日誌系統配置都放在config/logging設定檔裡面。在裡頭，你可以藉由各選項的配置來<br/>
定義自己的日誌頻道。預設，當在寫入日誌訊息時，Laravel是使用 **stack** 這個頻道，而該頻道<br/>
主要將多個日誌頻道聚集成單一個頻道，更多資訊會在後面提及。

### 設定頻道名稱
預設的頻道名稱會與當前環境匹配，如production或local。若要變更這個名稱，必須在頻道配置區塊<br/>
內額外添加 **name** 這個設定項。
```
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

### 可用的日誌頻道
| 頻道 | 描述 |
|--------|-------|
| stack | 一個包裹多頻道的日誌頻道 |
| single | 單一檔案或路徑的日誌頻道 |
| daily | 一個基於檔案的日循環之日誌頻道 |
| slack | Slack 日誌頻道 |
| syslog | 系統日誌之日誌頻道 |
| errorlog | 錯誤日誌之日誌頻道 |
| monolog | 一種Monolog型的工廠驅動，可使用任意支援Monolog的處理程序 |
| custom | 調用指定工廠以建立所需的頻道驅動 |

### 配置Slack頻道
Slack日誌頻道的必要設定項目為 **url**，該選項設定值必須與slack所設定的incoming webhook<br/>
一致。所謂的incoming webhook就是用來將外部訊息傳遞進Slack的一種方法。詳細可見[官方說明](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)

### 建立日誌堆疊
如同前面所述，stack驅動允許你將多個頻道合併成單一頻道。為了描繪如何使用該驅動，讓我們<br/>
看一下可能在線上環境出現的範例配置設定。
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
讓我們來剖析這一份配置，首先，該配置透過channels設定項聚集了其他兩種頻道syslog和slack。<br/>
所以，記錄訊息時，兩個頻道都有機會記錄到該訊息。<br/>
(會說只是有機會的原因是後面提到的層級因素，頻道是否記錄訊息還得透過比較訊息本身與頻道
設定的層級)

### 日誌層級
特別注意到上面範例中，syslog 與 slack配置區塊中的level設定項，該設定項用來決定記錄訊息時<br/>
的最低層級。Monolog為Laravel提供了符合[RFC 5424 規格](https://tools.ietf.org/html/rfc5424)<br/>
的日誌層級，依序由高到低：**emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, 及 **debug**。<br/>

想像一下我們使用了Log Facade 的 debug方法來記錄訊息，配合上面的配置範例，因為層級的關係，<br/>
該debug訊息只會記錄到syslog這個頻道。
```
Log::debug('An informational message.');
```

若改成記錄一條emergency層級的訊息，則兩個頻道都會進行記錄。
```
Log::emergency('The system is down!');
```

## 日誌訊息寫入
依前例，我們知道可以透過Log Facade來處理日誌的寫入，根據RFC 5424的標準，框架日誌處理<br/>
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

### 上下文情境資料
你可以用陣列型態將上下文情境資料傳遞給記錄訊息的方法，該資料會被格式化並且與訊息<br/>
一併顯示。
```
Log::info('User failed to login.', ['id' => $user->id]);
```

## 寫入特定日誌頻道
有時候你想讓訊息記錄到配置檔預設頻道以外的頻道時，你可以先調用Log Facade的channel 方法<br/>
來切換至任何一個有定義在config/logging中的頻道，接著調用你要的層級記錄方法。
```
Log::channel('slack')->info('Something happened!');
```

若你想指定一個由數個日誌頻道組成的日誌堆疊(log stack)，可以改用stack方法來指定。
```
Log::stack(['single', 'slack'])->info('Something happened!');
```

## 進階的Monolog日誌頻道客製
### 為日誌頻道客製Monolog
有時為取得現有頻道的完全控制，你會想為頻道進行更進一步的配置方式，以控制更細部的日誌行為。<br/>
例如，你想為特定的日誌處理器配置一個Monolog FormatterInterface介面的實現。首先，你需要在<br/>
配置區塊內定義**tap**這個選項，設定值為一陣列並包含一系列類別，一系列在Monolog實例創建後，<br/>
可能被客製實現特定行為的類。
```
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```