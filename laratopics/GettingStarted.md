# Laravel Getting Started

## Installation

#### 安裝Composer
Laravel利用Composr來管理所有的依賴套件，而Laradock的workspace環境<br/>
其實已準備好composer執行環境了。所以基本上，並不需要再重新安裝。<br/>
但若仍需於Linux下獨立安裝Composer，則可參考下面的安裝步驟
```
下載安裝檔
curl -sS https://getcomposer.org/installer > installer

建立composer.phar指令檔
php installer

移往/usr/local/bin方便全域使用
mv composer.phar /usr/local/bin/composer
```

#### Web Server 設定
主要是設定讓URL可以忽略index.php。<br/>
當然若使用的是Laradock，這些都會自動被設定好。
##### Apache
需先啟用apache的mod_rewrite模組。<br/>
預設，專案public目錄下的.htaccess便已加入了相關設定<br/>
若無法生效，可嘗試使用下面設定進行取代。
```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
##### Nginx
```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

## Confiruration
#### DotEnv
Laravel提供用戶依據不同環境配置不同的軟體運行設定。<br/>
底層是透過DotEnv套件來實現，各設定項皆能實現成可透過 **.env** 來覆寫的方式。<br/>
其是藉由使用env()函數來實現。<br/><br/>
**.env** 環境設定檔依據專案建立的方式會有不同的產生方式，簡述如下：<br/>
 * 透過laravel指令建立<br/>
    專案根目錄下只會有 **.env.example**，需透過手動變更成 **.env**。<br/>
    當然，依據團隊所需，可保留 **.env.example**並透過該檔保留必要設定，<br/>
    再由開發者複製更名為 **.env**並加入適用自己開發環境的設定
 * 透過composer指令建立<br/>
    透過composer建立的專案，會自動將 **.env.example**rename成 **.env**
#### 測試環境設定檔 **.env.testing**
該檔案作為PHPUnit執行期與執行夾帶--env-testing之Artisan指令時的第一順位環境設定檔，其設定值<br/>
會覆寫 **.env**。
#### 特殊設定格式
 * true、(true) => 皆被env()解析成布林值true
 * false、(false) => 皆被env()解析成布林值false
 * null、(null) => 皆被env()解析成null
 * empty、(empty) => 皆被env()解析成布空字串''
 * 當設定值內包含空格時，則設定值必須放在雙引號之間<br/>
    Ex. APP_NAME="My Application"

 #### 在設定檔內取得環境設定
  * 透過PHP的超全域變數$_ENV來取得<br/>
     當應用程式收到請求時， **.env**檔案內所有變數都會被載到 $_ENV 這個 PHP 超級全域變數
  * 透過Laravel的輔助函數env()來接收環境設定檔內的設定值<br/>
     Ex. env('設定項名稱','{預設值}')，當不存在該設定項的設定時，函數中的第二參數即作為預設使用值

#### 應用程式環境
Laravel官方文件有提及，應用程式的環境是取決於 **.env**內的變數APP_ENV設定。<br/>
程式內部則可藉由App facade的environment函數進行環境的判斷。判斷方式如下：
```
if (App::environment('local')) {
    // 如過應用程式環境為local，則執行此區塊程式碼
}

if (App::environment(['local', 'staging'])) {
    // 如過應用程式環境為local或staging，則執行此區塊程式碼
}
```

#### 應用程式多環境配置
APP_ENV可被server-level的APP_ENV設定覆蓋，因此可藉下列幾種設定方式，<br/>
達成單一應用多環境的配置。而載入的環境設定檔對象將由 **.env**轉變成 **.env.{APP_ENV}**<br/>
Ex. server-level APP_ENV = development，則改讀取環境設定檔 **.env.development**<br/>
PHP-FPM
```
修改 PHP-FPM 設定檔，加入APP_ENV設定，然後重啟PHP-FPM

env[APP_ENV] = {Your Application Environment Name}
```
Nginx
```
修改Nginx設定檔，加入APP_ENV設定，然後重啟Nginx

location = /index.php {
    index index.php;
    try_files $uri =404;
    fastcgi_pass unix__tmp_php_cgi_sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    fastcgi_param APP_ENV {Your Application Environment Name};  // 加入的APP_ENV設定
}
```
尚未釐清之處：<br/>
 * 不存在伺服層級之APP_ENV時<br/>
    更改APP_ENV雖可變更環境，但仍為同一份環境設定檔 **.env**，<br/>
    這要如何達到多配置的目的?
 * 存在伺服層級之APP_ENV設定<br/>
    各機器根據自己的APP_ENV設定，可使用專屬的環境配置檔。確實可達到多重環境配置效果。<br/>
    但透過php artisan env取得所屬環境時，卻仍舊顯示 **.env**內的設定而非 **.env.{environment}**的設定?

#### 應用程式內存取設定值
讀取設定值<br/>
config({設定檔名稱}.{設定項目名稱},{預設值});
```
config('app.name','Laravel Project');  // 如果不存在app.name這個設定項，則使用Laravel Project
```
執行期寫入設定值<br/>
config(['{設定檔名稱}.{設定項目名稱}' => '{設定值}']);
```
config(['app.timezone' => 'Asia/Taipei']);  // 將app.timezone設定成Asia/Taipei
```
設定值擷取優先度
```
設定檔: app.php
環境設定檔: **.env**
環境設定檔內的設定: APP_NAME=Develop
設定檔內的設定: name = env(APP_NAME,'Project');
程式內的取用: config('app.name','Product')

擷取優先順序: 執行期的寫值 > Develop > Project > Product
```

