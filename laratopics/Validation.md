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
如同前面所提，當驗證失敗後將自動產生重定向到表單提交前的位置。另外，所有的錯誤訊息都會自動快閃到<br/>
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
(這裡所謂的null值視為無效應該是指被視為是無效值而判定不存在，所以對應的規則就不會執行檢查)
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
### 建立表單請求
為了滿足更多更複雜的驗證情境，你可能會需要建立**表單請求**，一種包含驗證邏輯的客製請求類別。你可以使用<br/>
make:request這個Artisan指令來建立一個請求類別，而創建的表單請求類別會放在app/Http/Requests這個目錄下，而<br/>
初始的表單請求類別會包含rules與authorize兩種方法。

#### 指令創建
```
php artisan make:request StoreBlogPost
```

#### 驗證規則函數
將你的驗證規則邏輯放在rules函數中，該函數亦可藉由型別提示來將依賴注入，服務容器會自動進行解析。
```
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

#### 執行驗證規則
要套用表單請求類別中的驗證規則，只需要在控制器中利用型別提示將表單請求物件注入，其會在控制器方法被呼叫前<br/>
進行驗證，因此不會因為驗證邏輯而把控制器弄得一團亂。當驗證失敗時，重定向或錯誤訊息快閃到session等行為則一樣。
```
/**
 * Store the incoming blog post.
 *
 * @param  StoreBlogPost  $request
 * @return Response
 */
