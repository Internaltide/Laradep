# Laravel 任務排程


> &nbsp;<br/>
> 在過往，你可能會直接在伺服器上使用Cron來處理任務排程。但這可能很快就變成噩夢，因為<br/>
> 程式設計師再也無法透過代碼管理，你必須有權限SSH進入伺服器才能添加新的任務排程。現在，<br/>
> Laravel指令排程器允許你流暢且直覺地定義你的任務排程。當你使用了Laravel指令排程器後，你<br/>
> 只需在Cron中設定一條任務排程即可。你的任務排程將會在app/Console/Kernel.php的schedule方法<br/>
> 來進行定義。<br/>
> &nbsp;

## 啟動指令排程器
當你使用了指令排程器後，你必須到Cron中添加下面這一條排程設定。如果你不知道如何<br/>
在cron建立排程，可以使用Laravel Forge來進行Cron管理。
```
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

## 定義任務排程
你可以在App\Console\Kernel的schedule方法定義所有的任務排程。讓我們看下面這個範例，<br/>
我們將使用閉包函數來定義一個任務，並使其在在每天午夜被調用，閉包中則定義了資料表資料的清除。
```
<?php

namespace App\Console;

use DB;
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [
        //
    ];

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->call(function () {
            DB::table('recent_users')->delete();
        })->daily();
    }
}
```

除了以閉包來定義排程任務外，還可以基於其他類型定義任務並加入排程：
#### 把PHP 的 invokable object加入排程
所謂invokable object即一個可被當成方法調用的簡易物件，其內部具有一個魔術方法__invoke，<br/>
當物件被調用時，內部的__invoke就會自動被調用。
```
$schedule->call(new DeleteRecentUsers)->daily();
```

#### 把Artisan指令加入排程
你可以使用command方法並傳遞指令名稱或類別名稱來調用一個Artisan指令並加入排程。
```
$schedule->command('emails:send --force')->daily();

$schedule->command(EmailsCommand::class, ['--force'])->daily();
```

#### 把佇列任務加入排程
job方法可用來把佇列任務加入排程。此方法提供一個便捷的方式來把任務加入排程，而不是使用<br/>
call方法建立一個可將任務加入到佇列的閉包。
```
$schedule->job(new Heartbeat)->everyFiveMinutes();

