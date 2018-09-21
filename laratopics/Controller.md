# Laravel 控制器

> Laravel 可以直接在路由的閉包中定義請求的處理邏輯，但我們可以透過控制器<br/>
> 來組織這些行為，分門別類且妥善地將一群相關的請求群聚在一個控制類中。<br/>
> 控制器類主要放在app/Http/Controllers這個目錄中。<br/>


## 基礎控制器
### 定義控制器
下面為一個控制器該有的樣子，它必須繼承自Laravel的Controller基底類別。該基底類別<br/>
可以提供一些方便的方法，像是middleware方法就可以用來為控制器行為添加中介層。
```
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```
我們可以定義像是下列這樣的路由並將其指向上述的控制器，其中的路由參數<br/>
也會被傳遞到控制器的show方法中
```
Route::get('user/{id}', 'UserController@show');
```
### 路由裡的控制器命名空間
有一點非常重要!! 我們在定義路由時並不需要指定完整的控制器命名空間。因為，RouteServiceProvider<br/>
預設就是在包含命名空間的路由群組中加載路由文件的。所以，我們僅需要定義App\Http\Controllers後面<br/>
那一部分。舉例來說，控制器的完整命名空間為App\Http\Controllers\Photos\AdminController，那麼我們只需<br/>
定義路由如下：
```
Route::get('foo', 'Photos\AdminController@method');
```
### 單一行為控制器
如其名，定義只有單一行為的控制器時，我們只需定義一個_invoke方法
```
<?php

namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class ShowProfile extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```
在設定單一行為控制器的路由時，無須指定方法，只需指出控制器即可
```
Route::get('user/{id}', 'ShowProfile');
```
我們也可以在使用指令建立控制器的同時，透過附加 --invokable選項來宣告要創建的是<br/>
一個單一行為控制器
```
php artisan make:controller ShowProfile --invokable
```

## 控制器裡的中介層
### 配置路由時指定中介層
```
Route::get('profile', 'UserController@show')->middleware('auth');
```
### 建構式裡配置中介層
這是更為普遍且方便的方式，尤其在只想為部分控制器方法指定中介層時
```
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```
> 雖然可以只為特定Action套用中介層，但這也可能意味著控制器的職責範疇過大，建議<br/>
> 可以考慮將其切割成更多的小控制器類別。
### 使用閉包定義中介層行為
有時僅僅為了特定控制器提供中介層控制時，可直接用閉包定義中介層行為來取代建立一個中介層程式檔
```
$this->middleware(function ($request, $next) {
    // ...

    return $next($request);
});
```


## 資源控制器

### 建立控制器
使用指令make:controller時，若宣告定義的是資源控制器，則控制器內就會預建好處理CRUD<br/>
的典型Action，如create、edit、update等...
```
php artisan make:controller PhotoController --resource
```

### 資源路由
使用resource定義的路由是專為資源控制器而生，這個單一路由即會創建對應各種資源處理方法的路由。
#### 建立單一資源路由
```
Route::resource('photos', 'PhotoController');
```
#### 建立複數個資源路由
```
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

### 資源控制器操作列表(以前述photoController為例)
|  動作         | URI            | 行為          | 路由名稱     |
| ------------- |:-------------:|:-------------:| ---------------:|
| GET           | /phtots       | index          | photos.index |
| GET           | /phtots/create | create     | photos.create |
| POST         | /phtots       | store           | photos.store  |
| GET           | /phtots/{photo} | show   | photos.show |
| GET           | /phtots/{photo}/edit | edit | photos.edit |
| PUT/PATCH | /phtots/{photo} | update | photos.update |
| DELETE    | /phtots/{photo} | destory | photos.destory |

### 指定資源模型
若想採用路由模型綁定，可以在建立控制器時使用選項--model指定模型，建立控制器<br/>
會自動在資源方法中使用類型提示以注入模型實例
```
php artisan make:controller PhotoController --resource --model=Photo
```

### 偽造表單方法
透過額外增加名為 **_method** 的隱藏欄位，以支援目前尚不支援的表單提交方法，諸如<br/>
PUT、PATCH和DELETE。在視圖中可以使用Blade指令@method簡易的創立。
```
<form action="/foo/bar" method="POST">
    @method('PUT')
</form>
```

### 部分資源路由
宣告資源路由時，預設會建立對應資源控制器的所有CRUD相關Action。但也可以只指定<br/>
僅創建部分Action的對應路由。
```
Route::resource('photos', 'PhotoController')->only([
    'index', 'show'
]);

Route::resource('photos', 'PhotoController')->except([
    'create', 'store', 'update', 'destroy'
]);
```

### API式的資源路由與控制器
你可以改用apiResources方法來建立資源路由。該指令會排除掉顯示表單型路由，如create與edit，<br/>
因為API並不需要。
```
// Single
Route::apiResource('photos', 'PhotoController');

// Multi
Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```
同樣地概念，透過附加選項--api來建立API式的控制器
```
php artisan make:controller API/PhotoController --api
```

### 為資源路由命名
預設，每個資源控制器Action都會有一個對應的路由名稱，但可以傳入names陣列參數來予以覆寫。
```
Route::resource('photos', 'PhotoController')->names([
    'create' => 'photos.build'
]);
```

### 為資源路由參數命名
預設，資源路由內的路由參數都是以單數型態來命名的，但可以傳入parameters陣列參數來予以覆寫。
```
Route::resource('users', 'AdminUserController')->parameters([
    'users' => 'admin_user'
]);

// 上述設定會讓路由變成如下所示
/users/{admin_user}
```

### 為資源URIs進行本地化
預設，資源路由是使用英語系的動詞做為方法名稱，但可以在AppServiceProvider的boot方法內<br/>
透過Route::resourceVerbs進行覆寫。
```
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```
一但進行了上述的本土化設定，像 **Route::resource('fotos', 'PhotoController')** 所產生的路由URI<br/>
會變成如下
```
/fotos/crear

/fotos/{foto}/editar
```

### 一些關於路由控制器的補充
有時當我們需要替預設的資源控制器增加額外的進入路由時，那些路由設定必須在Route::resource<br/>
之前就定義好，否則可能產生問題。例如路由優先度。
```
Route::get('photos/popular', 'PhotoController@method');

Route::resource('photos', 'PhotoController');
```
> 記住!! 控制器也需要保持職責的專一，如果你發現出現了典型資源操作以外的方法，<br/>
> 你應該考慮將其分開在兩個不同的控制器，而非放在同一個控制器內。

## 依賴注入與控制器

## 路由快取