# Laravel Eloquent Collection


## 簡介
Eloquent 回傳的所有多重結果集都屬於 Illuminate\Database\Eloquent\Collection 物件的實例，包括透過 get 方法<br/>取得或透過關聯存取到的結果。Eloquent 集合物件繼承自 Laravel 基底集合類別，所以也自然的繼承了幾十種<br/>
可用於與 Eloquent 模型底層陣列協作的方法。而所有的集合都是一個迭代器，讓你可以像操作PHP陣列一樣的<br/>
遍歷。
```
$users = App\User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```
然而，集合比陣列強大許多，它通過了更直觀的介面來公開可以被鏈結調用的 map、reduce 等操作。
```
$users = App\User::all();

$names = $users->reject(function ($user) {
    return $user->active === false;
})
->map(function ($user) {
    return $user->name;
});
```
> ~~~
> 雖然多數的 Eloquent 集合方法都是回傳新的 Eloquent集合實例，但是 pluck、keys、zip、collapse、flatten 和
> flip方法則回傳基本的集合實例。同樣的，若 map 操作回傳一個不具任何 Eloquent 模型的集合，它將自動轉換成
> 基本的集合物件。
> ~~~

## 可用方法
所有 Eloquent 集合都繼承了基本的 Laravel 集合物件。因此它們亦繼承了基本集合類別所提供的所有方法。<br/>
 => [可用方法說明頁](https://github.com/Internaltide/Laradep/blob/master/laratopics/Collections.md#%E5%8F%AF%E7%94%A8%E6%96%B9%E6%B3%95)

## 客製集合
如果你希望返回自訂的集合物件並擁有自己擴充的集合方法，可以在你的模型內實作 newCollection 方法予以覆寫。<br/>
如此就會回傳你自定的集合實例而非預設的 Eloquent 集合實例。
```
<?php

namespace App;

use App\CustomCollection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Create a new Eloquent Collection instance.
     *
     * @param  array  $models
     * @return \Illuminate\Database\Eloquent\Collection
     */
    public function newCollection(array $models = [])
    {
        return new CustomCollection($models);
    }
}
```