// Dispatch the job to the "heartbeats" queue...
$schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
```
PS. 老實說不太理解這個功能的實際用意，光看範例程式好像是定期將任務加入到佇列的行為，需要再研究。

#### 把Shell指令加入排程
exec方法可以用來對操作系統發出指令，如執行一個腳本
```
$schedule->exec('node /home/forge/script.js')->daily();
```

### 排程的頻率設置
| 方法 | 描述 |
|--------|-------|
| ->cron('* * * * *') | 使用與Linux Cron類似的設定與法來定義執行頻率 |
| ->everyMinute() | 每分鐘執行一次任務 |
| ->everyFiveMinutes() | 每五分鐘執行一次任務 |
| ->everyTenMinutes() | 每十分鐘執行一次任務 |
| ->everyFifteenMinutes() | 每十五分鐘執行一次任務 |
| ->everyThirtyMinutes() | 每半小時執行一次任務 |
| ->hourly() | 每小時執行一次任務 |
| ->hourlyAt(17) |於每個小時的第17分鐘執行一次任務 |
| ->daily() | 每天執行一次任務 |
| ->dailyAt('13:00') | 於每天的下午一點執行一次任務 |
| ->twiceDaily(1, 13) | 於每天的1點與13點各執行一次任務 |
| ->weekly() | 每週執行一次任務 |
| ->weeklyOn(1, '8:00') | 於每週一的8點執行一次任務 |
| ->monthly() | 每個月執行一次任務 |
| ->monthlyOn(4, '15:00') | 於每個月4號的15點執行一次任務 |
| ->quarterly() | 每季執行一次任務 |
| ->yearly() | 每年執行一次任務 |
| ->timezone('America/New_York') | 設定時區用 |

以上這些方法還可以搭配其他限制條件，藉以產生更精細的排程。例如，僅在一週的特定幾天執行排程。
```
// Run once per week on Monday at 1 PM...
$schedule->call(function () {
    //
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
```

### 其他額外的限制條件
| 方法 | 描述 |
|--------|-------|
| ->weekdays() | 限制只能在平常的工作日執行  |
| ->sundays() | 限制只能在週日執行  |
| ->mondays() | 限制只能在週一執行 |
| ->tuesdays() | 限制只能在週二執行 |
| ->wednesdays() | 限制只能在週三執行 |
| ->thursdays() | 限制只能在週四執行 |
| ->fridays() | 限制只能在週五執行 |
| ->saturdays() | 限制只能在週六執行 |
| ->between($start, $end) | 限制只能在定義的起始時間區間內執行 |
| ->when(Closure) | 限制只能在某條件估算為真時才執行 |

#### 時間區間約束
between方法可以用來限制任務只能在一天當中的某時間區間來執行
```
// 在7點到22點間，每小時執行一次
$schedule->command('reminders:send')
                    ->hourly()
                    ->between('7:00', '22:00');
```
unlessBetween方法可以用來排除一天當中不能執行任務的時間區間
```
// 在23點到4點這段時間區間外的時間，每小時執行一次
$schedule->command('reminders:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
```

#### 條件驗證約束
使用when這個方法限制只能在指定的條件估算為真時才執行任務
```
// 當閉包內返回True，則任務就會執行
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});
```

skip方法具有類似的行為，只是剛好與when方法相反。
```
// 當閉包內返回真True，則忽略不予執行該任務
$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});
```

### 時區設置
使用timezone方法可以設置任務執行時間所處時區
```
$schedule->command('report:generate')
         ->timezone('America/New_York')
         ->at('02:00')
```
> ~~~
> 注意!! 某些時區會使用日光節約時間，這可能導致你的任務被執行兩次或根本不執行，所以應該盡量
> 避免使用那些時區來安排任務執行。
> ~~~

### 避免任務重複執行
預設情況下，即使之前相同的任務實例尚未結束，排定的任務仍會執行。為了避免這個問題，可以使用<br/>
withoutOverlapping 方法
```
$schedule->command('emails:send')->withoutOverlapping();
```
在上面的例子中，emails:send這個Artisan指令如果不是真正執行中，會每分鐘都嘗試執行一次。如果<br/>
各個排程任務執行時間差異甚大，無法準確預估執行時間，withoutOverlapping方法就會非常有用。<br/>

你也可以指定withoutOverlapping的鎖的有效時間，即定義鎖過期前經過的時間，預設是24小時。
```
$schedule->command('emails:send')->withoutOverlapping(10);
```

### 單一伺服器上執行任務
> ~~~
> 注意!! 要使用這個特性的前提是你必須使用memcached或redis的快取驅動。此外，必須有一台中央快取
> 伺服器供所有機器與其進行溝通。
> ~~~
如果你的應用程式是運行在多伺服器環境，你可能會需要限制所有排程任務只在同一台機器上被執行。<br/>
比如說你有一個每周五晚上產生報表的排程任務，如果該任務被運行在超過一台以上的機器，那麼報表<br/>
就會被重複產生數次，這並不是好現象。<br/>

為了指定任務只能在同一台機器上頭運行，在定義排程任務時，你必須使用onOneServer這個方法。<br/>
第一個執行該任務的伺服器將會獲得鎖，避免相同的任務又在其他機器上重複執行一次。
```
$schedule->command('report:generate')
                ->fridays()
                ->at('17:00')
                ->onOneServer();
```

### 維護模式
在維護模式下，排程任務是不會被執行的，因為我們不希望任務會干擾伺服器上尚未完成的維護工作。<br/>
然而，若想強制排程任務在維護狀態下依舊被執行，可以在定義任務時使用evenInMaintenanceMode這個<br/>
方法。
```
$schedule->command('emails:send')->evenInMaintenanceMode();
```

## 任務輸出
## 任務鉤子