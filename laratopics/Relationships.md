# Eloquent ORM Relationship

## 定義模型關聯
### 一對一
此為最基本的關聯，以user、phone兩個模型為例，為反映user擁有一支phone這樣的關係，<br/>
我們須在user模型(擁有者)中定義關聯方法(名稱無限制)；接著在關聯方法內調用hasOne方法並返回結果，<br/>
這樣就可以為user模型產生動態屬性，往後就可藉由模型動態屬性取得關聯資料。

#### 定義關聯方法；產生動態屬性
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 獲得與用戶關聯的電話紀錄。
     */
    public function phone()
    {
        return $this->hasOne('App\Phone');
    }
}
```
#### 透過動態屬性取得關聯資料
```
$phone = User::find(1)->phone;
```
#### 預設慣例
關聯模型用的鍵名皆有預設的名稱慣例。而為了方便，我們將外鍵所在模型稱之為下層模型<br/>
，另一個稱之為上層模型，以方便解釋。<br/>

依照慣例，下層模型的外鍵名稱會假定是上層模型名稱附加**_id**，因此前例 phone 模型的外鍵名稱<br/>
應該為 user_id。而上層模型的關聯用鍵則預設假定為 id。換句話說，預設認定上層模型user會使用欄位id<br/>
來與下層模型phone的外鍵user_id進行join。若要打破鍵名的慣例就須依下列方式進行調整<br/>
 * 傳遞下層模型的外鍵名到hasOne的第二個參數，覆寫預設的外鍵名稱
 * 傳遞上層模型的關聯鍵名到hasOne的第三個參數，覆寫預設的上層關聯鍵名id
```
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```
#### 反向關聯
依前面定義方式，我們可以從 user 取得 phone 資料，其屬於正向關聯。<br/>
若要反過來從下層模型 phone 去取得 上層模型的 user 資料，就要定義好反向關聯。<br/>
在 phone 模型(被擁有者)中定義關聯方法(名稱無限制)；接著在關聯方法內調用belongsTo方法並返回結果，<br/>
這樣就可以為phone模型產生動態屬性，往後再藉由模型動態屬性取得關聯資料。<br/>
另外，反向關聯與正向關聯的慣例差異，主要在於反向關聯時，下層模型的外鍵名稱則會假定為是關聯方法的<br/>
名稱再附加上**_id**，上層關聯鍵名則一樣使用id。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```
覆寫反向關聯慣例
```
return $this->belongsTo('App\User', 'foreign_key', 'other_key');
```
預設模型<br/>
belongs允許設定預設模型，即當返回結果為空時，即回傳設定的預設模型。<br/>
此功能可省去開發者額外撰寫判斷程式的時間。
```
// 若結果為空，返回App\User的空模型
return $this->belongsTo('App\User')->withDefault();

// 若結果為空，返回設定了name屬性的App\User模型
return $this->belongsTo('App\User')->withDefault([
    'name' => 'Guest Author',
]);

// 若結果為空，返回設定了name屬性的App\User模型
return $this->belongsTo('App\User')->withDefault(function ($user) {
    $user->name = 'Guest Author';
});
```

#### 示意表 ( **Users ⇦⇨ Phone** )
&nbsp;                    | 正向關聯(user has phone)                    | 反向關聯(phone belongs to user)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型users]     | 主鍵名稱為R(默認是id)                      | 主鍵名稱為R(默認是id)
&nbsp;                    |                                                                |
[關聯方法]             | phone()                                                   | user()
[調用方法]             | hasOne('App\Phone','F','R')                   | belongsTo('App\Users','F','R')
[調用位置]             | 上層模型                                               | 下層模型
&nbsp;                    |                                                                |
[下層模型phone]   | 主鍵名稱為F(默認是users_id)             | 主鍵名稱為F(默認是user_id)

