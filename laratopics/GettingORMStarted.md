# Eloquent ORM: Getting Started

> 一種數據庫與領域實體的關係映射模式，讓開發者直接透過使用領域模型來操作資料庫存取，<br/>
> ORM會藉由定義好的關聯自動映射產生SQL語句，省去與複雜SQL打交道的時間，只需簡單操作<br/>
> 物件的屬性與方法。<br/>
> 核心理念：**簡單**、**表達力**、**精確**<br/>
>
> 可能存在的缺點
> * 效能不彰 - 因多出的自動映射與管聯的管理，代價就是效能犧牲。只是在硬體、技術蓬勃發展的現在，<br/>
>                       此缺點的影響已降低很多。<br/>
> * 學習成本 - 使用了ORM不意味不用了解SQL，ORM需要額外的學習成本。其好處是體現在於資料存取<br/>
                          複用的時候，你無須再次糾結於一些關鍵細節上<br/>
> * 複雜SQL - 對於需要複雜SQL的任務上，ORM可能會有點力不從心，最後的ORM語法可能變得比原生SQL更難懂
>
> 用或不用？；或者何處該用何處要避開？<br/>
> 作者作為ORM的新信徒，無法給予太多意見，但網路上有一句話說得好<br/>
> "**向前邁進，享受它的優點，理解它的缺點，才能在現代和傳統開發知識的衝突上，找到屬於自己的平衡點。**"<br/>
> 所以 **Keep Going!!**
>
> ======


## 定義Eloquent模型
所有模型預設都放在app目錄下，亦可透過composer.json隨意放在可被自動加載的地方。<br/>
所有模型皆須繼承自**Illuminate\Database\Eloquent\Model**<br/>

### 創建模型指令
```
php artisan make:model Flight

// 同時建立數據庫遷移
php artisan make:model User --migration
OR
php artisan make:model User -m
```

### Eloquent模型的約定俗成
#### 表格名稱
預設約定資料表格名稱為類別名稱的複數型態，所以類別Flight的對應表格名稱就是flights。<br/>
只有在定義模型的$table屬性後，才會打破這個約束。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```
#### 主鍵
預設約定資料表格的主鍵名稱為**id**且為auto-increment的整數值。<br/>
定義模型的$primaryKey屬性，可自訂主鍵名稱<br/>
定義模型的$incrementing屬性為false，可取消主鍵的自增特性<br/>
定義模型的$keyType屬性，可變更主鍵的資料型態
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'my_flights';

    protected $primaryKey = 'flight_id';
    public $incrementing = false;
    protected $keyType = string;
}
```
#### 時間戳記
Laravel預設默認資料表存在created_at跟updated_at兩個時間欄位。實務上，若
 * 表格並不存在這兩個欄位 => 請將$timestamps屬性改為false以取消約束
 * 含既存且非約定名稱的兩種時間欄位名稱 => 請設定CREATED_AT及UPDATED_AT兩項類別常數
```
<?php
// 表格並不存在這兩個欄位時
namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    public $timestamps = false;
}
```
```
<?php
// 含既存且非約定名稱的兩種時間欄位名稱時
class Flight extends Model
{
    const CREATED_AT = 'e_date';
    const UPDATED_AT = 'u_date';
}
```
另外，如果要自定時間戳記的儲存格式，可設定$dateFormat屬性。<br/>
此屬性也涉及模型被序列化成陣列或 JSON時採用的時間儲存格式。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```
#### 資料庫連線
預設模型會使用設定檔內的Default連線，如果要使用不同的連線，需設定屬性$connection
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The connection name for the model.
     *
     * @var string
     */
    protected $connection = 'connection-name';
}
```

## 複數資料模型取得
Eloquent模型實例就是一個查詢建構器，所以可以使用所有查詢建構器的方法。<br/>
諸如all()、get()皆一次取回多筆資料，回傳時使用集合物件作為回傳值，內含多筆eloquent資料模型。
```
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
```
### 使用模型取得所有資料
method: **all**
```
<?php

use App\Flight;

$flights = App\Flight::all();

// 回傳的是內含數個eloquent資料模型的集合物件
foreach ($flights as $flight) {
    echo $flight->name;
}
```
### 資料集合存取
#### 如同陣列一般使用foreach來進行遍歷
```
foreach ($flights as $flight) {
    echo $flight->name;
}
```
#### 使用Laravel的集合方法作進一步的資料處理。
```
// 使用集合方法reject()過濾掉不要的資料
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
```
### 模型資料分塊
如同查詢建構器外，Eloquent模型亦使用相同的方法來進行分塊<br/>
method: **chunk**
```
Flight::chunk(200, function ($flights) {
    // 一次取得一小塊資料進行處理
    foreach ($flights as $flight) {
        //
    }
});
```

