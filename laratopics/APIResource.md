# Laravel Eloquent API Resource

> ~~~
> 當你在建立 API 時，可能會需要一個位於 Eloquent 模型和實際回傳給使用者的 JSON 回應之間的轉換層。
> 而 Laravel 的資源類別可以讓你更直觀且容易地將模型及模型集合轉換成 JSON。
> ~~~

## 創建資源
要建立一個資源類別，可以使用 Artisan 指令 make:resource。預設，資源類別檔會被放在應用程式的<br/>
app/Http/Resources 目錄下，而資源類別則繼承自Illuminate\Http\Resources\Json\JsonResource。
```
php artisan make:resource User
```

### 資源集合
除了創建用於轉換個別模型的資源外，你還可以為了轉換模型集合來建立專用的資源。這可以讓你的回應包含<br/>
了與給定資源之整個集合所相關的連結和其他更多的Meta資訊。<br/>

要建立資源集合，你必須使用在創建資源時使用 **--collection** 這個旗標。另一種方式是直接在類別名稱上包含<br/>
Collection 這個字詞，這也會讓 Laravel 認為是要創建一個資源集合類別。另外，資源集合類別則是繼承自Illuminate\Http\Resources\Json\ResourceCollection。
```
php artisan make:resource Users --collection

php artisan make:resource UserCollection
```


## 概念簡述
> ~~~
> 這段是一個關於資源與資源集合的高階描述。不過，還是強烈建議閱讀文件的其他部分，才能更深入理解資源到底為你
> 提供了那些自訂的能力。
> ~~~

在深入了解編寫資源時有哪些可用選項前，讓我們先來看一下 Laravel 是如何地運用這些資源。資源類別表示一個需要<br/>
被轉換成JSON結構的模型。下面舉一個簡易的 User 資源類別。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

在發送回應時，每個資源類別都實作了一個 toArray 方法予以定義一組將轉換為 JSON 的屬性陣列。要注意一點，我們<br/>
能夠直接從 $this 變數來存取底層模型的屬性。因為為了方便存取，資源類別會自動代理屬性及方法以存取底層的模型<br/>
。資源一旦被定義好，就可以從路由或控制器來將它們回傳。
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

### 資源集合
如果你正要回傳的是資源集合或分頁回應，您可以在路由或控制器中創建資源實例時使用 collection 方法。
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```
當然，這麼做的話就不再允許附加可能需要與集合一起回傳的 meta data。如果仍想要自行客製資源集合的<br/>
回應的話，就得自行建立一個專門用來表示集合的資源集合類別。
```
php artisan make:resource UserCollection
```

一但資源類別被創建後，你就能輕鬆定義要包含在回應當中的 Meta 資料。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

而在資源集合類別創建後，就可以在路由或控制器中直接予以回傳
```
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

## 編寫資源
本質上來說，資源其時是很簡單的，他們只是在將給定的模型轉換成陣列。也因此，每一個資源類別都包含了<br/>
toArray 方法，其可用來將模型屬性轉換成對API友善且可返回給使用者的資料陣列。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

而一但資源定義好後，就可以直接從路油或控制器中直接返回
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

### 關聯
如果想要在回應中包含關聯資源，你只需要將它們新增到 toArray 方法來回傳。在下方的範例中，將使用 Post 資源<br/>
的 collection 方法來將使用者的部落格發佈新增到資源回應中。
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
PS. 如果只是想要在已經載入的情況下才引入關聯，可以參考"條件式關聯"一節的說明。

### 資源集合
相對於資源是單個模型轉換成陣列；資源集合則是將模型集合轉換成陣列。為每個模型類別都定義一個資源<br/>
集合類別並非絕對需要。因為所有資源都會提供 collection 方法來動態產生臨時的資源集合。
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```
但如果你需要自訂額外的Meta資訊，就需要自定義出資源集合類別出來。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

就像單一資源一樣，資源集合亦可以從路由或控制器中直接回傳
```
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

### 資料包裝
預設，當資源回應要轉換成 JSON 時，最外層資源將會被包裝在 data 鍵中。例如，通常資源集合的回應會像下方範例
```
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ]
}
```
如果想要停用包裝最外層的資源，可以在基本的資源類別上使用 withoutWrapping 方法。一般來說，你應該在<br/>
AppServiceProvider 或其另一個會在每個請求中都被載入的服務提供者上頭進行調用。
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Http\Resources\Json\Resource;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Resource::withoutWrapping();
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
PS. withoutWrapping 方法只會影響最外層的回應，並不會移除你手動新增到自己的資源集合的 data 鍵。

### 包裝巢狀資源
對於如何包裝資源關聯，你擁有完全自由的決定權。只是如果想要所有資源集合都被包裝在 data 鍵，你都應該<br/>
為每一個資源定義其資源集合並於 data 鍵中返回集合。

當然，你可能會懷疑如此是否會導致最外層的資源被包裝在兩個 data 鍵中。無須擔心，因為Laravel 永遠不會讓你<br/>
發生資源被執行二次包裝的結果，所以不必擔心你要轉換的資源集合之巢狀層數。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return ['data' => $this->collection];
    }
}
```

### 資料包裝與分頁
當在資源回應中返回分頁集合時，Laravel 會強制把資源資料包裝在 data 鍵中，即便調用了 withoutWrapping 方法也<br/>
一樣。這是因為分頁回應總是包含了關於分頁狀態資訊的 meta 和 links 鍵：
```
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

