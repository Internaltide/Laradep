# Laravel Authorization(授權)

> ~~~
> 除了提供即用的認證服務外，Laravel 還提供了一個簡易的方法予以授權使用者可以針對特定資源執行操作。
> 就像身分驗證一樣，Laravel 的授權方法其實很簡單，其有兩種主要的方法就是 Gates 跟政策。
>
> Gates 與政策的關係就如同路由和控制器一樣。Gates 提供了簡單且採用閉包的方式去授權，而政策則像控制器，
> 群聚一組圍繞著特定模型或資源的邏輯。
>
> 在應用程式中，你並無需在 Gates 與政策之間做選擇。大多應用程式常常會同時使用 Gates 和政策，而 Laravel
> 也推薦這樣做。Gates 通常被應用在與模型、資源無關的操作上，例如查看後台儀表板。相反的，政策則是用在
> 與模型、資源相關的操作上。
> ~~~

## Gates
### 撰寫 Gates
Gates 是用來確認是否授權指定的行為給使用者，通常是在 App\Providers\AuthServiceProvider 的類別中使用 Gate<br/>
facade 來定義。Gates 的第一個參數主要接受使用者實例，並且可以接受額外的參數，像是 Eloquent 關聯模型。
```
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
}
```

Gates 也可以使用類似 **Class@method** 這樣風格的回呼字串，就像是路由中指定控制器那樣。
```
/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', 'App\Policies\PostPolicy@update');
}
```

#### 資源 Gates
你也可以使用 resource 方法一次定義多個 Gate 授權
```
Gate::resource('posts', 'App\Policies\PostPolicy');
```

上述的定義等價於下面的授權定義。
```
Gate::define('posts.view', 'App\Policies\PostPolicy@view');
Gate::define('posts.create', 'App\Policies\PostPolicy@create');
Gate::define('posts.update', 'App\Policies\PostPolicy@update');
Gate::define('posts.delete', 'App\Policies\PostPolicy@delete');
```

view、create、update 和 delete 這些代表CRUD的行為能力預設就會定義的。而你還能透過陣列作為第三參數<br/>
傳送到 resource 方法並予以複寫或增加預設的行為能力。陣列中的鍵值用以定義行為能力的名稱；陣列元素值則<br/>
對應方法名稱。例如，下列的程式碼將建立兩個新的 Gate 授權定義 => **posts.image** 和 **posts.photo**。
```
Gate::resource('posts', 'PostPolicy', [
    'image' => 'updateImage',
    'photo' => 'updatePhoto',
]);
```

### 授權判斷
要用 Gates 來判斷特定行為的授權，你應該用 allows 或 denies 方法。注意!! 你無需再傳遞認證通過的使用者<br/>
實例。因為 Laravel 會自動注入已認證通過的使用者實例到 Gate 閉包。
```
if (Gate::allows('update-post', $post)) {
    // The current user can update the post...
}

if (Gate::denies('update-post', $post)) {
    // The current user can't update the post...
}
```

因為預設自動注入的是當下通過認證的使用者實例，所以若是要檢查是否對特定使用者授權行為時，應該<br/>
要在 Gate 後鏈結調用 **forUser** 方法。
```
if (Gate::forUser($user)->allows('update-post', $post)) {
    // The user can update the post...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // The user can't update the post...
}
```

#### 攔截 Gates 檢查
有時候您可能會想要授權特定用戶擁有所有操作的能力。這時你可以使用before方法定義一個回調函數，其會在所有<br/>
授權檢查前就被執行。而如果該回調函數返回的是非 Null 的結果，其結果就會被直接認為是最後的授權檢查結果。
```
Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

你還可以使用 after 方法定義一個在授權檢查後才執行的回調函數，只是你無法從該回調函數來變更檢查的結果。<br/>
(作為預設的授權檢查？)
```
Gate::after(function ($user, $ability, $result, $arguments) {
    //
});
```

## 建立授權政策
政策就是一種組織特定模型或資源的授權邏輯之類別。例如，你有一個部落格的應用，你可能就會有一個 Post 模型<br/>
及對應用來授權使用者對該資源存取的 PostPolicy ，就像創建或更新 Post。

你可以透過使用 Artisan 指令 make:policy 來創建政策類別，創建的類別會放在 app/Policies 這個目錄下。不過，<br/>
預設指令建立的類別會是空實作的類別，如果想要預先建立CRUD等相關的政策方法，可以使用 **--model** 這個<br/>
選項。
```
// 建立空的政策類別
php artisan make:policy PostPolicy

// 建立函基本CRUD政策方法的的政策類別
php artisan make:policy PostPolicy --model=Post
```
PS. 所有政策類別的建構是也都支援型別提示以注入依賴，服務容器會自動解析。

### 註冊政策
當有政策存在時，就必須為其進行註冊。一個新的 Laravel 應用程式都會內建 AuthServiceProvider 服務提供者並且<br/>
包含了 policies 屬性，其用來映射 Eloquent 模型與其對應的政策。而註冊政策這個動作則會指示 Laravel 在進行<br/>
對特定模型授權特定行為時應該要依據那一種政策。
```
<?php

namespace App\Providers;

use App\Post;
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

## 撰寫授權政策
### 政策方法
一但創建的政策被註冊後，就可以開始為各種行為的授權來增加對應的方法。例如，讓我們在 PostPolicy 類別<br/>
中定義一個可以決定特定使用者是否能更新特定 Post 的 **update** 方法。update 方法接受使用者實例與 Post 實例<br/>
作為其參數，並應返回 true 或 false 以表示使用者是否被授權更新給定的  Post。所以，在下面的範例中，我們將<br/>
返回使用者 id 與 Post 的 user_id 是否匹配的結果。換句話說，就是只有該 Post 的作者才能進行更新。
```
<?php

namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```
你可以在此政策類別中繼續定義其他方法以作為各種行為所需要的授權邏輯。舉例來說，你可以定義 view <br/>
或 delete 方法來授權  Post 的其他行為。而且記得，你可以任意地為政策方法定義自己想要的名字。

### 沒有模型的方法
### 政策篩選器

## 利用政策對行為進行授權