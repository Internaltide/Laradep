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
上述關聯中，各模型的關聯鍵名皆有預設慣例。<br/>
phone模型外鍵名稱，慣例以上層模型名稱附加**_id**，所以前例外鍵名稱為user_id。<br/>
user模型則慣例使用id這樣的關聯鍵名。<br/>
所以，上層模型user就用欄位id與下層模型phone的外鍵user_id進行join。<br/>
要打破慣例就必須依下列方式進行調整<br/>
 * 傳遞外鍵名到hasOne的第二個參數，覆寫預設的外鍵名稱
 * 傳遞上層關聯鍵名到hasOne的第三個參數，覆寫預設的上層關聯鍵名id
```
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```
#### 反向關聯
依前面定義方式，我們可以從user取得phone資料，其屬於正向關聯。<br/>
若要反過來從phone取得user資料，就要定義好反向關聯。<br/>
在phone模型(被擁有者)中定義關聯方法(名稱無限制)；接著在關聯方法內調用belongsTo方法並返回結果，<br/>
這樣就可以為phone模型產生動態屬性，往後再藉由模型動態屬性取得關聯資料。<br/><br/>
另外，反向關聯與正向關聯的慣例差異，在於反向關聯的外鍵是以下層模型的關聯方法附加**_id**，<br/>
上層關聯鍵名則一樣使用id。
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

#### 示意表 ( **Users <=> Phone** )
&nbsp;                    | 正向關聯(user has phone)                    | 反向關聯(phone belongs to user)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型users]     | if R exist use R else use id                    | if R exist use R else use id
&nbsp;                    |                                                                |
[關聯方法]             | phone()                                                   | user()
[調用方法]             | hasOne('App\Phone','F','R')                   | belongsTo('App\Users','F','R')
[調用位置]             | 上層模型                                               | 下層模型
&nbsp;                    |                                                                |
[下層模型phone]   | if F exist use F else use user**s**_id    | if F exist use F else use user_id

### 一對多 ( **Posts <=> Comment** )
#### 示意表
&nbsp;                    | 正向關聯(post has comments)                    | 反向關聯(comment belongs to post)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型posts]          | if R exist use R else use id                    | if R exist use R else use id
&nbsp;                         |                                                                |
[關聯方法]                  | comments()                                            | post()
[調用方法]                  | hasMany('App\Comment','F','R')           | belongsTo('App\Posts','F','R')
[調用位置]                  | 上層模型                                               | 下層模型
&nbsp;                         |                                                                |
[下層模型comment]   | if F exist use F else use post**s**_id    | if F exist use F else use post_id
#### 取得資料
```
$comments = App\Post::find(1)->comments;

$comment = App\Post::find(1)->comments()->where('title', 'foo')->first();

$comment = App\Comment::find(1);
echo $comment->post->title;
```

### 多對多 ( **Users <= Role_User => Roles** )
#### 示意表
&nbsp;                    | 關聯一(user has roles)                    | 關聯二(role has users)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[模型一users]         | if U exist use U else use ***                  | if U exist use U else use ***
&nbsp;                    |                                                                |
[關聯方法]             | roles()                                                     | users()
[調用方法]             | belongsToMany('App\Roles','user_role','U','R')           | belongsToMany('App\Users','user_role','R','U')
[調用位置]             | 模型一                                                   | 模型二
&nbsp;                    |                                                                |
[模型二roles]         | if R exist use R else use ***                  | if R exist use R else use ***
[中介樞紐表]         | if user_role exist use user_role else role_user | if user_role exist use user_role else role_user
 * 中介樞紐表名稱會按字母順序合併兩個模型名稱做為預設值，所以預設表明會是roles_users
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
 我們在取出隸屬多對多的模型時，物件會自動被賦予名為pivot的樞紐屬性。<br/>
 其代表了中介表的模型，我們可以像操作eloquent模型一樣操作它。<br/>
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
#### 示意表 ( **Country => Users => Article** )
&nbsp;                    | 正向關聯(country has articles)                    | 反向關聯(article belongs to country)
:-----------------------|:----------------------------------------------:|:--------------------------------------:
[上層模型country] | if L1 exist use L1 else use ***              |
&nbsp;                    |                                                                |
[關聯方法]             | articles()                                                 |
[調用方法]             | hasManyThrough('App\Article','App\Users','F1','F2','L1','L2')|
[調用位置]             | 上層模型                                               |
&nbsp;                    |                                                                |
[目標模型article]   | if F2 exist use F2 else use ***                  |
[中介模型users]     | Fri: if F1 exist use F1 else use ***<br/>Pri: if L2 exist use L2 else use ***|
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
#### 示意表
#### 取得資料

## 查詢關聯
## 預先載入
## 插入、新增之關聯模型
## 上層時間戳記連動