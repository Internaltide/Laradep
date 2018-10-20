# Laravel Error Handling

> ~~~
> 當你開始一個Laravel專案時，框架就已經幫您配置好錯誤即例外的處理方式，而
> App\Exceptions\Handler則是記錄應用程式所觸發例外及響應給使用者的地方。
> 文件後續將會有更深入的說明。
> ~~~

## 配置
config/app這個配置檔中的 **debug** 設定項目是用來控制錯誤訊息顯示的仔細程度。預設，<br/>
該設定項會在環境設定檔.env下以 **APP_DEBUG** 來存在。對本地端開發環境來說，你應該將<br/>
該選項設定為True；反之，線上環境則要設定為False。如果你在線上環境使用了APP_DEBUG為True的<br/>
配置，就有可能產生暴露敏感資訊給用戶的風險。


## 例外處理器 **App\Exceptions\Handler**
這個處理器類別會處理所有的例外，並主要包含了 **report**、**render**這兩種處理方法。

### 處理器方法 Report
該方法主要用來紀錄例外或將例外傳遞到外部的Bug即時追蹤服務，諸如[Bugsnag](https://www.bugsnag.com/) 或 [Sentry](https://github.com/getsentry/sentry-laravel)。<br/>
預設，report方法只會將例外傳給記錄例外的基礎類別，然而這只是預設行為，你還可以<br/>
自行決定例外紀錄的處裡方式。如果你需要以不同的方式回報不同類型的例外，你可以使用 instanceof<br/>
類型運算子來進行判斷。
```
/**
 * Report or log an exception.
 *
 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
 *
 * @param  \Exception  $exception
 * @return void
 */
public function report(Exception $exception)
{
    if ($exception instanceof CustomException) {
        //
    }

    return parent::report($exception);
}
```
如果你有很多的客製例外，那麼可能會在report方法內部多次地使用instanceof判斷式，這並不是一個<br/>
好方式，建議可以改用其他方法，如後面會提到的[可報告例外](https://github.com/Internaltide/Laradep/blob/master/laratopics/ErrorHandle.md#)(reportable exception)。

#### 輔助方法 Report
有時候你會希望回報例外但不要中斷請求的處理，在client端程式可以透過調用report這個輔助方法，<br/>
它可以讓你使用例外處理器的 report 方法快速的回報例外，而不會顯示錯誤頁面。
```
public function isValid($value)
{
    try {
        // Validate the value...
    } catch (Exception $e) {
        report($e);

        return false;
    }
}
```

#### 依類型忽略例外
例外處理器的 $dontReport 屬性會包含一組例外類型的陣列。陣列中的例外並不會被記錄，例如，<br/>
404 錯誤導致的例外可能就將不該寫入你的日誌中。你可以根據實際需要來新增其他例外類型到<br/>
這組陣列中。
```
/**
 * A list of the exception types that should not be reported.
 *
 * @var array
 */
protected $dontReport = [
    \Illuminate\Auth\AuthenticationException::class,
    \Illuminate\Auth\Access\AuthorizationException::class,
    \Symfony\Component\HttpKernel\Exception\HttpException::class,
    \Illuminate\Database\Eloquent\ModelNotFoundException::class,
    \Illuminate\Validation\ValidationException::class,
];
```

### 處理器方法 Render
render方法主要用來將例外轉換成可以傳送給瀏覽器的Http響應。預設例外處理器會將例外傳入產生<br/>
回應的基礎類別，當然你也可以自由地檢查例外類型以便回傳自訂的響應。
```
/**
 * Render an exception into an HTTP response.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Exception  $exception
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```

### 可報告例外及可渲染例外
為了以不同方式回報不同類型的例外，除了在例外處理器的report與render方法進行例外類型檢查外，<br/>
也可在客製例外類別重新定義report與render方法，當這些方法存在時，框架會自動調用客製例外內部<br/>
所定義的report和render方法。
```
<?php

namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    /**
     * Report the exception.
     *
     * @return void
     */
    public function report()
    {
        //
    }

    /**
     * Render the exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        return response(...);
    }
}
```

## Http例外
一些常見用來描述Http錯誤的響應，諸如401、404與500錯誤。若想要在應用程式的任意地方都可以<br/>
方便的拋出這些響應，可以使用abort這個輔助方法來進行處理。
```
abort(404);
```

abort 輔助方法會立即將例外處理器渲染的例外提升以傳遞至瀏覽器端，當然，你也可以自訂義回應的<br/>
文字內容：
```
bort(403, 'Unauthorized action.');
```

### 自訂Http錯誤頁面
Laravel 可以很容易地為各種的 HTTP 狀態碼設計所要顯示的自訂錯誤頁面。例如，你想自訂404錯誤<br/>
頁面，你可以創建resources/views/errors/404.blade.php，該檔會自動成為應用程式的404顯示畫面。<br/>唯一要注意的是檔名要與狀態碼一致，而透過abort函式發出的 HttpException 實例會作為$exception<br/>變數傳入視圖中。
```
<h2>{{ $exception->getMessage() }}</h2>
```
