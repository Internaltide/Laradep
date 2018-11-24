# Laravel 分頁功能

> ~~~
> 在其它類型框架中，實作分頁功能可能是一個痛處。Laravel分頁器則整合了查詢建構器與Eloquent ORM，可以
> 提供一個方便簡單並基於資料庫資料的分頁結果。Laravel分頁器的分頁樣式則相容於Bootstrap CSS框架。
> ~~~

## 基本用法
### 依查詢建構器結果進行分頁
對資料進行分頁的方式有好幾種，最簡單的方式即在查詢建構器或Eloquent查詢上直接調用paginate方法，該方法會<br/>
依據目前使用者所瀏覽的頁數自動設定合適的分頁數量與偏移數。預設情況下，目前頁數會透過 HTTP 請求中的查<br/>
詢字串參數 page 的值來決定。當然，Laravel 會自動偵測這個值並且自動插入到分頁器所產生的連結。<br/>

在下面的範例中，傳入paginate 方法的唯一參數即想要在每頁顯示的資料數量，而我們想要顯示的是每頁15筆資料。
```
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Show all of the users for the application.
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->paginate(15);

        return view('user.index', ['users' => $users]);
    }
}
```
> ~~~
> 目前， Laravel 分頁器無法有效操作含有 groupBy 的 SQL 語句。如果需要對使用 groupBy 的查詢結果做分頁，建議
> 先行查詢資料庫後再手動製作分頁。
> ~~~

#### 簡易分頁
如果只是要在視圖上簡單顯示「下一步」和「上一步」這樣的連結，你可以使用 simplePaginate 方法來執行更高效<br/>
能的查詢。這對於為大型資料庫查詢結果進行分頁提供了效能較高的分頁方式。
```
$users = DB::table('users')->simplePaginate(15);
```

### 依Eloquent查詢結果進行分頁
你或許會想要對ORM查詢結果進行分頁，下面範例中會對user模型執行每頁15筆資料的方式來分頁。如你所見，<br/>
其語法結構與使用查詢建構器進行分頁的方式相當類似。
```
$users = App\User::paginate(15);
```

當然，也可以在對Eloquent模型設定了詢搜查尋條件後再呼叫 paginate 或 simplePaginate 方法，像是使用 where 子句。
```
$users = User::where('votes', '>', 100)->paginate(15);

$users = User::where('votes', '>', 100)->simplePaginate(15);
```

### 手動創建分頁器
有時候你會想要手動建立分頁實例，然後傳遞一個陣列格式的分頁資料給它。而 Illuminate\Pagination\Paginator 與<br/>
Illuminate\Pagination\LengthAwarePaginator 都是你可以用來創建的實例類型。對於 Paginator 類型的實例並不需<br/>
要知道資料總筆數，也因此它並未提供取得最後一頁頁碼的方法；相對地，LengthAwarePaginator 實例雖然所需參數與<br/>
Paginator 幾乎相同，但它則需要資料的總筆數。<br/>

換句話說，Paginator 對應於查詢建構器和 Eloquent 上的 simplePaginate 方法；LengthAwarePaginator 則對應於<br/>
paginate 方法。
> ~~~
> 當手動建立分頁器實例時，你就得手動對你的分頁資料陣列進行分割。如果你不確定如何做到這一點，
> 可以參考 PHP 的 array_slice 函式。
> ~~~

## 顯示分頁結果
當你調用 paginate 方法時，會返回一個 Illuminate\Pagination\LengthAwarePaginator 實例；調用 simplePaginate<br/>方法時，則返回 Illuminate\Pagination\Paginator 實例。這些分頁物件提供了數種方法用來描述結果集。除了這些<br/>
輔助方法外，分頁器實例也實現了疊代器的特性，可以像操作陣列一樣操作它。因此，一旦取得了結果，就可以<br/>
使用 Blade 的語法來顯示分頁結果並渲染出頁碼連結。
```
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```
範例中，links 方法即用來渲染出剩餘頁面的連結，而每一個連結都會包含適當的查詢字串參數 **page** 。記得，<br/>
使用links方法所產生的連結將相容於Bootstrap CSS 框架。