### 一對多 ( **Posts ⇦⇨ Comment** )
#### 示意表
&nbsp;                    | 正向關聯(post has comments)                    | 反向關聯(comment belongs to post)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型posts]          | 主鍵名稱為R(默認是id)                 | 主鍵名稱為R(默認是id)
&nbsp;                         |                                                           |
[關聯方法]                  | comments()                                       | post()
[調用方法]                  | hasMany('App\Comment','F','R')      | belongsTo('App\Posts','F','R')
[調用位置]                  | 上層模型                                          | 下層模型
&nbsp;                         |                                                           |
[下層模型comment]   | 主鍵名稱為F(默認是posts_id)        | 主鍵名稱為F(默認是post_id)
#### 取得資料
```
$comments = App\Post::find(1)->comments;

$comment = App\Post::find(1)->comments()->where('title', 'foo')->first();

$comment = App\Comment::find(1);
echo $comment->post->title;
```

### 多對多 ( **Users ⇦ Role_User ⇨ Roles** )
#### 示意表
&nbsp;                    | 關聯一(user has roles)                    | 關聯二(role has users)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[模型一users]         | 樞紐表對應外鍵名稱為U(默認是*)   | 樞紐表對應外鍵名稱為U(默認是*)
&nbsp;                    |                                                                |
[關聯方法]             | roles()                                                     | users()
[調用方法]             | belongsToMany('App\Roles','user_role','U','R')           | belongsToMany('App\Users','user_role','R','U')
[調用位置]             | 模型一                                                   | 模型二
&nbsp;                    |                                                                |
[模型二roles]         | 樞紐表對應外鍵名稱為R(默認是*)    | 樞紐表對應外鍵名稱為R(默認是*)
[中介樞紐表]         | 樞紐表名稱為user_role(默認是roles_users) | 樞紐表名稱為user_role(默認是roles_users)
 * 中介樞紐表名稱會按字母順序合併兩個模型名稱做為預設值，所以預設表名會是roles_users
 * belongsToMany方法參數說明如下
   * 第一參數 - 欲連結的目標模型。
   * 第二參數 - 連接中介樞紐表名稱，用以覆寫欲設值。
   * 第三參數 - 關聯方法所處模型在樞紐表上對應的鍵名，用以覆寫預設值。
   * 第四參數 - 欲連結的目標模型在樞紐表上對應的鍵名，用以覆寫預設值
 * 各模型的預設鍵名並不清楚，官方文件未說明，待釐清。
 #### 取得資料
 ```
$roles = App\User::find(1)->roles()->orderBy('name')->get();

$user = App\User::find(1);
foreach ($user->roles as $role) {
    //
}
 ```
 #### 取得中介樞紐表資料
我們在取出隸屬多對多的模型時，物件會自動被賦予名為 pivot 的樞紐屬性。<br/>
其屬性代表了中介表的模型，我們可以像操作 eloquent 模型一樣操作它。<br/>
但預設該樞紐物件只會提供模型的鍵，若需要包含中介表的其他屬性，則需要<br/>
自行在設定關聯時進行指定需要的欄位
```
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
```
#### 讓中介樞紐表自動維護created_at跟updated_at倆的時間戳記
```
return $this->belongsToMany('App\Role')->withTimestamps();
```
#### 自訂樞紐屬性名稱Via as
```
// change piovt attribute name
return $this->belongsToMany('App\Podcast')
                ->as('subscription')
                ->withTimestamps();

// and then we can using subscription instead of piovt
$users = User::with('podcasts')->get();
foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```
#### 使用樞紐模型資料進行過濾
method: **wherePivot**、**wherePivotIn**
```
return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);
```
#### 自訂樞紐模型
預設樞紐模型是會自動建立的，若要自行定義則需使用**using**方法載入。<br/>
自定義的模型則需繼承自Illuminate\Database\Eloquent\Relations\Pivot類別<br/>

自定義樞紐模型
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Relations\Pivot;