### 使用游標
一條SQL取得資料集，要遍歷它，一般是取得資料後再用foreach或集合方法。應用端需要的是一份資料集的記憶體量。<br/>
使用游標後變成一筆資料一次查詢的方式，直接從底層進行遍歷，一次回傳一筆資料。應用端只需一筆資料的記憶體量。<br/>
因此，官方是標榜對大資料量的處理，可以節省很多記憶體使用量。
```
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}
```
PS. 作者的疑慮，不曉得底層是不是使用資料庫游標(Database Cursor)來實作？<br/>
      若是的話，雖然節省了記憶體用量，但在時間效能上，是否反而可能產生瓶頸呢？

## 單一資料模型取得
### 使用find()、first()<br/>
相對於批量資料，此類方法會回傳單一的Eloquent模型物件實例。
```
// Retrieve a model by its primary key...
$flight = App\Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = App\Flight::where('active', 1)->first();

// 亦接受多主鍵值的陣列，只是回傳值將變回集合物件
$flights = App\Flight::find([1, 2, 3]);
```
### 自動返回例外
若希望在找不到資料時回傳例外，可改用findOrFail()、firstOrFail()<br/>
Laravel會在無結果時會傳Illuminate\Database\Eloquent\ModelNotFoundException這個例外
```
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```
PS. 若為捕抓到例外則返回404錯誤

## 聚合函數的使用
聚合函數只會返回數量值而非資料模型，如**count**、**sum**、**max**等
```
$count = App\Flight::where('active', 1)->count();

$max = App\Flight::where('active', 1)->max('price');
```

## 模型的插入與更新
### 插入
#### 單筆資料插入
要創建一筆資料，只需建立一個資料模型並設定好屬性，最後呼叫save方法即可。<br/>
其中，created_at跟updated_at會被自動設定。
```
<?php

namespace App\Http\Controllers;

use App\Flight;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * 建立一個新的航班實例。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 驗證請求...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}
```
#### Mass Assignment
Mass Assignment使用的方法為create()<br/>
網路上都把Mass Assignment翻譯成批量賦值，很容易被誤解是資料批次新增，但其實根本不是。<br/>
    關於兩個方法的比較：<br/>
    **save** - 必須自行建立模型實例、手動賦值，最後儲存<br/>
    **create** - 一個重新包裝的方法，只需將資料陣列傳入，剩下的動作都會自動在內部做掉，省事許多。<br/>

當然，create的方便性也潛藏著資安漏洞，它不像save方法是由開發者手動建立賦值的code，<br/>
自然會注意到什麼欄位不該寫。當資料陣列被惡意用戶安插進諸如is_admin的資料時，就可能<br/>
產生不當權限賦予的情形。<br/>
所以，Laravel特別設計**$fillable**、**$guarded**兩個屬性來定義可賦值欄位或不可賦值欄位，協助我們做過濾<br/>
預設是所有欄位皆不可經由Mass Assignment來寫入，所以，必須先設定好其中一個屬性，<br/>
方可執行Mass Assignment。

**使用$fillable屬性**
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 指定Mass Assignable的欄位。
     *
     * @var array
     */
    protected $fillable = ['name'];
}

// 設定好後，即可使用create新增資料。該方法會回傳已保存的模型實例
$flight = App\Flight::create(['name' => 'Flight 10']);

// 若已存在該模型的實例，則可改用方法fill()
$flight->fill(['name' => 'Flight 22']);
```
**使用$guarded屬性**
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 指定不可Mass Assignable的欄位。
     *
     * @var array
     */
    protected $guarded = ['price'];
}
```
**設定所有欄位皆Mass Assignable**
```
將屬性$guarded設為空陣列
protected $guarded = [];
```

### 更新
#### 單筆資料更新
如同單筆插入一樣，透過模型的save方法即可進行單筆資料更新<br/>
其中，updated_at會被自動設定。
```
$flight = App\Flight::find(1);

$flight->name = 'New Flight Name';

$flight->save();
```
#### Mass Update
Mass Update使用的方法為update()，同理應該也要受到$fillable跟$guarded的控管<br/>
只是官方文件的編排方式有點奇怪，只在Mass Assignment那段有提到，Mass Update又放在前面先講，<br/>
好像Mass Update就沒有風險一樣。
```
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```
PS. 通過eloquent進行Mass update時，saved跟updated這兩個模型事件將不被觸發。

