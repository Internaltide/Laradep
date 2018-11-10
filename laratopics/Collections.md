# Laravel 集合

## 簡介
Illuminate\Support\Collection 提供了一個更流暢、更方便處理陣列數據的封裝。例如，我們可以使用collect<br/>
這個輔助函數根據特定的陣列數據產生一個集合物件。接著，對每個元素使用strtoupper後再移除掉空元素。
```
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});
```
直接對集合物件進行上述的操作，比起直接處理陣列數據結構，可以具有更效率、更優美的寫法。正如你所見，集合<br/>
類別允許你鏈式調用各種方法，以達到對底層陣列數據結構進行map與reject的操作。而一般來說，集合物件並無法改<br/>
變，因此大多數的集合方法都會返回一個全新的集合實例。

## 創建集合
如同前面所述，輔助方法collect會返回一個特定陣列數據結構的Illuminate\Support\Collection實例。所以，創建一個<br/>
集合物件是非常簡單的。
```
$collection = collect([1, 2, 3]);
```
PS. 預設，Eloquent模型的查詢結果都會返回集合實例。

## 擴展集合
集合是 macroable 的，因此你可以在執行期為集合物件增加新的方法。以下面為例，為一個集合增加一個toUpper的方法
```
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```
PS. 一般來說，你應該在服務提供者中來宣告集合的方法擴充。

## 可用方法
|                |               |             |           |            |            |
| ---------- |:---------- |:-------- |:------- |:------- |:-------- |
| all | average | avg | chunk | collapse | combine |
| concat | contains | containsStrict | count | crossJoin | dd |
| diff | diffAssoc | diffKeys | dump | each | eachSpread |
| every | except | filter | first | firstWhere | flatMap |
| flatten | flip | forget | forPage | get | groupBy |
| has | implode | intersect | intersectByKeys | isEmpty | isNotEmpty |
| keyBy | keys | last | macro | make | map |
| mapInto | mapSpread | mapToGroups | mapWithKeys | max | median |
| merge | min | mode | nth | only | pad |
| partition | pipe | pluck | pop | prepend | pull |
| push | put | random | reduce | reject | reverse |
| search | shift | shuffle | slice | sort | sortBy |
| sortByDesc | sortKeys | sortKeysDesc | splice | split | sum |
| take | tap | times | toArray | toJson | transform |
| union | unique | uniqueStrict | unless | unwrap | values |
| when | where | whereStrict | whereIn | whereInStrict | whereInstanceOf |
| whereNotIn | whereNotInStrict | wrap | zip | | |
