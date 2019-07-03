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
除了模板繼承與顯示資料功能以外，Blade 還為常見的 PHP 控制結構提供了等價的快捷語法，像是<br/>
條件陳述式和迴圈等。這些快捷語法提供了乾淨、簡潔的方式來使用 PHP 的控制結構，同時也保留<br/>
對應於原生 PHP 中熟悉的語法。

### If 語句
你可以運用 **@if**、**@elseif**、**@else** 及 **@endif** 等指令建構 if 陳述式。這些指令<br/>的功用就如同在 PHP 中的 if 用法。
```
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

而為了方便，Laravel 還提供了更多的語法<br/>

**@unless**
```
@unless (Auth::check())
    You are not signed in.
@endunless
```

對應 PHP 同名函數功能的 **@isset** 和 **@empty**
```
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### 驗證型指令
**@auth** 和 **@guest** 這兩個指令用來快速檢查當下使用者是否通過驗證，或者只是個訪客用戶
```
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

倘若有需要，還能在使用 **@auth** 和 **@guest** 指令時指定認證守衛來確認身份
```
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### 區塊型指令
使用 **@hasSection** 指令則可判斷某區塊內是否含有內容
```
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

### Switch 語句
你可以使用 **@switch**、**@case**、**@break**、**@default** 以及 **@endswitch** 這些<br/>
指令來建構 Switch 陳述式。
```
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### Loop 語句
除了條件句之外，Blade 亦支援了原生 PHP 的迴圈語法
```
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```
PS. 當執行迴圈時，還可以使用迴圈變數來取得迴圈的訊息。例如想知道目前是在迴圈的第一次<br/>
還是最後一次。<br/>

有時候在使用迴圈的時候，你可能會想要終止或跳過當前的迭代，此時可以像下方範例一樣這麼做
```
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

同上述範例，你也可以直接將條件句放入到指令內，語法上會更簡潔
```
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### 迴圈變數
迴圈執行中時，在迴圈內可以使用迴圈變數 $loop。它可以提供存取一些有用的迴圈資訊，像是<br/>
迴圈索引以及當前迴圈是第一次還是最後一次迭代。
```
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

若你是在巢狀迴圈內部，則可以透過 **parent** 屬性來取得上一層的迴圈變數 **$loop**
```
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

#### 以下是迴圈變數所包含的有用資訊
| 屬性 | 描述 |
|:-------|:-------|
| $loop->index |	目前迭代迴圈的索引 (索引從0開始) |
| $loop->iteration |	目前的迭代迴圈 (從1開始) |
| $loop->remaining |	迴圈剩餘的迭代 |
| $loop->count |	迭代陣列內元素總數 |
| $loop->first |	判斷是否是第一次迭代 |
| $loop->last |	判斷是否是最後一次迭代 |
| $loop->depth |	目前迴圈的巢狀深度 |
| $loop->parent |	在巢狀迴圈中，代表父迴圈的迴圈變數 |

### 註解
Blade 也允許你在視圖進行註解。然而，與 HTML 註解不一樣的是，Blade 註解並不會隨同 HTML
返回到網頁上。
```
{{-- This comment will not be present in the rendered HTML --}}
```

### PHP
某些場景下，在視圖內嵌入 PHP 代碼是有其效用的。你可以透過 @php 指令在模板中直接執行一段<br/>
PHP 代碼。
```
@php
    //
@endphp
```
PS. 雖然 Blade 提供了這個功能，但若過度使用的話，可能就是一個視圖參雜過多邏輯的訊息，<br/>
你的應用程式也可能因此變得雜亂不優雅。

## 載入子視圖
@include 這個 Blade 指令可以讓你引入其他的 Blade 視圖，主視圖可用的變數亦可在子視圖使用。
```
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

不過，儘管引入的子視圖會繼承父視圖中的所有資料，你仍可傳遞額外資料到引入的子視圖。
```
@include('view.name', ['some' => 'data'])
```

當然，如果你嘗試載入一個不存在的視圖，Laravel 會拋出錯誤。因此，若你要載入的子視圖有<br/>
可能不存在，那應該改用 @includeIf 這個指令。只有子視圖存在時才會執行載入的動作。
```
@includeIf('view.name', ['some' => 'data'])
```

如果你是要在特定條件下才載入子視圖時，可以使用 @includeWhen 指令
```
@includeWhen($boolean, 'view.name', ['some' => 'data'])
```

你還可以使用 @includeFirst 指令，在一群指定的視圖中載入第一個存在的視圖。
```
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

