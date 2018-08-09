# Accessors & Mutators

> 當針對Eloquent模型進行屬性取用或設定時，存取器及修改器可以讓我們對<br/>
> 欄位資料進行格式化。例如想要加密儲存在資料庫的資料，就可以利用一個<br/>
> Mutators修改器進行加密格式的定義，在用一個Accessors存取器設定讓屬性被<br/>
> 存取時自動解密。

## 定義存取器
存取器函數名稱格式
```
get{屬性名稱}Attribute

Ex. 假設存取屬性為foo，模型上就要定義對應的存取器方法getFooAttribute()
```

以存取屬性first_name為例，當存取屬性first_name時，該屬性值就會被傳遞到存取器方法內，<br/>
經過方法內的處理後再回傳結果。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the user's first name.
     *
     * @param  string  $value
     * @return string
     */
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }
}


// 存取屬性
$user = App\User::find(1);
$firstName = $user->first_name;
```

也可以通過對已存在屬性進行組合或其他方式運算，然後建立可存取的新屬性
```
/**
 * Get the user's full name.
 *
 * @return string
 */
public function getFullNameAttribute()
{
    return "{$this->first_name} {$this->last_name}";
}
```

## 定義修改器
修改器函數名稱格式
```
set{屬性名稱}Attribute

Ex. 假設存取屬性為foo，模型上就要定義對應的存取器方法setFooAttribute()
```
以設定屬性first_name為例，當設定屬性first_name時，設定值就會被傳遞到修改器方法內，<br/>
在方法內經過指定的處裡後再將值設定到模型的$attributes。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Set the user's first name.
     *
     * @param  string  $value
     * @return void
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}


// first_name被設為Sally時，即會調用修改器方法setFirstNameAttribute
// 修改器會對設定值進行小寫轉換後存入模型$attributes
$user = App\User::find(1);
$user->first_name = 'Sally';
```

## 時間存取器
預設狀況下，created_at跟updated_at兩個屬性會被Eloquent轉換成Carbon實例，<br/>
所已我們就可以使用任何的Carbon方法直接對屬性值做額外處理。Ex.$user->deleted_at->getTimestamp();<br/>
當然我們還可以藉由覆寫模型的$dates屬性來決定有哪些欄位可以自動被轉換或，完全禁止。<br/>
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
     */
    protected $dates = [
        'created_at',
        'updated_at',
        'deleted_at'
    ];
}
```
當欄位被認為是日期時，我們可以把數值設定成為 UNIX 時間戳記、日期字串（Y-m-d）、<br/>
日期時間（ date-time ）字串、或者是DateTime、Carbon 實例，日期數值會自動正確的儲存到資料庫中。
```
$user = App\User::find(1);
$user->deleted_at = now();
$user->save();
```
預設來說，時間儲存格式將以'Y-m-d H:i:s'為主。如果你想要自訂時間儲存格式，就需在模型中設定$dateFormat屬性。
這個屬性定義了時間屬性該如何被儲存到資料庫，以及模型被序列化成一個陣列或 JSON 時的格式。
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

## 自訂屬性轉換
### 型別轉換
模型的$casts屬性提供設定屬性要以什麼樣的型別回傳。$casts為一個陣列，鍵名代表需要被轉換的<br/>
欄位名稱，而值就是代表你想要把欄位轉換成什麼樣的型別。支援的型別如下<br/>
**integer, real, float, double, string, boolean, object, array,  collection, date, datetime, and timestamp**

範例：
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}


// 當存取is_admin屬性時，is_admin皆會自動被轉換成布林值，即使它是以整數型態存在於資料庫
$user = App\User::find(1);
if ($user->is_admin) {
    //
}
```

### 陣列轉換
對於存有序列化JSON資料且型別為json或text型別的欄位，在模型$casts屬性中指定<br/>
該欄位轉換為array型態，則存取該欄為屬性時，它會自動被反序列化為陣列。<br/>
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該被轉換成原生型別的屬性。
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];
}
```
對於被指定array型別轉換的欄位，特別的是當你設定該屬性的值，給定的陣列將亦會自動的被序列化回JSON以進行儲存
```
// 取用的屬性會自動被轉換成陣列型態
$user = App\User::find(1);
$options = $user->options;
$options['key'] = 'value';

// 設定陣列值給屬性時，亦自動反序列回JSON型態
$user->options = $options;

$user->save();
```

### 日期轉換
當我們使用的是date或datetime的轉換型別時，還可以額外定義時間格式。<br/>
這對於模型是被序列化的陣列或JSON時特別有用。
```
/**
 * The attributes that should be cast to native types.
 *
 * @var array
 */
protected $casts = [
    'created_at' => 'datetime:Y-m-d',
];
```