public function store(StoreBlogPost $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```

#### 為表單請求加上驗證後的掛勾
如果你想增加一個掛勾並放置於表單驗證之後，可以透過定義withValidator方法，並在內部調用after來新增掛勾。<br/>
withValidator方法接受一個完整建構的驗證器，讓你可以在驗證規則實際被執行前呼叫驗證器的任何方法。(執行前???)
```
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

#### 授權表單請求
表單請求類別所包含的 authorize 方法，可以用來確認使用者是否真有其權限可以更新特定資料。例如，當使用者嘗試<br/>
更新部落格文章的評論時，就可以先判斷這是否是他的評論？
```
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```
由於表單請求類別是繼承自基底的請求類別，因此你也可以調用 **user** 來取得已驗證的使用者實例。同時我們將焦點<br/>
轉移到上述範例中所調用的 **route** 方法，該方法讓你可以取得呼叫路由時的 URI 參數，就像下方定義的 {comment} 參數一樣。
```
Route::post('comment/{comment}');
```
如果 **authorize**  方法回傳false，Laravel會自動回傳一個 403 的 HTTP 回應，且不會執行控制器方法。當然，<br/>
如果你打算在應用程式的其他部分處理授權邏輯，可以讓 authorize 方法回傳 true。
```
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```
PS. authorize方法亦支援型別提示以注入依賴。

#### 自訂驗證失敗的錯誤訊息
透過覆寫 **messages** 方法，你就可以自訂自己的錯誤訊息。該方法必須回傳一個包含成對的屬性與規則及對應錯誤
訊息的陣列。
```
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    // '屬性.規則' => '錯誤訊息'
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
```

## 手動建立驗證器
如果不想要使用請求物件的validate方法，你可以改用Validator Facade來自訂驗證器，而Validator Facade的make方法<br/>
就可以用來返回一個新的驗證器實例。
```
<?php

namespace App\Http\Controllers;

use Validator;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Store the blog post...
    }
}
```
傳遞給make方法的第一參數為驗證用的資料，第二參數則為驗證規則。<br/>

在驗證失敗後，可以調用withErrors方法將錯誤訊息快閃到session中，而在調用該方法且重定向後，就會將$errors<br/>
綁定到視圖中共享使用。withErrors方法可接受驗證器、MessageBag或是PHP陣列。

### 自動重定向
在手動建立驗證器後，若仍想維持原先使用請求物件之validate驗證方法所產生的重定向功能，可以改調用驗證器物件<br/>
自身的validate方法。
```
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

### 命名錯誤清單
當網頁含有一個以上的表單時，若可以替每個錯誤的 MessageBag 進行命名，這對於在視圖中取得特定表單的錯誤<br/>
訊息會有幫助。你可以在調用withErrors這個方法時，透過其第二個參數來定義其名稱。
```
return redirect('register')
            ->withErrors($validator, 'login');
```
接著就可以從 $errors 變數中取得已命名的 MessageBag 實例
```
{{ $errors->login->first('email') }}
```

### 驗證後的掛勾
驗證器允許你附加一個可以在驗證完成後執行的回呼函數。這可以讓你做進一步驗證甚至在錯誤訊息集合中增加更多<br/>
的錯誤訊息。你可以使用驗證器實例來調用 after 方法即可。
```
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```

## 處理錯誤訊息
在驗證器調用了errors方法後，你會得到一個Illuminate\Support\MessageBag實例。該實例包含了幾個便利的方法來<br/>
處理錯誤訊息，而所有視圖中共用的 $errors 變數就是 MessageBag 類別的實例。

### 取出特定欄位的第一個錯誤訊息
```
$errors = $validator->errors();

echo $errors->first('email');
```

### 取出特定欄位的所有錯誤訊息
```
foreach ($errors->get('email') as $message) {
    //
}
```
如果你驗證的表單欄位是個陣列，可以用 * 字元來取出每個陣列元素的訊息
```
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

### 取出所有欄位的所有錯誤訊息
```
foreach ($errors->all() as $message) {
    //
}
```

### 判斷特定欄位是否有錯誤訊息
```
if ($errors->has('email')) {
    //
}
```

### 客製錯誤訊息
如果有需要，你可以自訂驗證的錯誤訊息來取代預設的錯誤訊息。有多種指定自訂訊息的方法。

#### 把自訂訊息傳到 Validator::make 方法的第三個參數
```
$messages = [
    'required' => 'The :attribute field is required.',
];

$validator = Validator::make($input, $rules, $messages);
```
上述例子中，**:attribute** 即為欄位名稱的佔位符，在驗證過後就會被實際欄位名稱取代。<br/>

其他可以在驗證訊息中使用的佔位符
```
$messages = [
    'same'    => 'The :attribute and :other must match.',
    'size'    => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in'      => 'The :attribute must be one of the following types: :values',
];
```

#### 指定自訂訊息給特定的屬性
有時候你可能只想要對特定的欄位自訂錯誤訊息。你可以在屬性名稱後加上「.」符號，並加上指定的驗證規則
```
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```

#### 在語系檔中指定自訂訊息
大多數的情況，你可能會希望在語系檔中指定自訂的訊息，而不是被直接傳進驗證器。你可以透過把訊息加到<br/> resources/lang/xx/validation.php 語系檔中的 custom 陣列來達成目的。
```
'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```

#### 在語系檔中指定自訂屬性
如果你想把驗證訊息裡的 :attribute 取代成自訂的屬性名稱，可以在 resources/lang/xx/validation.php 語言檔中<br/> 的attributes 陣列來指定名稱。
```
'attributes' => [
    'email' => '電子信箱',
],
```

## 可用的驗證規則
[詳見官方文件](https://laravel.com/docs/5.6/validation#available-validation-rules)

## 有條件的增加規則
### 存在時才驗證
有時候你會希望當輸入資料有該欄位時才進行驗證，要達成此目的，需要增加 sometimes 規則到到規則清單中。
```
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```
在上面範例中，email 欄位的驗證，只會在 $data 陣列含有此欄位時才會執行檢查。

### 複雜的條件驗證
有時候你希望以更複雜的條件邏輯來增加驗證規則。例如：<br/>
 - 希望某個欄位，在另一個欄位的值超過 100 時才為必填。
 - 希望當某個指定欄位有值時，另外兩個欄位要符合特定值。

添加這樣複雜的驗證規則並不困難，首先，先利用你熟悉的方式建立一個驗證器實例：
```
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```
接著，假設網頁應用程式是為了遊戲收藏者而設計，對於收藏超過100款遊戲的收藏家，我們會希望他們填寫擁用這麼<br/>
多遊戲的原因，換句話說就是理由這個欄位會變成了必填。此時我們可以在 Validator 實例上調用 sometimes 方法，<br/>
其中，第一參數為欄位名稱、第二參數為驗證規則、第三參數為一個閉包函數。當閉包函數返回true時，該驗證規則<br/>
才會被加入。
```
$v->sometimes('reason', 'required|max:500', function ($input) {
    return $input->games >= 100;
});
```

利用sometimes這個方法可以輕鬆的建立複雜的條件驗證，也可以一次對多個欄位增加驗證規則
```
$v->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
```
PS. 傳入閉包的 $input 參數是 Illuminate\Support\Fluent 實例，可以用來取得輸入的資料和檔案。

## 驗證陣列
驗證從表單提交過來的陣列資料並不困難，可以使用「.」符號來驗證陣列中的屬性。比如說傳入的 HTTP 請求包含了<br/>
photos[profile]這樣的資料時，就可以像下方這樣驗證。
```
$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

也可以驗證陣列中的每個元素。例如，驗證給定的某欄位之陣列資料中的 e-mail 都是唯一。
```
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

同樣地，也可以在語系檔中用 * 字元來定義驗證訊息。
```
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique e-mail address',
    ]
],
```

## 自訂驗證規則

### 使用規則物件
Laravel提供了各式各樣的驗證規則，然而，你可能會想定義一些自己的規則。你可以使用規則物件來註冊一個<br/>
客製的驗證規則，而使用Artisan指令make:rule就可以初始化一個新的規則物件並存放到app/Rules目錄下。<br/>

假設我們建立一個規則物件用來驗證字串是否為大寫
```
php artisan make:rule Uppercase
```

一但規則物件被建立後，我們就可以著手定義它的行為。一個規則物件會包含**passes** 和 **message**兩種物件。<br/>
 - passes 方法接收屬性值和名稱，被用來定義驗證規則，會根據屬性值是否合法來回傳 true 或 false。
 - message 方法則回傳驗證失敗時使用的錯誤訊息。<br/>
 ```
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
 ```
當然，你可以在message方法中使用輔助方法trans來從語係檔中擷取錯誤訊息。
```
/**
 * Get the validation error message.
 *
 * @return string
 */
public function message()
{
    return trans('validation.uppercase');
}
```
定義好規則後，可以藉由傳遞規則物件實例來附加到其他的驗證規則中
```
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

### 使用閉包
如果你只需要在應用程式中使用某客製規則一次而已，沒有重複使用的需求，你可以簡單的使用閉包函數來達成目的。<br/>
該閉包函數可以接受屬性的名稱與值，也接受一個如範例中一樣的回調函數$fail，其會在驗證失敗後被調用。
```
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function($attribute, $value, $fail) {
            if ($value === 'foo') {
                return $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

### 使用擴充方法
另一個註冊自訂規則的方式為使用 Validator Facade 的 extend 方法，我們可以在服務提供者內部使用該方法進行<br/>
自訂規則的註冊。
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
            return $value == 'foo';
        });
    }

    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```
自訂的驗證閉包接收四個參數：驗證的屬性名稱、屬性的值、傳入驗證規則的參數陣列及驗證器實例。除了使用閉包，<br/>
你也可以傳入類別和方法到 extend 方法中而非閉包。
```
Validator::extend('foo', 'FooValidator@validate');
```

#### 定義錯誤訊息
你也需要對客製的驗證規則定義其錯誤訊息，可以使用行內的自訂訊息陣列或在驗證語系檔中添加對應的訊息條目。<br/>
但記住!! 這個訊息應該被放在陣列的第一層，而不是放在對應特定屬性錯誤訊息的 custom 陣列：
```
"foo" => "Your input was invalid!",

"accepted" => "The :attribute must be accepted.",

// The rest of the validation error messages...
```

當建立了自訂的驗證規則，有時也會需要客製錯誤訊息的佔位符，可以使用上述方式建立自訂的驗證器後，呼叫<br/>
Validator  facade 的 replacer 方法，並且請在服務提供者中的 boot 方法來做這些事。
```
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```

#### 隱性擴充
預設情況下，如同 required 規則定義一般，當被驗證的屬性不存在或為空值時，則一般的驗證規則或包含自定的擴充，<br/>都不會被執行。以 unique 規則來說，當值為 null 時，該規則就不會被執行。
```
$rules = ['name' => 'unique'];

$input = ['name' => null];

Validator::make($input, $rules)->passes(); // return true
```

要當屬性為空時依然執行該規則，那麼該規則必須暗示該屬性為必填。要建立這樣的一個「隱式」擴充功能，可使用<br/>
Validator::extendImplicit 方法。
```
Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});
```