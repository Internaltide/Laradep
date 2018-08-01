# Laravel Query Builder

> Laravel所提供的一個方便且流暢的資料庫操作介面，在所有Laravel支援的資料庫下皆可正常運作，<br/>
> 相較於原生SQL來說，建構器的跨資料庫能力相對更高。<br/>
> 底層還會自動使用PDO的參數綁定以避免SQL注入，所以手動過濾字串的動作可以被忽略。

## Retrieving Results

### 取得資料表所有結果
 * 使用DB table方法取得特定表的建構器實例
 * 使用建構器的get方法取得內含PHP Stand Object的資料集合
```
users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```
### 取得一筆資料(資料行)
```
// 回傳值直接為PHP標準物件
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
```
### 取得特定欄位的資料
```
$email = DB::table('users')->where('name', 'John')->value('email');
```
### 取得一列資料(資料列)
```
$titles = DB::table('roles')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```
### 指定資料集合中欄位資料的鍵值
```
$roles = DB::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```
### 資料分塊
```
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    // 一次取得一小塊資料進行處理
    foreach ($users as $user) {
        // 隨時可依據狀況中斷繼續取用資料分塊
        return false
    }
});
```
PS. 尚未確認，不確定是直接結束還是只打斷該分塊的資料處理

### 聚合方法
count、sum、avg、min、max
#### count
```
$users = DB::table('users')->count();
```
#### min、max
```
$minPrice = DB::table('orders')->min('price');
$maxPrice = DB::table('orders')->max('price');
```
#### avg
```
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```

### 判斷資料是否存在
```
return DB::table('orders')->where('finalized', 1)->exists();

return DB::table('orders')->where('finalized', 1)->doesntExist();
```

## Select Clause
### 定義選用欄位
method: **select**
```
$users = DB::table('users')->select('name', 'email as user_email')->get();
```
### 返回不重覆結果
method: **distinct**
```
$users = DB::table('users')->distinct()->get();
```
### 增加選用欄位
method: **addSelect**
```
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

## Raw Expressions
採用原生SQL語法的部分注入，因此若非標準，可能會降低跨資料庫的能力，要注意。
### 原生字串
method: **raw**<br/>
該法不接受參數的Binding，所以會有SQL注入的風險，使用上要注意
```
$users = DB::table('users')
                ->select(DB::raw('count(*) as user_count, status'))
                ->where('status', '<>', 1)
                ->groupBy('status')
                ->get();
```
### 原生Select
method: **selectRaw**
```
// 可使用參數綁定
$orders = DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```
### 原生Where
method: **whereRaw、orWhereRaw**
```
$orders = DB::table('orders')
                ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                ->get();
```
### 原生Having
method: **havingRaw、orHavingRaw**
```
$orders = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();
```
### 原生Order by
method: **orderByRaw**
```
$orders = DB::table('orders')
                ->orderByRaw('updated_at - created_at DESC')
                ->get();
```

## Join Clause
### 內部連接
method: **join**<br/>
第一參數為表名；剩餘參數皆為連接約束條件(Ex. on user.id=contacts.user_id))
```
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```
### 左外部連接
method: **leftJoin**
```
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```
### 交叉連接
method: **crossJoin**
```
$users = DB::table('sizes')
            ->crossJoin('colours')
            ->get();
```
### 進階連接
method: **join、on**<br/>
使用join方法並傳遞Closure作為第二個參數，Closure接收JoinClause並可直接在其內部指定約束條件
```
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();

DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```
### 連接子查詢
method: **joinSub、leftJoinSub、rightJoinSub**
```
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();
```

## Unions
```
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```
Ps. unionAll也有相同效果

## Where
### Simple Where
```
$users = DB::table('users')->where('votes', '=', 100)->get();

// 若只是簡單判斷值是否"相等"，可忽略運算符
$users = DB::table('users')->where('votes', 100)->get();

// 其他還有
$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```
### Multi Where
```
// 傳遞條件陣列給where方法，作為多重條件(彼此間關係為and)
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();

// 使用orWhere進行串接(彼此間關係為or)
$users = DB::table('users')
                    ->where('status', '=', '1')
                    ->orWhere('subscribed', '<>', '1')
                    ->get();
