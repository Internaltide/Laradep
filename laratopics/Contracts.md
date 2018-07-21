# Laravel Contracts

> Laravel 的 Contracts 即一組定義了框架核心服務的介面(Interface)。
> 相對於Facades來說 ，Facade是提供一個簡單易用的方法來使用服務，
> 它並不像Contract一樣需要明確透過型別提示來達成依賴的注入。
>
> 而大部分情況下，Facade都有對應等價的Contract，兩者並沒有太大
> 差異。選擇上，一個提供便利性，一個明確定義相依性，端看喜好而定。

## 透過介面實作的原因
 * 低耦合<br/>
    透過綁定介面而非特定套件，可以降地偶合度。屆時要抽換套件會比較輕鬆。
 * 簡單性<br/>
    當所有的 Laravel 服務都簡潔的使用簡單的介面定義，就能夠很簡單的決定<br/>
    一個服務需要提供的功能。還能將 contracts 視為說明框架特色的簡潔文件。

PS. Kentbeck實作模式理其實有提過不要輕易使用介面，因為修改成本可能會很大。<br/>
      作者是認為修改原因在於不穩定，但對於常用的服務(如郵件寄送、佇列處理)來說，<br/>
      該有什麼樣的方法，應該都趨於穩定了。所以，該用上介面的還是要用，因為確實可提升維護性。<br/>
      唯一或許要考量的是領域層內的服務介面，若還有很高的異動性，則在介面的使用上就要多加留意。

## 使用Contracts
在建構式使用型別提示時，直接使用Contracts的介面名稱即可注入介面的實作實例。
舉例：
```
<?php

namespace App\Listeners;

use App\User;
use App\Events\OrderWasPlaced;
use Illuminate\Contracts\Redis\Database;

class CacheOrderInformation
{
    /**
     * The Redis database implementation.
     */
    protected $redis;

    public function __construct(Database $redis)
    {
        $this->redis = $redis;
    }

    public function handle(OrderWasPlaced $event)
    {
        //
    }
}
```