### 自訂分頁的URI
你可以透過分頁器實例調用withPath方法來自訂你的分頁連結。舉例來說，如果你想要產生像下方這樣一個連結的話，<br/>
**http://example.com/custom/url?page=N** ，你就應該傳遞 **custom/url** 這樣一個字串值到 withPath 方法。
```
Route::get('users', function () {
    $users = App\User::paginate(15);

    $users->withPath('custom/url');

    //
});
```

### 附加查詢字串參數到分頁連結
你還可以使用appends方法來附加額外的查詢字串參數到分頁連結上。舉例來說，若想要附加像 **sort=votes** 這樣的<br/>
查詢字串到每一個分頁連結上頭，你可以按以下方式來調用 appends 方法
```
{{ $users->appends(['sort' => 'votes'])->links() }}
```
如果你是想要附加一個Hash片段到分頁連結上，你必須改用 fragment 方法。例如想要附加 **#foo**，則請按下列方式<br/>
來調用 fragment 方法。
```
{{ $users->fragment('foo')->links() }}
```
[Hash Fragment 的特性](https://github.com/Internaltide/Laradep/blob/master/laratopics/Pagination.md#hash-fragment)

## 將分頁結果轉換成JSON
Laravel 分頁器結果類別實作了 Illuminate\Contracts\Support\Jsonable 介面，其中包含了 toJson 方法。該方法<br/>
可以讓我們更輕易地將方頁結果轉換為JSON。你也可以透過在路由或控制器直接回傳分頁器實例，其會自動轉換<br/>
成 JSON。
```
Route::get('users', function () {
    return App\User::paginate();
});
```
從分頁器實例轉換成JSON後，其會包含了像是 **total**, **current_page**, **last_page** 等更多的 meta 資訊，<br/>
而原實例內的資料則可透過 JSON 陣列中的 data 鍵中取得。下面即為路由回傳的分頁器實例轉換成 JSON 的一個例子。
```
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}
```

## 自訂分頁的視圖
預設，渲染出的分頁連結是相容於 Bootstrap CSS 框架的。然而，如果你的應用並非使用 Bootstrap，你依然可以自由<br/>
地定義自己的分頁連結渲染方式。當你在透過分頁器實例調用links方法時，請傳遞你自訂的分頁視圖名稱作為第一參數。
```
{{ $paginator->links('view.name') }}

// Passing data to the view...
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```
不過，自訂分頁視圖更容易的方式是使用 vendor:publish 指令直接將視圖導入到 resources/views/vendor 目錄中。
```
php artisan vendor:publish --tag=laravel-pagination
```
上面這個指令會將視圖放到 resources/views/vendor/pagination 這個目錄下，目錄中的 **bootstrap-4.blade.php**
檔案則是預設的Bootstrap分頁視圖，你可直接對該檔案進行客製修改。

如果你想改用其他預設視圖，可以在**AppServiceProvider**當中使用defaultView、defaultSimpleView來予以變更。
```
use Illuminate\Pagination\Paginator;

public function boot()
{
    Paginator::defaultView('pagination::view');

    Paginator::defaultSimpleView('pagination::view');
}
```


## 分頁器實例的可用方法
每個分頁器實例都是透過以下方法來提供額外的分頁資訊
 - $results->count()
 - $results->currentPage()
 - $results->firstItem()
 - $results->hasMorePages()
 - $results->lastItem()
 - $results->lastPage() (Not available when using simplePaginate)
 - $results->nextPageUrl()
 - $results->onFirstPage()
 - $results->perPage()
 - $results->previousPageUrl()
 - $results->total() (Not available when using simplePaginate)
 - $results->url($page)

>
> ## Hash Fragment
>
> 網頁資源是透過URI來進行標示與區別，而URI裡的 fragment 則是用來標示次級資源，可以看成是用來標示資源<br/>
> 裡的某個特定資源。其方式為透過在URI尾端附加一個以 **#** 為開頭的字符串，其中 # 則不屬於fragment的值。<br/>
> 另外，fragment 還有幾個特別之處：<br/>
> 1. 與識別符 **?** 有所不同， ? 後面的查詢字串會提交到伺服器端，但 fragment 則不會。
> 2. 更改 fragment 的值並不會刷新頁面，但仍會產生瀏覽歷史紀錄。
> 3. 瀏覽器會利用 fragment 的值並依據媒體類型(MIME TYPE)來進行相應地處理。
> 4. Google 爬蟲會忽略掉Hash Fragment的定義。
>
> ~~~
> ~~~
>