## 其他創建方法進行Mass Assign
**firstOrCreate**<br/>
以第一個參數指定的鍵與值做為查詢基準，若不存在於資料庫則建立一筆新資料
```
// Retrieve flight by name, or create it if it doesn't exist...
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

// 同上，但使用第二參數額外指定要寫入的其他資料
$flight = App\Flight::firstOrCreate(
    ['name' => 'Flight 10'], ['delayed' => 1]
);
```
**firstOrNew**<br/>
以第一個參數指定的鍵與值做為查詢基準，若不存在於資料庫則以該資料建立模型實例並回傳，需再執行save才會存入資料庫
```
// Retrieve by name, or instantiate and return if it doesn't exist
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

// 同上，但使用第二參數額外指定要設定的其他資料
$flight = App\Flight::firstOrNew(
    ['name' => 'Flight 10'], ['delayed' => 1]
);
```
**updateOrCreate**
以第一個參數指定的鍵與值做為更新的對象基準，並使用第二參數進行更新，若不存在於資料庫則建立一筆新資料
```
// 如果有從奧克蘭飛往聖地牙哥的航班，將價格設定為99美金
// 如果不存在該航班資料就自己創建一個
$flight = App\Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99]
);
```

## 模型的刪除
### 刪除一筆資料
method: **delete**
```
$flight = App\Flight::find(1);

$flight->delete();
```
### 通過主鍵刪除資料
method: **destroy**
```
App\Flight::destroy(1);

App\Flight::destroy([1, 2, 3]);

App\Flight::destroy(1, 2, 3);
```
### 透過查詢刪除資料
```
$deletedRows = App\Flight::where('active', 0)->delete();
```
PS. 通過eloquent進行Mass delete時，deleting跟deleted這兩個模型事件將不被觸發。
### 軟刪除
顧名思義，資料不是真的從資料庫中被刪除，而是標記刪除。<br/>
另外，被標記刪除的資料會自動排除在查詢結果之外。<br/>
若要啟動軟刪除功能，須滿足下列條件：
 * 使用Illuminate\Database\Eloquent\SoftDeletes Trait
 * 需在模型的$dates屬性中添加deleted_at
 * 資料表也需含有deleted_at欄位
完成上述三項條件後，往後的刪除就會自動變成軟刪除模式

 **定義屬性**
 ```
 <?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;

    /**
     * 需要被轉換成日期的屬性
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}
 ```
 **使用Schema Builder建立deleted_at欄位**
 ```
 Schema::table('flights', function ($table) {
    $table->softDeletes();
});
 ```
 ### 判斷資料是否已遭軟刪除
 method: **trashed**
 ```
 if ($flight->trashed()) {
    //
}
 ```
### 查詢軟刪除的資料
#### 查詢結果包含軟刪除資料
預設是會排除軟刪除資料，但可透過使用**withTrashed**將軟刪除資料也包含在查詢結果內
```
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();

// withTrashed亦可用在關聯查詢
$flight->history()->withTrashed()->get();
```
#### 僅查詢軟刪除資料
使用方法為**onlyTrashed**
```
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```
#### 恢復被軟刪除的資料
使用方法為**restore**，與其他Mass Assignment方法一樣，不會觸發任何模型事件
```
$flight->restore();

App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();

// 亦可用在關聯查詢
$flight->history()->restore();
```
#### 永久刪除被軟刪除的資料
使用方法為**forceDelete**
```
// Force deleting a single model instance...
$flight->forceDelete();

// Force deleting all related models...
$flight->history()->forceDelete();
```

## 定義查詢作用域
### Global Scope
定義一個全域性的查詢約束，凡經套用即可讓模型的每個查詢都受到一定的約束。
Laravel的軟刪除就是利用全域的查詢作用域來排除已經軟刪除的資料。
### 編寫全域作用域並加以套用
#### 獨立定義
建立Class並繼承於Illuminate\Database\Eloquent\Scope介面，然後實作apply method以定義全域的查詢約束。
```
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>', 200);
    }
}
```
套用全域作用域<br/>
於模型的boot方法內使用addGlobalScope方法，將全域作用域物件傳入。
```
<?php

namespace App;

use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```
以User::all()為例，其底層SQL會變成如下
```
select * from `users` where `age` > 200
```