```
### 其他常用Where
method: **whereBetweer、whereNotBetween**
```
$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();

$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
```
method: **whereIn、whereNotIn**
```
$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();

$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();
```
method: **whereNull、whereNotNull**
```
$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();

$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
```

### 時間Where
非常方便的方法，不用再像從前一樣做一堆時間字串的處理<br/>
mehtod: **whereDate**
```
$users = DB::table('users')
                ->whereDate('created_at', '2016-12-31')
                ->get();
```
mehtod: **whereMonth**
```
$users = DB::table('users')
                ->whereMonth('created_at', '12')
                ->get();
```
mehtod: **whereDay**
```
$users = DB::table('users')
                ->whereDay('created_at', '31')
                ->get();
```
mehtod: **whereYear**
```
$users = DB::table('users')
                ->whereYear('created_at', '2016')
                ->get();
```
mehtod: **whereTime**
```
$users = DB::table('users')
                ->whereTime('created_at', '=', '11:20:45')
                ->get();
```

### 欄位與欄位間的運算比較
method: **whereColumn**<br/>
```
// 比較是否相等
$users = DB::table('users')
                ->whereColumn('first_name', 'last_name')
                ->get();

// 判斷更新日期是否晚於建立日期
$users = DB::table('users')
                ->whereColumn('updated_at', '>', 'created_at')
                ->get();

// 直接傳入條件陣列，個條件間的關係為and
$users = DB::table('users')
                ->whereColumn([
                    ['first_name', '=', 'last_name'],
                    ['updated_at', '>', 'created_at']
                ])->get();
```

### 參數分組
原生SQL<br/>
select * from users where name = 'John' and **(votes > 100 or title = 'Admin')**<br/>
⇓<br/>
使用查詢建構器<br/>
藉由傳遞閉包給where，並於閉包內實作條件約束，達到如上面SQL粗體的分組效果
```
DB::table('users')
            ->where('name', '=', 'John')
            ->where(function ($query) {
                $query->where('votes', '>', 100)
                      ->orWhere('title', '=', 'Admin');
            })
            ->get();
