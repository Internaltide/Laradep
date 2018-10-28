# Laravel 欄位驗證


> ~~~
> Laravel 提供了多種方法來驗證應用程式傳遞進來的資料。預設，Laravel基礎控制器類別會使用ValidatesRequests
> 這個Trait並搭配各種強大的驗證規則予以更加便利地驗證請求資料。
> ~~~

## 快速上手
要了解 Laravel 強大的驗證功能，可以先從建構一個表單驗證並顯示錯誤訊息給使用者的完整範例來做初步的了解。

### 1. 定義路由
首先，先假設我們在routes/web下有如同下方這樣的路由定義。
```
// 顯示讓使用者新增部落格文章的表單
Route::get('post/create', 'PostController@create');

// 將提交的部落格新文章儲存到資料庫
Route::post('post', 'PostController@store');
```

### 2. 定義控制器
相對於上方路由的簡易控制器範例，store方法的內容暫時先留白
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### 3. 撰寫驗證邏輯
現在我們將在store方法內添加資料驗證的邏輯，我們會直接使用請求物件提供的 validate 方法來實現驗證功能。<br/>
如果通過驗證規則，程式碼將正常執行就好像沒有驗證邏輯存在一樣；反之，若是驗證失敗，則拋出一個例外<br/>
並把適當的錯誤訊息回傳給使用者。在傳統的 HTTP 請求中，會產生一個重定向響應，對於 AJAX 請求則返回<br/>
JSON 回應。<br/>

為了更理解 validate 方法，我們先回到 store 方法中：
```
/**
 * Store a new blog post.
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...
}
```
如你所見，我們會把所需的驗證規則傳進 validate 方法中。再次提醒，如果驗證失敗，會自動產生適當的回應。如果<br/>
驗證通過，控制器會繼續正常執行。

#### 首次驗證失敗即停止驗證
因為一個欄位資料可以搭配多個驗證規則，有時你會希望某個欄位驗證失敗後就停止該欄位的其他驗證規則。這時，你<br/>
可以為該欄位指定一個**bail**的規則。以下方例子來說，當標題的唯一性規則檢查沒有通過時，後續的文章最大數量<br/>
限制規則就會被忽略而不進行檢查。另外，其驗證的順序會與規則指派的順序相同。
```
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

#### 關於巢狀資料的驗證提醒
如果 HTTP 請求包含了「巢狀」的資料，可以在驗證規則中使用「點」語法來指定資料。
```
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### 3. 顯示驗證錯誤的訊息
如同前面所提，當驗證失敗後將自動產生重定向到表單提交前的位置。另外，所有的錯誤訊息都會自動閃存到<br/>
session中。注意!! 我們並不需要在路由中明確的把錯誤訊息綁定到視圖中，這是因為 Laravel 會自動檢查 session<br/>
內是否存在錯誤訊息，如果錯誤訊息存在的話，會自動綁定這些錯誤訊息到視圖。你可以在視圖中使用$error物件<br/>
來取得錯誤訊息，而該物件是一個Illuminate\Support\MessageBag的實例。
> ~~~
> $errors變數是透過 web 中介層群組所提供的 Illuminate\View\Middleware\ShareErrorsFromSession 中介層來綁定到
> 視圖中。當應用這個中介層時，視圖中會永遠存在一個可用的 $errors 變數，你可以方便的假設 $errors 變數總是
> 有被定義且可以安全使用。
> ~~~

在範例中，驗證失敗時會將使用者重定向到控制器的 create 方法，讓我們可以在這個視圖中顯示錯誤訊息：
```
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

### 4. 一些注意事項、提醒
#### 關於可選欄位的說明
預設，Laravel 含有 TrimStrings、ConvertEmptyStringsToNull 這兩個全域的中介層。所以，如果你不想要讓驗證器<br/>
認為 null 值是無效的，你就需要把「可選」的請求欄位標註為 nullable。
```
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

#### AJAX 請求與驗證
在這個範例中，我們使用傳統的表單來發送資料給應用程式。然而，更多的應用會是透過AJAX請求來提交資料。<br/>
當我們對一個AJAX請求使用validate方法進行資料驗證時，Laravel並不會產生重定向的響應。取而代之的是一個<br/>
包含驗證錯誤的JSON響應，此 JSON 回應還會包含 422 的 HTTP 狀態碼。

## 表單請求驗證

## 手動建立驗證器

## 處理錯誤訊息

## 可用的驗證規則

## 有條件的增加規則

## 驗證陣列

## 自訂驗證規則