# Laravel Artisan 指令列


## 簡介
### 查詢可用指令
Artisan 是 Laravel 內建的命令指令行，它能提供許多好用的指令來協助開發程式。可以使用list來<br/>
呈列出可用的指令。
```
php artisan list
```

### 指令輔助說明
每一個Artisan指令都具有一個輔助說明，可以告知該指令的可用參數及選項。要開啟輔助說明視窗，<br/>
只要在Artisan指令名稱之前加上help即可，例如：
```
php artisan help migrate
```

### Laravel REPL
所有Laravel應用程式都包含Tinker，它是基於 PsySH 這個套件所提供的 REPL，可提供互動式的編程。<br/>
它讓你直接在命令列指令行視窗上與應用程式進行互動， 透過執行tinker這個指令，即可進入Tinker環境。
```
php artisan tinker
```

## 撰寫指令
除了Artisan內建指令外，你可以創建自己的指令。一般來說，指令程式會被放在app/Console/Commands<br/>
這個目錄下。當然，你可以變更放置的目錄位置，只要指令可以被 Composer 載入就好。<br/>

### 創建指令
要創建一個新指令，你可使用make:command這個內建的Artisan指令。該指令會在app/Console/Commands<br/>
目錄下創建一個初始的指令程式檔，而產生的指令則會包括所有指令中預設的屬性與方法。
```
php artisan make:command SendEmails
```

### 指令結構
創建新指令後，必須優先完成指令類中的 **signature** 與 **description** 兩個屬性，這兩個屬性<br/>
將會被用在指令輔助說明上。接著是定義指令邏輯操作的 **handle** 方法，其會在指令執行時被調用。
> ~~~
> 為了程式碼能更有效的複用，建議終端指令的程式碼保持輕量化，並讓它們緩載到應用程式服務的
> 任務完成。在下列範例中，請注意！我們注入了一個服務類別來完成發送信件的「重任」。
> ~~~
```
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
    protected $drip;

    /**
     * Create a new command instance.
     *
     * @param  DripEmailer  $drip
     * @return void
     */
    public function __construct(DripEmailer $drip)
    {
        parent::__construct();

        $this->drip = $drip;
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->drip->send(User::find($this->argument('user')));
    }
}
```
讓我們來看一下上面這個範例，注意!! 我們可以在指令類別的建構式或handle方法注入我們所需的<br/>
依賴，Laravel容器服務會自動根據型別的提示來注入對應的依賴。

### 閉包指令
使用閉包來定義指令是另一種定義指令的方式，就像是路由閉包是定義控制器的替代方案一樣。<br/>
實際上，app/Console/Kernel.php的command方法中，你會發現框架載入了檔案routes/console.php。<br/>
即使這個檔案並未定義Http的路由，它卻代表著你的應用程式指令的進入點。
```
/**
 * Register the Closure based commands for the application.
 *
 * @return void
 */
protected function commands()
{
    require base_path('routes/console.php');
}
```

在route/console.php這個檔案中，你可以使用Artisan::command這個方法來定義所有基於閉包的路由。<br/>
command 方法可以接受兩個參數，一個是指令名稱，另一個則是可取得指令參數與選項的閉包。另外，<br/>
因為閉包綁定最底層的指令實例，所以你完全可以使用指令類別的所有輔助方法。
```
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

#### 型別提示注入依賴
閉包中除了上述提及可以接收指令名稱與指令參數、選項外，也可以透過型別提示來將依賴注入，服務<br/>
容器會自動解析。
```
use App\User;
use App\DripEmailer;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

#### 閉包指令描述
當你使用閉包在定義指令時，你可以使用**describe**方法來定義指令的描述，當你使用諸如<br/>
php artisan list 或 php artisan help {command}這樣的Artisan指令時，你所定義的描述文就會顯示出來。
```
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```

## 定義期望輸入
當在定義新指令時，一般都會需要從使用者端接受參數與選項。你可以透過使用signature類別屬性來<br/>
定義你期望的使用者輸入，該屬性允許你定義之項目包含指令的名稱、參數以及選項。<br/>