```

### Where Exist
原生SQL
```
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```
⇓<br/>
使用查詢建構器
```
DB::table('users')
            ->whereExists(function ($query) {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
```

### Json Where
用來支援查詢JSON字串內的屬性值，並使用操作子 **->** 來取得Json欄位值<br/>
僅在原生資料庫支援JSON數據時才有效用
#### 使用where
```
$users = DB::table('users')
                ->where('options->language', 'en')
                ->get();

$users = DB::table('users')
                ->where('preferences->dining->meal', 'salad')
                ->get();
```
#### 使用whereJsonContains查詢Json內的陣列是否包含特定值
```
$users = DB::table('users')
                ->whereJsonContains('options->languages', 'en')
                ->get();

$users = DB::table('users')
                ->whereJsonContains('options->languages', ['en', 'de'])
                ->get();
```

## Order、Grouping、Limit ＆ Offset
### 資料排序
method: **orderBy**
```
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
```
### 日期排序<br/>
預設使用create_at做排序，若欲改用其他時間欄位進行排序，可將欄位名傳入方法中<br/>
method: **latest、oldest**
```
$user = DB::table('users')
                ->latest()
                ->first();
```
### 隨機排序
method: **inRandomOrder**
```
$randomUser = DB::table('users')
                ->inRandomOrder()
                ->first();
```
### groupBy / having
method: **groupBy、having**<br/>
groupBy接收一個或以上的參數進行分組
```
$users = DB::table('users')
                ->groupBy('first_name', 'status')
                ->having('account_id', '>', 100)
                ->get();
```
### skip / take
同offset / limit<br/>
method: **skip、take**
```
$users = DB::table('users')->skip(10)->take(5)->get();

// 與上面有相同效果
$users = DB::table('users')
                ->offset(10)
                ->limit(5)
                ->get();
```

## Conditional Clauses
在希望於特定條件、情境下才套用特定過濾條件的時候使用<br/>
method: **when**
```
// 取得請求資料role
$role = $request->input('role');

// 僅有在請求資料$role為真時，閉包內的條件設定才會生效
$users = DB::table('users')
                ->when($role, function ($query, $role) {
                    return $query->where('role_id', $role);
                })
                ->get();
```
when method亦接受閉包作為第三個參數，當第一個參數不為真時，<br/>
第二個閉包參數將不會執行，但改執行第三個閉包參數。<br/>
這個特徵常被應用在預設排序的功能上
```
$sortBy = null;

$users = DB::table('users')
                ->when($sortBy, function ($query, $sortBy) {
                    return $query->orderBy($sortBy);
                }, function ($query) {
                    return $query->orderBy('name');
                })
                ->get();
```

## Inserts
method: **insert**
```
// 一個子陣列代表一筆資料
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```
當資料表含有auto-increment id時，可改用下列方法於insert後回傳新資料id值<br/>
**(當底層資料庫為postgreSQL且欄位名稱不是id時，需額外將欄位名傳入insertGetId的第二參數)**<br/>
method: **insertGetId**
```
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

## Updates
method: **update**
```
// 一般欄位資料
DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);

// Json欄位資料
DB::table('users')
            ->where('id', 1)
            ->update(['options->enabled' => true]);
```
method: **increment、decrement**
更方便執行數值增減的更新方法
```
// votes +1
DB::table('users')->increment('votes');

// votes +5
DB::table('users')->increment('votes', 5);

// votes -1
DB::table('users')->decrement('votes');

// votes -5
DB::table('users')->decrement('votes', 5);

// 指定數值增減量時，以可同時指定其他要更新的欄位
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

## Deletes
method: **delete**
```
DB::table('users')->delete();

DB::table('users')->where('votes', '>', 100)->delete();
```
method: **truncate**<br/>
此法會刪除所有資料並重設auto-increment 欄位值為0
```
DB::table('users')->truncate();
```

## 悲觀鎖定(合併官方文件與網路說明，未經測試，目前理解有限)
用來處理數據的併發寫入，避免同時對同一條數據更新產生問題<br/>
以下兩種方法，通常擇一使用。
### 共享鎖定
method: **sharedLock**<br/>
```
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```
防止資料被更改的一種鎖定方式，需置放於交易內部才能生效<br/>
同時刻的其他DB操作不得更新被鎖定資料行，直到sharedLock所在交易被commit為止

### 更新鎖定
method: **lockForUpdate**<br/>
```
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```
防止資料被更改或查詢的一種鎖定方式，需置放於交易內部才能生效<br/>
同時刻的其他DB操作不得更新被鎖定資料行，直到lockForUpdate所在交易被commit為止<br/>
同時刻的其他交易內部無法再鎖定同樣資料行，直到lockForUpdate所在交易被commit為止<br/>

**以計數器計數更新為例，並假設同一時間存在兩個計數更新交易A、B ( 計數器初始數值為0 )**
> 兩個交易若使用sharedLock進行鎖定，結果可能是計數器變成1而非2。因為sharedLock<br/>
> 並不會鎖定讀取。所以同一時刻，兩邊的交易讀取到的當下計數數值都會是0，即便<br/>
> 實際上計數是更新了兩次，但依舊只增加了一次。<br/>
> ( A: 讀0加1得1  =>  B: 讀0加1得1  =>  最終計數為1 )
>
> 此時，可改用lockForUpdate來進行鎖定，則結果會變成符合預期的2。因為，當數據<br/>
> 一經lockForUpdate鎖定後，另一交易內的鎖定會被阻塞直到第一個被執行的交易提交後才開始。<br/>
> 所以第二個交易所讀到的數據一定是第一個交易計數更新後的結果，即增加一次<br/>
> 後的數據，接著，第二次交易再完成更新，計數就變成了2。<br/>
> ( A: 讀0加1得1  =>  B: 讀1加1得2  =>  最終計數為2 )

PS. 在Mysql上使用悲觀鎖時，須注意要先set autocommit=0，因為mysql預設為自動提交。

[Return Document Home](https://github.com/Internaltide/Laradep/blob/master/README.md)<br/>
[Return Database Getting Started](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingDatabaseStart.md#%E6%9F%A5%E8%A9%A2%E5%BB%BA%E6%A7%8B%E5%99%A8)