#### 設定快取
使用php artisan config:cache可對設定檔進行快取，提升框架載入設定的速度。<br/>
通常該指令會被納入到產品佈署流程中，但無須在開發環境中被使用。<br/>
要注意的是，使用快取指令後， **.env**檔案不會再被載入，使用env函數也只會<br/>
回傳null。所以，若某設定項目仍要以 **.env**為優先，就要確保該設定項使用了env函數來載入。

#### 維護模式
注意!! 維護模式下，佇列任務將會停止直到維護模式被關閉。<br/><br/>
啟用維護模式
```
php artisan down
```
啟用維護模式並傳遞消息及設定HTTP Header 的 Retry-After<br/>
Http Retry-After: 用以宣告用戶需要等待多久後才能繼續發送下一個請求
```
php artisan down --message="Upgrading Database" --retry=60
```
啟用維護模式，但限定特定IP或網段仍可存取
```
php artisan down --allow=127.0.0.1 --allow=192.168.1.0/24
```
關閉維護模式
```
php artisan up
```
客製化維護模式頁面
```
重新定義模板 resources/views/errors/503.blade.php
```

## 目錄結構
 * app<br/>
    核心程式的主要存放目錄，主要分為下列幾項：
    * 預設存在
      * Http<br/>
         用途：存放所有相關處理請求的類別，如Controller、middleware及form requests等
      * Providers<br/>
         用途：服務提供者類別的存放位置，可通過服務容器進行服務綁定、事件註冊等工作。
      * Console<br/>
         用途：控制台核心，也是存方自定義Artisan指令與排程任務的地方。<br/>
         控制程式產生指令： **make:command**
      * Exceptions<br/>
         用途：存放各例外處例類別或自定義例外類別。可透過修改目錄下的Handler.php<br/>
         來客製化自己的例外產生、紀錄的方式。
    * 預設不存在(各項子目錄皆在對應的Artisan指令執行後產生)
      * Broadcasting<br/>
        用途：存放各類事件廣播頻道類別。<br/>
        控制程式產生指令： **make:channel**
      * Events<br/>
         用途：存放各事件類別。<br/>
         控制程式產生指令： **event:generate** 或 **make:event**
      * Jobs<br/>
         用途：存放佇列任務類別，建立的任務可選擇性放到佇列或設定成與當下請求的生命週期同步。<br/>
         控制程式產生指令： **make:jobs**
      * Listeners<br/>
         用途：存放各類事件監聽器類別。<br/>
         控制程式產生指令： **event:generate** 或 **make:listener**
      * Mail<br/>
         用途：存放應用程式的寄送任務封裝類別，主要針對一些複雜的寄送任務邏輯。<br/>
         控制程式產生指令： **Mail::send**
      * Notifications<br/>
         用途：存放各類交易式通知任務，底層可使用各種Driver，諸如Email、Slack、SMS等。<br/>
         控制程式產生指令： **make:notification**
      * Policies<br/>
         用途：存放各類授權驗證類別，用以管理資源存取權限。<br/>
         控制程式產生指令： **make:policy**
      * Rules<br/>
         用途：存放各類請求資料驗證規則的封裝類別。<br/>
         控制程式產生指令： **make:rule**
 * bootstrap<br/>
    放置引導設定加載的文件，如app.php。<br/>
    子目錄cache則用來存放相關的cache文件。
 * config<br/>
    存放應用程序的所有設定文件
 * database<br/>
    主要存放資料庫遷移文件、數據填充文件、model factories跟Sqlite資料庫
 * public<br/>
    主要存放資源文件及入口文件index.php
 * resources<br/>
    主要存放未編譯的資源、語系字典檔跟Blade模板
 * routes
   * web.php<br/>
      置於web中間件內，這邊的路由含有session、CSRF保護跟cookie加密的處理過程。
   * api.php<br/>
     置於api中間件內，這邊的路由含有頻率限制、token身分認證的處裡過程。<br/>
     另外，其屬於無狀態且無法存取session。
   * console.php<br/>
      定義所有控制台命令的地方
   * channels.php<br/>
      所有事件廣播頻道的註冊地
 * storage<br/>
    * app<br/>
       存放應用程式產生的文件
    * framework<br/>
       存放框架生成的文件跟快取。如編譯過的Blade模板、File形式的session檔案跟檔案快取
    * logs<br/>
       存放應用程式日誌檔
 * tests<br/>
    存放自動化測試程式，內建使用PHPUnit，每個測試類都須以Test為後綴
    ```
    使用 phpunit 或 php vendor/bin/phpunit 來啟動測試運行
    ```
 * vendor<br/>
    所有的composer依賴包存放位置

## Deployment
主要提供幾個能確保佈署正確的任務
 * 優化自動加載 - 使composer可以快速找的正確的文件並加載
 ```
 composer install --optimize-autoloader --no-dev

 注意!! 請將composer.lock加入版控，有了該檔案可以讓依賴更快速被安裝
 ```
 * 優化設定載入 - 在佈署流程中快取設定
 ```
 php artisan config:cache
 ```
 * 優化路由載入 - 快取路由 (對具有很多路由的大型應用可以有效提升路由效能)
 ```
 php artisan route:cache

 注意!! 該指令無法快取含有閉包的路由
 ```