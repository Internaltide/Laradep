# Laravel Facades

> Laravel Facade是一個靜態介面，為服務容器中的底層類別提供靜態代理<br/>
> 簡潔且更輕鬆的使用Laravel核心功能。
>
> 所有的Facades都定義在Illuminate\Support\Facades這個命名空間中，<br/>
> 以便我們簡單取出Facade來使用。

## 何時使用
### 注意事項
因為Facade太容易使用，不經意地就可能造成類別職責變得太大且不自覺，<br/>
它不像依賴注入一樣，可藉由建構式的逐漸肥大而有所感覺，所以使用Facade時需要<br/>
特別注意職責範疇的問題。
### Facade Test
傳統靜態類別方法無法使用mock或stub來進行測試，但因為Laravel Facade用了特殊的動態方法
對容器內物件進行代理，故仍可像依賴注入那樣測試Facade
例如以下面的路由為例
```
Route::get('/cache', function () {
    return Cache::get('key');
});
```
撰寫測試驗證Cache::get的正確性
```
use Illuminate\Support\Facades\Cache;

/**
 * 基本的函式測試範例
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

### 與Facade等價的輔助方法
很多輔助方法提供了和 Facade 一樣的功能，而Facade和其等價的輔助方法之間並無實質差異，<br/>
因為輔助方法底層仍是呼叫Facade的方法。<br/>
下面這個 Facade 和輔助方法即為等價的
```
return View::make('profile');

return view('profile');
```

## Facade工作原理
Facade就是一個提供存取容器內物件的類別。無論框架自帶或自訂的Facade<br/>
都必須繼承自基底類別Illuminate\Support\Facades\Facade。<br/>
而這個基底類別也是促使所有Facade得以正常運作的關鍵，因為<br/>
該基底類別使用了__callStatic()這個魔術方法，讓呼叫看起來像是在Facade內部發生，<br/>
但實則是調用了容器內部已解析物件方法。<br/>
另外，每一個Facade都必須實作getFacadeAccessor()，用以回傳服務容器綁定的名稱。