class UserRole extends Pivot
{
    //
}
```
載入自定樞紐模型
```
return $this->belongsToMany('App\User')->using('App\UserRole');
```

### 遠程一對多
當希望取得特定國別出處的文章時，但article table卻可能沒有對應country table的外鍵。所以，<br/>
我們須透過中介的user table先取得特定國別下的User IDs，在利用他們到article table查詢。<br/>
這種情況在Eloquent中就是使用遠程一對多的關聯設定來實現。
#### 示意表 ( **Country ⇨ Users ⇨ Article** )
&nbsp;                    | 正向關聯(country has articles)                    | 反向關聯(article belongs to country)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型country] | 主鍵名稱為 L1                                     |
&nbsp;                    |                                                                |
[關聯方法]             | articles()                                                 |
[調用方法]             | hasManyThrough('App\Article','App\Users','F1','F2','L1','L2')|
[調用位置]             | 上層模型                                               |
&nbsp;                    |                                                                |
[目標模型article]   | 連接中介模型的外鍵名稱為 F2                  |
[中介模型users]     | 連接上層模型的外鍵名稱為 F1<br/>主鍵名稱為 L2 |
* 官方隻字未提反向的關聯設定方式，似乎未支援? 待釐清
* hasManyThrough方法參數說明如下
   * 第一參數 - 欲查詢的最終目標模型。
   * 第二參數 - 中介資料表名稱。
   * 第三參數 - 中介模型連結上層模型的外鍵名。
   * 第四參數 - 目標模型連結中介模型的外鍵名。
   * 第五參數 - 上層模型連結中介模型的鍵名，通常為其主鍵。
   * 第六參數 - 中介模型連結目標模型的鍵名，通常為其主鍵。
 * 各模型的預設鍵名並不清楚，官方文件未說明，待釐清。
 #### 取得資料
 ```
$articles = App\Country::find(1)->articles;
 ```

### 多型關聯
**posts**<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**\\\\**<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**\\\\ type:posts**<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**comment**<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**// type:videos**<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**//**<br/>
**videos**<br/>
多型關聯意指模型可藉單一關聯從屬到多個上層模型。以posts、videos、comment三張表為例，就像是<br/>
單一comment資料表可藉commentable_id、commentable_type兩個欄位，同時滿足posts跟videos兩表的<br/>
留言數據存放。commentable_id存放的是post或video的資料ID，commentable_type則存放所屬上層模型的類別名稱。<br/>
其中，commentable_type也是供ORM判斷單一comment數據是關聯到那個上層模型
#### 示意表
&nbsp;                | posts has comment  | videos has comment | comment belongs to posts/video
:-----------------------|:------------------------:|:---------------------------------:|:--------------------:
[模型一posts]         |                                  |                                              |
&nbsp;                    |                                  |
[關聯方法]             | comments()              | comments()                          | commentable()
[調用方法] | morphMany('App\Comment', 'commentable') | morphMany('App\Comment', 'commentable') | morphTo()
[調用位置]             | 模型一                     | 模型二                                 | 下層模型
&nbsp;                    |                                  |                                              |
[模型二videos]       | &nbsp;                     | &nbsp;                                 | &nbsp;
[下層模型comment] |                                |                                             |
#### 取得資料
```
// Retrieve Post's Comment
$post = App\Post::find(1);
foreach ($post->comments as $comment) {
    //
}

// Retrieve Comment's Owner
$comment = App\Comment::find(1);
$commentable = $comment->commentable;
```
#### 多型關聯解偶
預設，Laravel會限制模型需使用類別名做為辨識關聯的上層模型。如同前面的例子<br/>
comment屬於post或video，所以模型內commentable_type的值只會是App\Posts或App\Videos。<br/>
但如此一來就會導致資料庫結構與程式內部產生耦合，若不希望產生無謂的耦合，可以使用<br/>
morphMap方法註冊多型映射表，指示Eloquent使用自訂名稱而不是類別名稱。<br/>
多型映射表的註冊需在服務提供者中的boot方法內進行實作，可以選擇AppServiceProvider或其他獨立的<br/>
服務提供者。
```
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'posts' => 'App\Post',
    'videos' => 'App\Video',
]);
```

### 多型多對多關聯
posts ⇦⇦ taggables ⇨⇨ tags<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;⇩<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;⇩<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;videos<br/>
若把多型關聯看做是多個一對多關聯共用同一個下層模型的結構，則多對多多型關聯就是多個<br/>
多對多關聯共用同樣的下層模型與中介樞紐模型的結構體。<br/>
範例結構如下：
```
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer        // 存放post或video的ID
    taggable_type - string      // 存放所屬上層模型之類別名稱
