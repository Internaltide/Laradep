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

## 控制器中介層

## 資源控制器

## 依賴注入與控制器

## 路由快取