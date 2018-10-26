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
#### 型別提示注入依賴
#### 閉包指令描述

## 定義期望輸入

## 指令 I/O

## 指令註冊

## 使用程式碼調用指令