# Database Migration

>
> 資料庫Migration就像是對資料庫的版本控制，讓團隊輕鬆修改並共享資料庫結構。
>

## 建立遷移
### 建立遷移檔指令
```
php artisan make:migration create_users_table
```
PS. 新建立的遷移檔案預設都會放在database/migrations目錄中。每個遷移文檔名稱也都會含有<br/>
一個時間戳記，以供判斷遷移的順序。<br/>

### 指令延伸參數
**--table**
```
// 會預先於up、down兩個方法內指定好資料表名稱為 users
php artisan make:migration add_votes_to_users_table --table=users
```
**--create**
```
// 會預先於up方法內寫好建立新表 users的程式碼；於down方法內寫好drop table的程式碼
php artisan make:migration create_users_table --create=users
```
**--path**
```
// 自訂遷移檔路徑，須為相對於應用程式的相對路徑
php artisan make:migration foo --path=app/migrations
```

## 遷移檔案結構
遷移類別主要包含兩種方法up、down。<br/>
up用來建立表格、欄位、索引等；down則為up方法的反向操作，<br/>
可恢復資料庫到調用up方法前的狀態。<br/>
以建立航班資料表為例
```
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
```

## 遷移管理
### 執行遷移
基本指令<br/>
command: **migrate**
```
php artisan migrate
```

因為資料庫遷移有可能導致資料遺失，所以預設，在Production環境下執行遷移時，系統會先徵詢管理者的同意，<br/>
只有在管理者同意後才會正式運行遷移。若要強制遷移則需要加上force option<br/>
command: **migrate**
```
php artisan migrate --force
```

### 遷移還原
使用指令rollback可還原遷移的操作，該指令會取消最後一次執行的**批次**遷移，<br/>
所謂批次意味著可能是包含多筆遷移檔的一次性執行，所以取消的並不一定僅僅是<br/>
單一遷移檔的遷移動作。<br/>
command: **migrate:rollback**
```
php artisan migrate:rollback
```
大量還原遷移<br/>
command: **migrate:rollback、migrate:reset**
```
// 使用step選項，指定還原最後五次的遷移
php artisan migrate:rollback --step=5

// 還原所有的遷移
php artisan migrate:reset
```
### 重建遷移
command: **refresh** (先還原後重建)
```
// 針對所有的遷移，先行還原後再重新遷移一次
php artisan migrate:refresh

// 重建遷移並進行資料填充
php artisan migrate:refresh --seed

// 指定最後5次遷移進行重建，一樣先還原再重新遷移
php artisan migrate:refresh --step=5
```
command: **fresh** (先刪表後重建)
```
// 刪除所有表格後重新執行遷移
php artisan migrate:fresh

// 重建資料庫後並進行資料填充
php artisan migrate:fresh --seed
```

## 遷移細部設定
### 資料表
建立資料表<br/>
mehtod: **create**
```
// 建立資料表users
Schema::create('users', function (Blueprint $table) {
    // 開始逐一建立資料表的欄位
    $table->increments('id');
});
```
判斷表格或欄位是否存在<br/>
mehtod: **hasTable、hasColumn**
```
// 判斷users表格是否存在
if (Schema::hasTable('users')) {
    //
}

// 判斷users表格是否存在email欄位
if (Schema::hasColumn('users', 'email')) {
    //
}
```
預設Schema Facade會使用default連線，若欲改用其他連線，<br/>
可使用connection指定連線。<br/>
mehtod: **connection**
```
Schema::connection('foo')->create('users', function (Blueprint $table) {
    $table->increments('id');
});
```
重新命名資料表<br/>
mehtod: **rename**
```
Schema::rename($from, $to);
```
重新命名之資料表若存在外鍵約束，則須注意以下事項：
> Before renaming a table, you should verify that any foreign key constraints <br/>
> on the table have an explicit name  in your migration files instead of <br/>
> letting Laravel assign a convention based name. Otherwise, the foreign key<br/>
> constraint name will refer to the old table name.<br/>

移除資料表<br/>
mehtod: **drop、dropIfExists**
```
Schema::drop('users');

Schema::dropIfExists('users');
```

#### 其他設定資料表的項目
| 設定項目                                             | 描述                                                    |
| :-------------------------------------------- | :-------------------------------------------- |
| $table->engine = 'InnoDB';                  | 指定Mysql使用的資料庫引擎          |
| $table->charset = 'utf8';                        | 指定Mysql使用的資料編碼字符集  |
| $table->collation = 'utf8_unicode_ci';  | 指定Mysql資料庫的定序規則          |
| $table->temporary();                            | 建立臨時表格(不支援SQL Server)   |


### 資料欄位
既有表格內部建立欄位資訊<br/>
```
Schema::table('users', function (Blueprint $table) {
    // 為users資料表建立字串型態的email欄位
    $table->string('email');
});
```
<a href="https://laravel.com/docs/5.6/migrations#creating-columns">更多可設定的欄位項目( Available Column Types )</a>

修飾表格內部既有欄位<br/>
```
Schema::table('users', function (Blueprint $table) {
    // 將users資料表的email欄位修飾為可為null
    $table->string('email')->nullable();
});
```
<a href="https://laravel.com/docs/5.6/migrations#column-modifiers">更多的欄位修飾項目( Available Column Modifiers )</a>