### 參數
參數與選項都是使用大括號來包裹示意，在下面的例子中就是定義了一個必要的參數 **user**
```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```
你也可以把參數定義為可選或者為其指定預設值
```
// Optional argument...
email:send {user?}

// Optional argument with default value...
email:send {user=foo}
```

### 選項
選項與參數很雷同，是另外一種定義用戶輸入的方式。無論是命令行指令中使用或定義時，皆需連續使用
兩個連字符 **"-"**。<br/>而選項有兩種形式，一種就像參數一樣可接受指定值；另一種為真假開關不可接受值，有存在時為開啟，反之關閉。
```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```
```
php artisan email:send 1 --queue
```

#### 賦值的選項
如果使用者必須為選項指定一個值，只需要在名稱定義後面加入 = 符號。
```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```
在上述定義之後，你應該像下面這樣使用指令
```
php artisan email:send 1 --queue=default
```

你也可以在定義某選項需要指定值的同時，設定一個預設值給它。當使用者未指定值給該選項時，<br/>
則會直接使用定義的預設值。
```
email:send {user} {--queue=default}
```

為選項定義一個簡寫，在定義名稱時可以透過分隔符號 **"|"**來隔開簡寫與完整的選項名稱。
```
email:send {user} {--Q|queue}
```

#### 輸入陣列
如果你想定義參數或選項可指定多值並使用一個陣列接收所有設定時，可以使用 **\*** 符號來定義輸入陣列。<br/>
首先，讓我們先看一個陣列參數的實際例子。
```
email:send {user*}
```
在上述定義下，下面一條指令應用等同於為user參數設置了一個['foo', 'bar']這樣的陣列值。其中，<br/>
值的順序則是按指令上頭的定義順序。
```
php artisan email:send foo bar
```

當定義的輸入陣列為選項型態時，其各別輸入選項值也都要加上前綴：
```
email:send {user} {--id=*}

php artisan email:send --id=1 --id=2
```

#### 輸入描述
在定義輸入時也可以一併定義參數或選項的描述，你可在個別的定義中使用冒號來將描述文與以分隔。如果你需要<br/>
額外的空間來定義你的指令，可以直接跨越多行撰寫。
```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

## 指令 I/O
### 取得輸入
當指令執行期間，明顯地一定會需要存取到使用者傳遞的參數值或選項值。在閉包定義的指令中，可以透過閉包函數的<br/>
參數取得；類別定義的指令中，則需要在handle方法內部使用 **argument** 及 **option** 來取得單一項的參數值或選項值。
```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    // Retrieve a specific parameter...
    $userId = $this->argument('user');

    // Retrieve a specific option...
    $queueName = $this->option('queue');
}
```
若想要一次取得所有參數值，則請改用方法 **arguments** 及 **options** 來取得，其會返回陣列格式的參數資料。
```
// Retrieve all parameters...
$arguments = $this->arguments();

// Retrieve all options...
$options = $this->options();
```
PS. 如果指定的參數或選項並不存在，則方法回值皆回傳null

### 互動式輸入
除了顯示輸出外，你也可以在指令執行期間，要求使用者輸入資料。使用**ask**方法將可以觸發問題詢問以提示<br/>
使用者進行輸入回覆，接著把輸入轉傳給指令使用。
```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('What is your name?');
}
```

另外，還有一個與ask方法很相似的secret，只是使用secret這個方法後，使用者在視窗上的輸入會變成不可見的。<br/>
這對於一些較敏感性的資料起到了保護的作用，如密碼輸入。
```
$password = $this->secret('What is the password?');
```

#### 要求確認
如果你想要求使用者做個簡單的確認，可以使用confirm方法。預設，該方法會回傳false，但在互動過程中，若使用者
輸入了y或yes，則會改回傳true。
```
if ($this->confirm('Do you wish to continue?')) {
    //
}
```

#### 自動完成
anticipate方法可以被用來提供自動完成功能的可能選項。當然使用者仍可以忽略掉定義的可能選項而使用任意的指定值。
```
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

