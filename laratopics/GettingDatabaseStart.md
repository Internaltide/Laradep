# Database: Getting Started
> Laravel可以使用原生SQL、查詢建構器跟Eloquent ORM來與資料庫進行互動，<br/>
> 並支持MySQL、MSSQL、SQLite、PostgreSQL。<br/>
> 主要設定檔案位於 config/database.php，每個類型的資料庫皆提供預設設定，<br/>
> 可以此為模板新增額外的連線設定，幾個地方特別注意即可<br/>
>* charset跟collation要設定正確<br/>
>* driver名稱不要打錯，各類型資料庫之driver名稱為sqlite、mysql、pgsql、sqlsrv

## 讀寫分離設定
在主設定陣列中，使用read、write兩個Key各自代表讀、寫兩種設定。<br/>
各設定子陣列基本至少含host設定指向對應的主機IP，剩餘未指定項目則同主陣列設定值，<br/>
換句話說，即讀、寫兩區塊會共享主陣列的設定值，並以區塊內的設定值作覆寫。<br/>
因此，若要覆寫共享設定則需要複製一份主陣列配置項進子陣列內做修改。

#### read、write
```
'mysql' => [
    // 讀取設定子陣列
    'read' => [
        'host' => ['192.168.1.1'],
        'password' => '123456', // 獨立的存取密碼
        // ...其於設定項目同主陣列設定值
    ],
    // 寫入設定子陣列
    'write' => [
        'host' => ['196.168.1.2'],
        // ...其於設定項目同主陣列設定值，如密碼為空
    ],
    'sticky'    => true,
    'driver'    => 'mysql',
    'database'  => 'database',
    'username'  => 'root',
    'password'  => '',
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
],
```
#### sticky(Optional)
於讀寫分離下，藉由設定sticky為真，可確保在同一個請求週期內，讀取數據皆以該請求內寫入的數據為主。<br/>
亦即，在同一個請求週期內，執行過資料寫入後的資料讀取皆強制採用寫入的連線來取得資料。

## 多資料庫連線
在設定中僅會有一組預設連線，但仍可以設定多個連線設定。<br/>
有同時使用多個連線需求時，可使用DB Facade的connection方法指定連線。
```
$users = DB::connection('連線名稱')->select(...);
```
## 資料查詢
### 原生SQL
Facade：**DB**<br/>
相關方法：**select**、**update**、**insert**、**delete**、**statement**<br/>
#### Select Query
```
$users = DB::select('select * from users where active = ?', [1]);

// 回傳結果為內含PHP Stand Object的陣列，可用foreach進行遍歷
foreach ($users as $user) {
    echo $user->name;
}
```
例中的綁定亦可改用名稱綁定的方式
```
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

#### Insert Statement
```
// 會回傳受影響資料筆數
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```
#### Delete Statement
```
// 會回傳受影響資料筆數
$deleted = DB::delete('delete from users');
```
#### General Statement
```
DB::statement('drop table users');
```
### [查詢建構器](https://github.com/Internaltide/Laradep/blob/master/laratopics/QueryBuilder.md)
### [Eloquent ORM](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingORMStart.md)

## 監聽SQL查詢
如果想要監控應用程式的每一個SQL查詢，可以在服務提供者下使用listen方法進行監聽事件註冊
```
public function boot()
{
    DB::listen(function ($query) {
        // $query->sql
        // $query->bindings
        // $query->time
    });
}
```
## 資料庫交易 via DB Facade
#### Use transaction method
使用DB Facade的transatciont執行一系列的SQL操作，若發生任何例外，則系統會自行rollback；反之則自動提交。<br/>
開發者並無須手動加上回滾或提交的程式碼，且該方法亦可控制建構器與Eloquent ORM的資料庫交易
```
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});

// 傳入第二參數控制重試次數以避免deadLock。一旦嘗試次試用盡，直接回傳一個例外
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
}, 5);
```

#### Manually Control Transactions
```
DB::beginTransaction();
DB::rollBack();
DB::commit();
```