#### Via Closure(匿名全域作用域)
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class User extends Model
{
    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope('age', function (Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```
### 移除Global Scopes
method: **withoutGlobalScope**
```
// 使用作用域類別名做為參數
User::withoutGlobalScope(AgeScope::class)->get();

// 直接使用Closure時
User::withoutGlobalScope('age')->get();
```
method: **withoutGlobalScopes**<br/>
用以移除多個作用域約束
```
// Remove all of the global scopes...
User::withoutGlobalScopes()->get();

// Remove some of the global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

### Local Scope
用以定義常用且可以復用的局部作用域，方式是直接在模型中建立作用域方法，
並予以回傳建構器實例。其中，每個作用域方法都必須以scope這個字詞做前綴。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     *
     * @param \Illuminate\Database\Eloquent\Builder $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}
```
呼叫作用域方法時，並不需要scope前綴。以上面為例，呼叫active()就等同用了scopeAcive這個作用域方法
```
$users = App\User::popular()->active()->orderBy('created_at')->get();
```

### Local Dynamic Scope
前述幾個局部作用域方法，都只見傳入$query參數。但其實作用域方法還可定義第二個參數。
藉由定義此參數即可達到動態作用域的效果。
```
/**
* 限制查詢只包括指定類型的用戶。
*
* @return \Illuminate\Database\Eloquent\Builder
*/
public function scopeOfType($query, $type)
{
    return $query->where('type', $type);
}

// 調用方式
$users = App\User::ofType('admin')->get();
```

## 模型比對
提供比對兩個eloquent模型，看是否相同，可快速驗證兩個模型是否為相同主鍵、相同表格及相同連線<br/>
method: **is**
```
if ($post->is($anotherPost)) {
    //
}
```

## 事件
Eloquent模型定義了幾個事件的觸發點，讓開發者方便在模型生命週期內的幾個流程點上<br/>
額外定義其他任務，方便做擴充或監控。<br/>
Defined Events: **retrieved**, **creating**, **created**, **updating**, **updated**, **saving**, **saved**, **deleting**,  **deleted**, **restoring**, **restored**<br/>

事件觸發點定義說明：( 注意!! Mass Assignment相關方法都不會觸發事件 )
* **retrieved** - 從資料庫取得數據後觸發
* **creating、created** - 新資料第一次被建立時
* **updating、updated** - 資料模型已存在於資料庫並執行save()時
* **saving、saved** - creating、created、updating及updated觸發時，此兩者也都會被觸發
* **deleting、deleted** - 刪除資料時
* **restoring、restored** - 回復軟刪除資料時

### 前置步驟
定義模型屬性$dispatchesEvents，將需要的觸發時間點映射到定義好的Event Class上
```
<?php

namespace App;

use App\Events\UserSaved;
use App\Events\UserDeleted;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The event map for the model.
     *
     * @var array
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

## 觀察者
我們可以將監聽事件群聚到單一觀察者物件內，並使物件方法反映出模型事件名稱(使其一致)，<br/>
每個方法並接收模型實例做為其參數。最後，將觀察者物件註冊到服務提供者。

### 指令建立觀察者類別
```
php artisan make:observer WelcomeUserObserver --model=User
```
PS. 預設，觀察者類別會被建立在app/observers下，該目錄預設不存在，但在指令執行後會自動建立
```
<?php

namespace App\Observers;

use App\User;

class WelcomeUserObserver
{
    /**
     * Handle to the User "created" event.
     *
     * @param  \App\User  $user
     * @return void
     */
    public function created(User $user)
    {
        // 使用者註冊後寄信件
        Mail::send('emails.welcome', ['user' => $user], function($message) use ($user)
        {
            $message->to($user->email, $user->first_name . ' ' . $user->last_name)->subject('Welcome to My Awesome App, '.$user->first_name.'!');
        });
    }

    /**
     * Handle the User "updated" event.
     *
     * @param  \App\User  $user
     * @return void
     */
    public function updated(User $user)
    {
        //
    }

    /**
     * Handle the User "deleted" event.
     *
     * @param  \App\User  $user
     * @return void
     */
    public function deleted(User $user)
    {
        //
    }
}
```
### 註冊觀察者
請在服務提供者的boot method下，使用模型的observe方法進行觀察者的註冊<br/>
method: **observe**
```
<?php

namespace App\Providers;

use App\User;
use App\Observers\WelcomeUserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        User::observe(WelcomeUserObserver::class);
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

[Next Topic](https://github.com/Internaltide/Laradep/blob/master/laratopics/Relationships.md)<br/>
[Return Document Home](https://github.com/Internaltide/Laradep/blob/master/README.md)<br/>
[Return Database Getting Started](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingDatabaseStart.md#eloquent-orm)