> ~~~
>注意!! 你應避免在 Blade 視圖中使用常數 __DIR__ 和 __FILE__，因為他們返回的會是視圖快取
> 的位置。
> ~~~

### 為集合渲染視圖
你可以使用一行 Blade @each 指令就將迴圈及子視圖載入兩個動作結合在一起。
```
@each('view.name', $jobs, 'job')
```
上述指令會對陣列或集合中的元素資料進行渲染，第一個參數就是每一次渲染所使用的局部<br/>
視圖，第二個參數則是要迭代的陣列或集合，第三個參數為迭代時被分配至視圖中的變數名稱。<br/>
所以，舉例來說，如果你正迭代一個 jobs 陣列，通常你會希望在局部視圖中透過名稱為 job 的變<br/>
數來存取每一個 job。另外，目前迭代的鍵值則會在局部視圖以 key 變數來存取。

而你還能為 @each 指令傳遞第四個參數，其作為當元素資料為空時所使用的局部視圖。
```
@each('view.name', $jobs, 'job', 'view.empty')
```
> ~~~
> 要注意!! 視圖透過 @each 渲染時並不會繼承父視圖的變數。如果子視圖需要這些變數，你必須
> 改用 @foreach 和  @include。
> ~~~

## Stacks
Blade 可以讓你使用 @push 指令將資料推送到命名堆疊中，該堆疊還以在其他視圖或 Layout 中<br/>
渲染。這對於在子視圖中指定任何需要的 Javascript 函式庫時特別地有用。
```
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

你可以根據需求多次推送資料到堆疊中。要渲染堆疊裡的完整內容時，只要將堆疊名稱傳遞進<br/>
@stack 指令即可。
```
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

另外，你還可以利用指令 @prepend 來將內容添加到堆疊的開頭。
```
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

## 服務注入
Blade 的 @inject 指令用於從服務容器中將特定服務注入到視圖中來使用。傳遞給 @inject 的第<br/>一個參數為存放該服務用的變數名稱，第二個參數則為想要解析的服務類別或介面的名稱。
```
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

## Blade 擴充
Blade Facade 還提供了 directive 方法供你客製你的 Blade 指令。當 Blade 編譯器遇到自訂的<br/>
指令時，它就會呼叫 directive 內的回調。<br/>

下面範例中將會建立一個接受參數 $var 的 **@datetime** 的 Blade 指令。其用途為對給定的<br/>
$var 進行格式化並返回一個 DateTime 實例。
```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
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
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
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

如你所見，我們會串接 format 方法到任何傳入指令的表達式。因此在範例中的指令最後對應<br/>
產生的 PHP 程式碼會像是下面這樣。
```
<?php echo ($var)->format('m/d/Y H:i'); ?>
```
> ~~~
> 當 Blade 指令的邏輯被更新後，你會需要刪除所有的 Blade 視圖快取。被快取的 Blade 視圖可
> 以藉由使用 Artisan 的 view:clear 指令來移除。
> ~~~

### 自訂 If 語句
當要自訂一個簡單的條件陳述句時，編寫客製指令的方式有時可能會比實際需要的還複雜。為此，<br/>
Blade 提供了 **Blade::if** 方法，其允許你使用閉包快速地自訂 Blade 條件陳述句指令。例如<br/>
，讓我們建立一個檢查當下應用環境的條件式指令，而我們可以在AppServiceProvider 的 boot 方<br/>
法內部進行定義。
```
use Illuminate\Support\Facades\Blade;

/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

而一旦自訂了 Blade 條件句，我們就能在視圖上輕易地使用它們。
```
@env('local')
    // The application is in the local environment...
@elseenv('testing')
    // The application is in the testing environment...
@else
    // The application is not in the local or testing environment...
@endenv
```