#### 多選題
如果想要提供使用者一系列預先定義好的選擇，可以使用choice這個方法。你也可以透過使用陣列鍵值來指定當使用者<br/>
沒有選擇時的預設使用值。
```
// 預設值為鍵值為1的 Dayle
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], 1);
```

### 自訂輸出
要將資料輸出到終端視窗中，可以使用 line、info、comment、question 和 error 這些方法。這些方法對會對應使用<br/>
能夠表達其目的的ANSI顏色。例如，讓我們使用info方法來顯示一般資訊給使用者，這些資訊將以綠色文字型態來呈現。
```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->info('Display this on the screen');
}
```

使用error方法顯示錯誤訊息，其將被以紅色字體來呈現。
```
$this->error('Something went wrong!');
```

如果你只想要單純輸出未上色的明文到終端視窗，可以使用 line 方法
```
$this->line('Display this on the screen');
```

#### 表格化
使用 table 方法可以將資料以行列方式進行表格化，只要指定好標頭與行，框架會自訂根據現有資料來計算合適的寬與高。
```
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

#### 進度條
對於一些耗時較久的任務，顯示其執行進度會是有幫助的。使用輸出物件，我們就可以開始、前進和停止進度條。<br/>
首先，要定義好總共的執行階段數，然後在每個階段完成後就讓進度條前進。
```
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```
更多關於進度條的進階項目，可參考 [Symfony Progress Bar component documentation](https://symfony.com/doc/current/components/console/helpers/progressbar.html)

## 指令註冊
由於在你的終端 kernel 的 command 方法中使用了call 這個方法，因此所有在app/Console/Commands這個目錄中的<br/>
指令類別都會自動被註冊進入Artisan中。不過事實上，你仍然可以額外再使用load這個方法來掃描其它放置指令<br/>
類別的目錄。
```
/**
 * Register the commands for the application.
 *
 * @return void
 */
protected function commands()
{
    $this->load(__DIR__.'/Commands');
    $this->load(__DIR__.'/MoreCommands');

    // ...
}
```
你也可以直接手動將自訂的指令類別加入到app/Console/Kernel.php的 $command 屬性中。當 Artisan 啟動後，<br/>
該屬性中列出的所有指令將由服務容器解析並註冊到 Artisan 指令上。
```
protected $commands = [
    Commands\SendEmails::class
];
```

## 使用程式碼調用指令
有時候你可能會需要從終端機介面外的環境去執行 Artisan 指令。例如，你會想在路由或控制器方法內部去調用<br/>
Artisan指令，你可以使用 Artisan Facade 的call方法來完成這樣的工作。該方法可以接受指令名稱或指令類別名稱做為
其第一個參數；然後以陣列格式裝載參數資料做為其第二參數值。指令執行後的退出碼將會被回傳：
```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

使用Artisan Facade的queue方法則可以將Artisan指令加入佇列工作中。在使用此方法前，請先確認好佇列的配置及<br/>
是否存在運行中的佇列監聽器。
```
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

你也可以在 Artisan 指令後面選擇所需的分發佇列或連線。
```
Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

#### 傳遞陣列
如果你的指令定義了選項可以接收陣列資料，則可以簡單地透過陣列格式將資料傳給選項。
```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--id' => [5, 13]
    ]);
});
```

#### 傳遞布林值
如果你需要定義一個不能接受字串值的選項，像是 migrate:refresh 指令的 --force 旗標，你就可以傳遞 true 或 false：
```
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

### 在指令中呼叫其它指令
有時候你還會想在現存的Artisan指令中再呼叫其他指令，你仍可以使用call來達到目的。
```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```

當你在指令中又調用其他指令時，如果你要抑制這些內部調用指令的輸出時，可以使用callSilent這個方法。<br/>
callSilent和 call 方法使用方式一樣。
```
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```
