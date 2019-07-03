# Laravel Eloquent Serialization

> ~~~
> 建構 JSON API 時，你經常會需要將模型或關聯轉換成陣列或 JSON。在 Eloquent 中已經內建了可用來轉換的便利<br/>
> 方法，並且可以控制序列化過程中應該包含哪些屬性。
> ~~~

## 序列化模型與集合
### 序列化為陣列
要將模型及載入的關聯轉換成陣列，你需要使用 toArray 方法。該方法會遞迴式地將所有屬性、關聯乃至於關聯的<br/>
關聯都轉換成陣列。
```
$user = App\User::with('roles')->first();

return $user->toArray();
```

你也可以將整個模型集合轉換成陣列
```
$users = App\User::all();

return $users->toArray();
```

### 序列化為JSON
要將模型轉換成陣列，你需要使用 toJson 方法。如同 toArray 一樣，toJson方法也會遞迴地將所有屬性、關聯轉換成<br/>
陣列。你還可以自行指定PHP所支援JSON編碼。
```
$user = App\User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

或者，你可以將模型或集合轉換成字串，其會自動在模型或集合上調用 toJson 方法來執行轉換。
```
$user = App\User::find(1);

return (string) $user;
```

由於模型與合集被轉換成字串時會轉換成 JSON，你可以直接從應用程式的路由或控制器中回傳 Eloquent 物件。
```
Route::get('users', function () {
    return App\User::all();
});
```

## JSON 隱藏屬性
有時你可能希望對模型陣列或JSON內的屬性進行限制，如隱藏密碼的顯示。此時，你可以新增 $hidden 屬性到你的<br/>
模型類別中來達到這個目的。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```
> 若是要隱藏關聯時，則使用關聯的方法名稱。

或者，你也可以使用 visible 屬性定義一個白名單，藉以定義可被包含在模型陣列或 JSON 的屬性。當模型被轉換<br/>
成陣列或 JSON 時，白名單以外的屬性都會被隱藏起來。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be visible in arrays.
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

### 暫時變更屬性可見性
如果你想將模型實例上原先隱藏的屬性暫時變更為可見，你可以使用 makeVisible 方法。而 makeVisible 方法還會<br/>
返回模型實例，讓你可以方便鏈結調用其他方法。
```
return $user->makeVisible('attribute')->toArray();
```

相同地，如果你想要將模型實例上原先顯示的屬性暫時變更為隱藏，你可以改用 makeHidden 方法。
```
return $user->makeHidden('attribute')->toArray();
```

## 附加資料到JSON
有時候在把模型轉換為陣列或 JSON 時，可能還會希望增加一些原本就不存在的欄位屬性。要達到此目的，首先<br/>
需要為這個值定義存取器。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the administrator flag for the user.
     *
     * @return bool
     */
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
}
```

另外，在建立存取器後，需將該屬性名稱新增到模型上的 appends 屬性。請注意，屬性名稱通常採用 snake case，<br/>
即使你是採用駝峰式命名來定義存取器。一但你將它新增到 appends 屬性後，該新增的屬性就會被包含到模型陣列與<br/>
JSON 中。而 appends 中的屬性一樣適用於使用 visible 和 hidden 的設定。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The accessors to append to the model's array form.
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```

### 即時附加
您可以使用 append 方法指示模型實例追加新的屬性。您還可以使用 setAppends 方法覆蓋掉模型實例既有的 appends <br/>
屬性設置。
```
return $user->append('is_admin')->toArray();

return $user->setAppends(['is_admin'])->toArray();
```

## 日期序列化
### 自訂每個屬性的日期格式
你可以為 Eloquent 模型內的日期屬性定義其序列化時使用的格式，方式是透過在 cast 屬性中定義其日期格式。
```
protected $casts = [
    'birthday' => 'date:Y-m-d',
    'joined_at' => 'datetime:Y-m-d H:00',
];
```

### 透過Carbon套件進行全域客製
Laravel 對 Carbon 日期函式庫進行了擴充，以其更方便客製 Carbon 的 JOSN 序列化格式。為了客製所有Carbon<br/>
日期在應用程式中被序列化的方式，可以使用 Carbon::serializeUsing 這個方法。而 serializeUsing 這個方法則<br/>
接受一個閉包函數，其會返回一個以字串行形式表示的JSON序列化日期。
```
<?php

namespace App\Providers;

use Illuminate\Support\Carbon;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Carbon::serializeUsing(function ($carbon) {
            return $carbon->format('U');
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```