### 分頁
你可能總是會將分頁實例傳入到資源或自訂資源集合的 collection 方法。
```
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

而分頁回應也總會包含了關於分頁狀態且以 meta 和 links 為鍵的資訊。
```
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

### 條件式屬性
有時你可能希望只有在滿足指定條件時在才在資源回應中引入特定屬性。例如，你會希望只有目前使用者是管理員時<br/>
才引入特定屬性值。Laravel 提供了各式的輔助方法來支持這樣的情境。其中，when 方法就可被用於有條件的新增屬性<br/>
到資源回應中。
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
在上述範例中，僅在 Auth::user()->isAdmin() 方法回傳 true 時，secret 鍵才會包含在最後的資源回應中並返回；<br/>
相反的，如果回傳 false時，secret 鍵就會在反會給客戶端前被移出。when 方法讓你在建構陣列時，不需要在使用<br/>
條件語句就能明確地定義你的資源。<br/>

when 方法可以接收閉包作為第二個參數，用以回傳一個計算過的結果值。
```
'secret' => $this->when(Auth::user()->isAdmin(), function () {
    return 'secret-value';
}),
```

### 合併條件式屬性
有時候，在相同的條件判斷下，你可能有數個屬性需要包含到資源回應中。這個時候，你可以改用 mergeWhen 方法<br/>
來引入多個屬性。
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen(Auth::user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
同樣地，如果給定條件被判定為 false，在發送給客戶端前，這些屬性就會從資源回應中被完整移除。
> ~~~
> 注意!! mergeWhen 方法不應該被用於混合字串和數字鍵的陣列中。此外，它也不該被用於沒有依序排列的數字鍵陣列。
> ~~~

### 條件式關聯
除了有條件地載入屬性外，也可以根據關聯是否已經載入到模型上來決定是否在資源回應中引入該關聯。這可以讓你<br/>
改從控制器決定應該載入哪一個關聯到模型上，進而讓資源能更輕易地在確實要被載入時才引入。<br/>

而最終，這也可以較容易避免在你的資源中發生「N+1」的查詢問題。而 whenLoaded 方法就是被用於有條件地載入<br/>
關聯。為了避免不必要的關聯載入，該方法可以接受關聯名稱，而非關聯本身。
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
上述範例中，如果關聯沒有被載入，則在傳送到客戶端前，posts 鍵就會從資源回應中被完全被移除。<br/>

#### 條件式樞紐資訊
除了有條件地在資源回應中引入關聯資訊，你還可以使用 whenPivotLoaded 方法來有條件地從多對多關聯中介表<br/>
中引入資料。whenPivotLoaded 方法接受中介表名稱作為它的第一個參數。第二個參數則可接受閉包函數，它定義<br/>
了當模型上的中介樞紐資訊有效時要回傳的值。
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_users', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

### 增加 Meta 資訊
某些 JSON API 標準會有新增額外 Meta 資料到資源和資源集合回應中的需求。常見的像是 links 或關於描述資源<br/>
本身的資料。如果需要回傳有關資源的 Meta 資料，只需將它直接引入到 toArray 方法即可。例如，在轉換資源集合時<br/>
需要引入 link 資訊時：
```
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request
 * @return array
 */
public function toArray($request)
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```
PS. 當資源回應包含分頁時，你無須擔心意外覆寫了Laravel自動增加的 links 或 meta 鍵。你所定義之任何額外<br/>
的 links 都會自動與分頁器提供的 links 進行合併。

#### 上層的 Meta 資訊
你可以槽狀地載入關連資源，但總是會存在一個頂層資源，而有時你就希望只有資源被當成最頂層資源回傳時再<br/>
引入某些 Meta 資料。通常，這會包含整個回應資訊的 Meta 資料。要定義這樣的 Meta 資料，你需要在你的資源<br/>
類別中實作 with 方法。而只有當資源是以最頂層資源被回傳時，with 方法內定義返回的 Meta 資料陣列會引入到<br/>
資源回應中。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return parent::toArray($request);
    }

    /**
     * Get additional data that should be returned with the resource array.
     *
     * @param \Illuminate\Http\Request  $request
     * @return array
     */
    public function with($request)
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

#### 建構資源時新增 Meta 資料
當在路由或控制器中建構資源實例時，你也可以新增 Meta 資料到資源頂層。你可以用 additional 方法，其在各類型<br/>
資源都可使用。該方法會接受應該要被新增到資源回應中的資料陣列。
```
return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
```

## 資源回應
正如前面多次提及過的，資源可以從路由或控制器中直接回傳
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

然而，有時你的確需要在回應發送到客戶端前自訂要輸出的 HTTP 回應。有兩種方法可以達到這種目的。<br/>

 - 在資源類別上鏈結調用  response 方法，該方法會返回 Illuminate\Http\Response 實例，這可以讓你完全控制<br/>
    回應的標頭。
```
use App\User;
use App\Http\Resources\User as UserResource;

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```

 - 在資源類別裡實作 withResponse 方法，這個方法會在資源作為頂層資源時被調用。
```
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * Customize the outgoing response for the resource.
     *
     * @param  \Illuminate\Http\Request
     * @param  \Illuminate\Http\Response
     * @return void
     */
    public function withResponse($request, $response)
    {
        $response->header('X-Value', 'True');
    }
}
```
