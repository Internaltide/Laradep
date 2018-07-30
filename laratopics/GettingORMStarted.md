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

## 複數Eloquent資料模型取得
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

## 取用單一Eloquent資料模型

[Return Document Home](https://github.com/Internaltide/Laradep/blob/master/README.md)<br/>
[Return Database Getting Started](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingDatabaseStart.md)
