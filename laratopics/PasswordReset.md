# 密碼重設功能

> **快速上手**，你仍可透過使用**php artisan make:auth**指令直接建立用戶認證，<br/>
> 如此一來，預設就已經包含了密碼重設程序。主要的控制器如下<br/>
> - Auth\ForgotPasswordController
> - Auth\ResetPasswordController<br/>
>
> 裡頭包含了密碼重設連結的郵件寄送跟重設密碼的所有邏輯

## 必要條件
 - 用戶模型必須使用Illuminate\Notifications\Notifiable trait。
 - 用戶模型必須實作Illuminate\Contracts\Auth\CanResetPassword這個介面，<br/>
   Illuminate\Auth\Passwords\CanResetPassword這個Trait就包含了對應的實作方法。<br/>

## 資料庫注意事項
密碼重設功能需要一個能夠儲存reset token的地方，因此你需要為用戶資料表建立一個儲存<br/>
reset token的欄位。預設內建的用戶遷移檔就包含了相關設定，若是自建的用戶資料表，<br/>
則可以參照該遷移檔重新設定。

## 路由設置
參考Laravel內建的認證路由，密碼重設所需路由如下：
```
$this->get('password/reset', 'Auth\ForgotPasswordController@showLinkRequestForm')->name('password.request');
$this->post('password/email', 'Auth\ForgotPasswordController@sendResetLinkEmail')->name('password.email');
$this->get('password/reset/{token}', 'Auth\ResetPasswordController@showResetForm')->name('password.reset');
$this->post('password/reset', 'Auth\ResetPasswordController@reset');
```

## 視圖建立
透過指令快速建立認證時，相關的視圖也會一併建立並放置在resources/views/auth/passwords下面，<br/>
可以其為基礎，按需求進行客製修改。

## 密碼重設後
密碼成功重設後，預設是讓用戶自動登入並重導到/home。可透過ResetPasswordController的redirectTo屬性來<br/>
覆寫預設的重定向位置。
```
protected $redirectTo = '/dashboard';
```

## 密碼重設令牌有效時間
預設，reset token有效時間為一個小時，但可以透過config/auth的expire設定項目來變更

## 客製化
### 自訂使用的守衛
為了讓User在完成重設密碼後可以直接登入，Laravel預設實作會在重設密碼後再調用guard並執行login方法。<br/>
而我們可以在ResetPasswordController中重寫guard方法，予以覆寫預設使用的守衛。<br/>
```
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```
PS. 若要更改重設密碼的實作方式，可以重寫ResetPasswordController的resetPassword方法。<br/>
       預設的restPassword則透過ResetsPasswords Trait來提供。

### 自訂使用的密碼代理
config/auth中可以設定多組的密碼代理設定，以因應不同用戶表的不同行為。<br/>
執行期要指定不同的設定需透過在ForgotPasswordController及ResetPasswordController<br/>
兩個控制器內重寫broker方法。
```
protected function broker()
{
    return Password::broker('name');
}
```

### 變更密碼重設的通知方式
首先，必須要重寫User模型的sendPasswordResetNotification方法。<br/>
可以輕鬆透過修改通知類來進行變更，也可以指定任何其他的通知類別。<br/>
該方法接收的第一個參數就是reset token。
```
/**
 * Send the password reset notification.
 *
 * @param  string  $token
 * @return void
 */
public function sendPasswordResetNotification($token)
{
    $this->notify(new ResetPasswordNotification($token));
}
```