```
#### 示意表
&nbsp;                    | posts has tag       | videos has tag     | tag has posts     | tag has video
:-----------------------|:--------------------:|:--------------------:|:------------------:|:----------------:
[模型一posts]         |                            |                             |                           |
&nbsp;                    |                            |                             |                           |
[關聯方法]             | tags()                  | tags()                   | posts()               | videos()
[調用方法] | morphMany('App\Tag', 'taggable') | morphMany('App\Tag', 'taggable') | morphedByMany('App\Post', 'taggable'); | morphedByMany('App\Video', 'taggable');
[調用位置]             | 模型一               | 模型二                | 下層模型          | 下層模型
&nbsp;                    |                            |                             |                           |
[模型二videos]       | &nbsp;               | &nbsp;                | &nbsp;              | &nbsp;
[下層模型tag]         |                           |                             |                           |
[中介樞紐模型]      |                           |                             |                           |

#### 取得資料
```
$post = App\Post::find(1);
foreach ($post->tags as $tag) {
    //
}

$tag = App\Tag::find(1);
foreach ($tag->videos as $video) {
    //
}
foreach ($tag->posts as $post) {
    //
}
```

## 查詢關聯
當您透過關聯方法設定了所有資料表模型關聯後，即可不實際運行關聯查詢來取的關聯資料。<br/>
除了當作動態屬性直接存取外，亦可視為方法調用，並當作查詢建構器實例繼續鍊式串接條件約束<br/>

### 取得存在特定關聯的數據
method: **has**
```
// 取得有留言的Post
$posts = App\Post::has('comments')->get();

// 取得三則留言以上的Post
$posts = App\Post::has('comments', '>=', 3)->get();

// 取得至少含一則被按讚留言的Post
$posts = App\Post::has('comments.votes')->get();
```
method: **whereHas、orWhereHas**
```
// 取得留言內容符合foo%條件的Post
$posts = App\Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

### 取得不存在特定關聯的數據
method: **doesntHave、orDoesntHave**
```
// 取得不含留言的Post
$posts = App\Post::doesntHave('comments')->get();

```
method: **whereDoesntHave、orWhereDoesntHave**
```
// 取得留言內容不符合foo%條件的Post
$posts = App\Post::whereDoesntHave('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 取得不含留言作者被禁的Post
$posts = App\Post::whereDoesntHave('comments.author', function ($query) {
    $query->where('banned', 1);
})->get();
```

### 模型數據統計
method: **withCount**<br/>
增加統計計數但不載入實際數據，取用統計數字時使用符合{relation}_count格式的欄位名稱
```
// 除Post資料外，額外取得留言統計數字
$posts = App\Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```
一次取得多個統計計數，並為其加上約束條件
```
// 除Post資料外，額外取得按讚次數與留言內容符合foo%條件的計數
$posts = App\Post::withCount(['votes', 'comments' => function ($query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```
為關聯統計數據新增別名
```
$posts = App\Post::withCount([
    'comments',
    'comments as pending_comments_count' => function ($query) {
        $query->where('approved', false);
    }
])->get();

echo $posts[0]->comments_count;

echo $posts[0]->pending_comments_count;
```
PS.如上例， 同一個關聯可以定義多個不同原則的統計方式

## 預先加載
預設的關聯資料取得都是使用Lazy Loading，意即在存取其特性時才會開始查詢並加載數據。<br/>
這在下面的例子中會造成N+1次的查詢，可能有效能上的問題，可以使用預先加載來解決。
```
<?php
// 關聯方法
public function author()
{
    return $this->belongsTo('App\Author');
}

// 關聯數據的查詢會在迴圈中的每一次動態特性autho存取r時都發生一次
$books = App\Book::all();          // 一次查詢
foreach ($books as $book) {       // 開始N次查詢，共產生N+1次查詢
    echo $book->author->name;
}
```
### 使用**with** method在第一條查詢就預先加載author的關聯資料，避免N+1查詢
```
$books = App\Book::with('author')->get();
foreach ($books as $book) {
    echo $book->author->name;
}

// 加了with('author')的書籍資料查詢，等同於多了一條
// select * from authors where id in (1, 2, 3, 4, 5, ...)
// 而迴圈內的資料存取就變成是單存的記憶體資料操作

// with方法亦可以同時加載多個關聯資料
$books = App\Book::with(['author', 'publisher'])->get();

// 加載嵌套的關聯資料，預載作者資料及其連絡資料
$books = App\Book::with('author.contacts')->get();
```
PS. 官方說在query parent model時可以使用with，意味著反相關聯不能使用? 待確認

