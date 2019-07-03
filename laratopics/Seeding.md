# Laravel Seeding

> ~~~
> Laravel 提供了簡單的方式即利用 seed 類別來將測試資料填充到資料庫中。所有創建 seed 的類別預設都會<br/>
> 放在 database/seeds 目錄下。你可以任意為 Seed 類別命名，但應該遵守某些命名慣例，像是 UserTableSeeder<br/>
> 這樣的類別名稱。預設，Laravel以為你內建了一個 DatabaseSeeder 的 seed 類別，在該類別中，你可以使用 call <br/>
> 這個方法來執行其他的 seed 類別，藉此控制資料填充的執行順序。
> ~~~

## 撰寫資料填充器
首先，你可以使用 Artisan 指令 make:seeder 來創建 seed 類別，創建的類別則會放在 database/seeds 目錄下。
```
php artisan make:seeder UsersTableSeeder
```

預設，指令創建的 seed 類別會包含一個方法 **run**，該類別方法會在Artisan 指令 db:seed 執行後被調用。在<br/>
run 這個方法中，你可以新增任何想要的資料到資料庫中，你可以使用查詢建構器Eloquent模型工廠來實作資料的新增。<br/>
另外，Mass Assignment 的保護措施會在資料庫資料填充過程中自動停用。<br/>

舉個例子，讓我們修改一下 DatabaseSeeder 這個內建的 seed 類別，並在 run 方法內建立一些資料新增的資料庫語法。
```
<?php

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => str_random(10),
            'email' => str_random(10).'@gmail.com',
            'password' => bcrypt('secret'),
        ]);
    }
}
```
PS. 你可以直接在 run 方法藉由型別提示來自動注入你要的依賴，服務容器自動進行解析。

### 使用模型工廠
當然，為每一個筆的資料填充手動指定屬性是非常繁瑣的，你可以使用模型工廠予以取代，其可以更方便地產生大量<br/>
的資料庫紀錄。首先，可以先查看[模型工廠文件](https://laravel.com/docs/5.6/database-testing#writing-factories)來學習如何定義一個工廠。一但定義好了工廠，你就可以使用輔助<br/>
方法factories來將資料新增到你的資料庫中。<br/>

下面為一個建立50筆使用者資料並為其建立關聯的例子：
```
/**
 * Run the database seeds.
 *
 * @return void
 */
public function run()
{
    factory(App\User::class, 50)->create()->each(function ($u) {
        $u->posts()->save(factory(App\Post::class)->make());
    });
}
```

### 呼叫其他資料填充器
在 DatabaseSeeder 這個類別中，你可以使用 **call** 這個方法來執行其它的 seed 類別。使用 call 方法就能把資<br/>
料填充邏輯拆解到多份 seed 類別中，這免於了單一 seed 類別過於肥大的情況。而使用該方法時，只需傳送 seeder <br/>類別希望執行的名稱即可。
```
/**
 * Run the database seeds.
 *
 * @return void
 */
public function run()
{
    $this->call([
        UsersTableSeeder::class,
        PostsTableSeeder::class,
        CommentsTableSeeder::class,
    ]);
}
```

## 運行資料填充
一但你撰寫好了資料填充類別，你可能會需要使用 composer 的 dump-autoload 指令來重新生成自動加載。
```
composer dump-autoload
```

接著，就可以使用 Artisan 指令 db:seed 來執行資料填充。預設，該指令會先運行DatabaseSeed這個類別，而該類<br/>
內部還可以調用其它的 seed 類別。當然，你可以使用 **--class** 這個選項來指定獨立運行其它的 seed 類別。
```
php artisan db:seed

php artisan db:seed --class=UsersTableSeeder
```

你也可以使用 Artisan 指令 migrate:refresh 來將填充資料，改指令會先回復所有執行過的資料填充，接著再重新跑<br/>
一次所有的資料填充。這個指令對於要重建數據庫時相當有用。
```
php artisan migrate:refresh --seed
```