# Laravel Blade Template

> ~~~
> Blade 是 Laravel 所提供的一個簡單有強大的模板引擎。不同於其他知名的 PHP 模板引擎，Blade 並不會限制
> 你必須在視圖中使用 PHP 程式碼。事實上，所有的 Blade 視圖都會被編譯成一般的 PHP 程式碼並快取直到
> 被更動為止。這也意味著 Blade 不會對你的應用程式產生負擔。
> Blade 視圖檔案會以 **.blade.php** 作為附檔名，並且存放於 resources/views 目錄下。
> ~~~

## 模板繼承
### 定義 Layout
使用 Blade 的兩個主要優點就是繼承與繼承。讓我們來看個簡單的例子，首先先檢查主要的頁面佈局為何，<br/>
因為多數網頁應用程式在不同頁面都會使用相同的佈局方式，這就方便我們將該佈局定義成單一的 Blade 視圖，<br/>
以便於將來重複使用，也可以適度降低各頁面的複雜度。
```
<!-- Stored in resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```
如你所見，上面的範例內容包含了傳統的 HTML 語法。然而，請注意到 **@section** 跟 **@yield** 這兩個指令。<br/>
@section 就如同其名，是用來定義單一區塊的內容；相反地，@yield 則是被用來顯示指定區塊的內容。<br/>

現在，我們定義好了應用程式的Layout，接著，就能繼續定義一個繼承此Layout佈局的子頁面。

### 繼承 Layout
當我們在定義子頁面時，我們可以使用 Blade 的 **@extends** 指令，其可用來定義要繼承自那一個Layout。而該<br/>
繼承了 Layout 的視圖則可以使用 @section 指令定義要注入到 Layout section 中的內容。記住，@section('content') <br/>
定義的內容就會出現在上述範例中使用了 @yield('content') 指令的位置上。
```
<!-- Stored in resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```
另外，在上述範例中，側邊欄部分則先使用了 @parent 指令，再以附加的形式將內容加上去。而 @parent 指令處就<br/>
會在視圖渲染時被置換成 Layout 的內容。因此，最後側邊欄應該會顯示如下：
```
This is the master sidebar.

<p>This is appended to the master sidebar.</p>
```
> ~~~
> 有一點澄清一下，這裡的 sidebar 結束時是使用 @endsection 而不是 @show。 @endsection 指令用來定義一個區塊，
> 而 @show 則會直接產生這個區塊的內容。
> ~~~

你還可以在路由中直接使用全域的 view 輔助方法來返回一個 Blade 視圖
```
Route::get('blade', function () {
    return view('child');
});
```

## 元件與插槽
元件之於插槽就好像 Layout 之於 Section ，它們都提供了類似的好處，但某些人可能認為元件跟插槽的概念更容易讓<br/>
人理解。首先，想像有一個可以再應用程式中重複使用的 alert 元件。
```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```
其中，**{{ $slot }}** 變數是我們希望插入到元件中的內容。而這個時候，我們能使用 Blade 中的 @component 指令<br/>
來建構這個元件。
```
@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
```

有時候替元件定義多個插槽是很有幫助的。讓我們修改一下先前的 alert 元件，讓它能再注入一個標題。
```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>

    {{ $slot }}
</div>
```
現在，我們可以透過使用 **@slot** 指令來將內容注入到命名為 title 的插槽。而任何未包含在 @slot 指令間的內<br/>
容則會被歸類成 $slot 內容的一部分。
```
@component('alert')
    @slot('title')
        Forbidden
    @endslot

    You are not allowed to access this resource!
@endcomponent
```

### 傳遞額外的資料到元件
有時候會需要傳遞額外的資料到元件中。為此，你可以傳遞陣列格式的資料作為 @component 指令的第二個參數。<br/>
而這些資料將會以變數的形式提供給元件的模板來使用。
```
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

### 別名元件
如果你的 Blade 元件是存放在子目錄下，可能會希望能夠為其另取別名以方便存取。以一個存放在<br/>
resources/views/components 下的 alert.blade.php 元件檔為例，你可以使用 Blade Facade 的 component 方法<br/>
將名稱由 components.alert 改為 alert。而一般來說，你應該在 AppServiceProvider 的 boot 方法進行實作。
```
use Illuminate\Support\Facades\Blade;

Blade::component('components.alert', 'alert');
```
而一但元件設定了別名，你就可以使用同名指令渲染該元件
```
@alert(['type' => 'danger'])
    You are not allowed to access this resource!
@endalert
```

另外，如果元件並未設置插槽，則可以忽略元件參數的傳遞
```
@alert
    You are not allowed to access this resource!
@endalert
```

## 資料顯示
要將傳入到 Blade 視圖的資料顯示出來，你應該使用大括號將變數包裹起來。例如下面這個例子：
```
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```
你應該如下面的方式將名稱為 name 的變數內容顯示出來
```
Hello, {{ $name }}.
```
當然，你並未被限制只能顯示傳入 Blade 的變數內容，你也可以直接調用 PHP 原生函數並顯示其結果。<br/>
事實上，你可以放置任何你需要的 PHP 程式碼到 Blade 顯示語法裡面。
```
The current UNIX timestamp is {{ time() }}.
```
> ~~~
> 記住，Blade 的 {{}} 語法會自動以 PHP 的 htmlentites 函式進行 XSS 攻擊的防禦。
> ~~~

#### 顯示未經跳脫的資料
預設，Blade 的 {{}} 語法就會使用htmlentites 函式來防禦 XSS 攻擊，如果你不想要內容經過跳脫的處裡，須改用<br/>
下列方式進行顯示。但這樣一來，你就得自行注意使用者的內容，避免XSS攻擊。
```
Hello, {!! $name !!}.
```

#### 渲染 JSON
有時候為了方便初始化JS變數，會直接傳送陣列進入視圖以便轉換成JSON格式。例如：
```
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

或者，你可以直接使用更為直覺的 Blade 指令 **@json** 來取代 PHP json_encode 原生函數的使用。
```
<script>
    var app = @json($array);
</script>
```

#### Html 實體編碼
預設，Blade 跟輔助方法 **e** 皆會對 Html 實體執行雙重編碼。如果要停用這種功能，可以在 AppServiceProvider <br/>
的 boot 方法中使用 Blade::withoutDoubleEncoding 方法予以停用。
```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
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
        Blade::withoutDoubleEncoding();
    }
}
```

### Blade 與 Javascript 框架
由於許多 JavaScript 框架會使用大括號在瀏覽器中顯示特定的表達式。你可以使用 @ 符號來告知Blade 引擎該表達<br/>
式應該維持原樣。舉個例子：
```
<h1>Laravel</h1>

Hello, @{{ name }}.
```
在該範例中，@ 符號將會被 Blade 移除。而 Blade 則會保留 {{ name }} 表達式，如此一來便可讓其它 JavaScript <br/>框架所應用。

#### @verbatim 指令
如果要透過 Javascript 框架渲染的區域很大，你可以將 HTML 內容放到 @verbatim 中，如此一來<br/>
就不需要在每個 Blade echo 語句前加上 @ 符號。
```
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## 控制結構


## 載入子視圖
## Stacks
## 服務注入
## Blade 擴充