### 只加載特定欄位的關聯資料
```
$users = App\Book::with('author:id,name')->get();
```
PS. 官方提及id column 總是應該被加入清單中，不太確定含意，待確認
### 預先加在有額外約束條件的關聯資料
```
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
```
### 延遲的預先加載
當需要父模型資料來判斷是否是否加載關聯資料時，<br/>
則可以使用此方法，將關聯資料預載延後到存取完父模型資料後才開始
```
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}

// 添加約束的延後預載
$books->load(['author' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```
### 僅在未加載時才加載
避免直接全域加載，只在特定狀況下才載入關聯資料
```
public function format(Book $book)
{
    $book->loadMissing('author');

    return [
        'name' => $book->name,
        'author' => $book->author->name
    ];
}
```

## 關聯模型的插入與更新
### 插入新的關聯模型
method: **save**
```
$comment = new App\Comment(['message' => '一條新的評論。']);

$post = App\Post::find(1);

// 在關聯模型的操作方式下，外鍵post_id會自動添加，無須手動指定
$post->comments()->save($comment);

// 如果處理的是多對多關聯，還可以傳遞第二個參數插入額外資料到中間樞紐表
App\User::find(1)->roles()->save($role, ['expires' => $expires]);
```
### 插入多個新的關聯模型
method: **saveMany**
```
$post = App\Post::find(1);

$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);
```
### 插入新的關聯模型(Mass Assignment)
method: **create**<br/>
功能等價於save，唯一差別在於其接受的參數為PHP陣列，而save則須是模型實例
```
$post = App\Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```
### 插入多個新的關聯模型(Mass Assignment)
method: **createMany**<br/>
```
$post = App\Post::find(1);

$post->comments()->createMany([
    [
        'message' => 'A new comment.',
    ],
    [
        'message' => 'Another new comment.',
    ],
]);
```
### 附加belongsTo關聯
method: **associate**
此法會自動在子模型中設定好外鍵
```
// 將ID為10的帳戶關連到$user(Let Account belongs to $user)
$account = App\Account::find(10);
$user->account()->associate($account);
$user->save();
```
### 取消belongsTo關聯
method: **dissociate**
子模型外鍵會被設成null
```
$user->account()->dissociate();
$user->save();
```
### 多對多關聯的附加
method: **attach**
```
$user = App\User::find(1);
$user->roles()->attach($roleId);

// Attach three roles to user...
$user->roles()->attach([1, 2, 3]);

// 亦可透過傳遞一陣列參數來寫入額外數據
$user->roles()->attach($roleId, ['expires' => $expires]);
$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires]
]);
```
### 多對多關聯的移除
```
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach three role from user...
$user->roles()->detach([1, 2, 3]);

// Detach all roles from the user...
$user->roles()->detach();
```
### 同步關聯
method: **sync、syncWithoutDetaching**
接受一包含多個ID的陣列參數
```
// 建立ID為1、2、3的關聯，其餘皆移除
$user->roles()->sync([1, 2, 3]);

// 同上，另外對ID為1的數據進行expires資料更新
$user->roles()->sync([1 => ['expires' => true], 2, 3]);

// 建立ID為1、2、3的關聯，但不做Detache的動作
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

### 關聯切換
method: **toggle**
關聯關係的反向操作。原attached改為detach；原detached改為attach
```
$user->roles()->toggle([1, 2, 3]);
```

### 更新中間樞紐表的資料
method: **updateEistingPivot**
```
// 透過外鍵$roleId找到對應記錄後，依指定的第二參數進行資料更新
$user = App\User::find(1);
$user->roles()->updateExistingPivot($roleId, $attributes);
```

## 上層時間戳記連動
當子模型belongsTo或belongsToMany父模型時，又具子模型數據更新須連動更新所屬父模型時間戳記的需求時，<br/>
可以利用模型的屬性**touches**進行設定。
```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 設定要連動的上層模型
     * 所有設定的關聯都會被連動
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * Get the post that the comment belongs to.
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}

//現在，當你更新一個 Comment，它所屬的 Post 的 updated_at 欄位也會同時更新
$comment = App\Comment::find(1);
$comment->text = 'Edit to this comment!';
$comment->save();
```