變更表格內部既有欄位<br/>
先決條件：必須先裝有composer套件**doctrine/dbal**，相關變更方法才會生效
```
composer require doctrine/dbal
```
變更欄位屬性<br/>
method: **change**
```
// 將欄位name改為長度50的string
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});

// 將欄位name改為長度50的string並改為nullable
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->nullable()->change();
});
```
注意!! 並非所有欄位皆可變更，以下為可變更欄位清單
> **bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json,**<br/>
>
> **longText, mediumText, smallInteger, string, text, time, unsignedBigInteger,** <br/>
>
> **unsignedInteger unsignedSmallInteger**<br/>

欄位更名<br/>
method: **renameColumn**
```
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```
PS. **enum**欄位上不支援變更名稱

移除欄位
method: **dropColumn**
```
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});

// 一次移除多個欄位
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```
PS. SQLite 資料庫並不支援在單行遷移中移除或修改多筆欄位。


#### 其他可用的別名方法
| 設定項目                                             | 描述                                                 |
| :-------------------------------------------- | :------------------------------------------ |
| $table->dropRememberToken();         | 移除remember_token欄位              |
| $table->dropSoftDeletes();                  | 移除delete_at欄位                          |
| $table->dropSoftDeletesTz();              | dropSoftDeletes()方法的別名        |
| $table->dropTimestamps();                 | 移除created_at跟updated_at欄位   |
| $table->dropTimestampsTz();             | dropTimestamps()方法的別名       |

### 資料索引
#### 索引建立
建立普通索引
```
$table->index('account_id');
```
建立複合索引
```
$table->index(['account_id', 'created_at']);
```
建立unique唯一索引
```
// 建立欄位後建立索引
$table->string('email')->unique();

// 對存在的email欄位建立索引
$table->unique('email');

// Laravel會自動生成合理的索引名稱，但亦可手動指定。
// 僅需將自訂索引名稱傳進方法的第二參數
$table->unique('email', 'unique_email');
```

##### 其他類型的索引建立方法
| 設定項目                                             | 描述                                               |
| :------------------------------------------- | :----------------------------------------- |
| $table->primary('id');                          | 建立主鍵id                                     |
| $table->primary(['id', 'parent_id']);     | 建立複合主鍵                               |
| $table->unique('email');                      | 建立唯一索引                                |
| $table->index('state');                          | 建立普通索引                                |
| $table->spatialIndex('location');          | 建立空間索引(Sqlite不支援)       |


#### 索引長度 & MySQL / MariaDB
Laravel預設使用utf8mb4編碼字符集，它支持在DB儲存**emojis**表情圖示。<br/>
但若Mysql或MarialDB的版本不夠時，則要為該資料欄位正確建立索引前，就必須<br/>
先手動設定在遷移運行時使用的預設字符串長度。<br/>
<br/>
在AppServiceProvider中調用Schema::defaultStringLength
```
use Illuminate\Support\Facades\Schema;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```
PS. 另一個可行作法是開啟資料庫的innodb_large_prefix選項。開啟方式則依各大資料庫文件指示。

#### 索引更名
method: **renameIndex**
```
$table->renameIndex('current', 'desired')
```

#### 移除索引
在移除索引前，必須明確給定索引名稱，而Laravel預設則會自動定義合理的索引名稱。<br/>
Laravel預設的索引名稱格式為為串接資料表名稱、欄位名稱與索引類型。

#### 各類索引移除案例
| 移除索引方式                                                             | 索引操作描述                        |
| :-------------------------------------------------------------- | :----------------------------------- |
| $table->dropPrimary('users_id_primary');                   | 移除 users table 的主鍵         |
| $table->dropUnique('users_email_unique');                | 移除 users table 的唯一索引 |
| $table->dropIndex('geo_state_index');                         | 移除 geo table 的普通索引   |
| $table->dropSpatialIndex('geo_location_spatialindex'); | 移除 geo table 的空間索引|

當傳遞的是欄位名為主的陣列資料時，則會自行根據所在的表名，<br/>
給定的欄位名及索引類型來決定要移除的索引名稱。
```
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // 將會移除索引 'geo_state_index'
});
```
#### 外鍵約束
建立外鍵欄位與約束<br/>
method: **foreign**
```
Schema::table('posts', function (Blueprint $table) {
    // 先建立外鍵欄位
    $table->unsignedInteger('user_id');

    // 再建立引用至users table的id欄位之外鍵約束
    $table->foreign('user_id')->references('id')->on('users');
});
```
串聯刪除或更新<br/>
method: **onDelete('cascade')、onUpdate('cascade')**
```
// 當users table某id的資料被刪除時，則外鍵資料表的相關關聯資料也一併刪除
$table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');
```
刪除外鍵約束( 外鍵約束名稱格式與索引相同 )<br/>
method: **dropForeign**
```
$table->dropForeign('posts_user_id_foreign');
```
開啟或關閉外鍵約束<br/>
method: **enableForeignKey、disableForeignKey**
```
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```