# Laravel Encrypter & Hashing

> Laravel 加密器使用OpenSSL並以AES-256、AES-128進行加密。而且加密後的結果還會使用<br/>
> MAC進行簽名，以檢驗加密值是否被竄改。官方強烈建議改用Laravel Encrypter<br/>
> 來取代自行建構的加密演算，會有更高強度的安全性<br/><br/>
> Reference: https://zh.wikipedia.org/wiki/%E8%A8%8A%E6%81%AF%E9%91%91%E5%88%A5%E7%A2%BC

## Encryption
### 使用前配置
Laravel加密器在加密時會使用一組APP KEY來執行加密演算以提升安全性，沒有該KEY的加密<br/>
都算不上安全。所以，使用Laravel加密功能之前務必於config/app設定好APP_KEY，框架會使用<br/>
安全且隨機的方式來生成APP_KEY。通常建立專案的同時就會產生好一組KEY值，若沒有也可以<br/>
自行使用下列指令產生
```
php artisan key:generate
```

### 使用方法
Helper method: **encrypt / decrypt**
```
/**
    * Store a secret message for the user.
    *
    * @param  Request  $request
    * @param  int  $id
    * @return Response
    */
public function storeSecret(Request $request, $id)
{
    $user = User::findOrFail($id);

    $user->fill([
        'secret' => encrypt($request->secret)
    ])->save();
}
```

Facade method: **encryptString / decryptString**<br/>
encrypt方法預設會使用serialize進行序列化，所以encrypt可以支援物件或陣列的加密，<br/>
只是這樣一來非PHP客戶端就必須執行反序列化的動作。如果不想經過序列化的程序，<br/>
可以改用Crypt Facade的encryptString來對字串進行加密
```
use Illuminate\Support\Facades\Crypt;

$encrypted = Crypt::encryptString('Hello world.');

$decrypted = Crypt::decryptString($encrypted);
```

### 解密失敗時
當加密字串無法被解密時，如MAC無效時，框架就會拋出<br/>
Illuminate\Contracts\Encryption\DecryptException這個例外
```
use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}
```

## Hashing
Laravel 提供了Bcrypt跟Argon2兩種雜湊演算法，內建的LoginController跟RegisterController都預設<br/>
使用Bcrypt來協助註冊與驗證。另外，要特別提的是Bcypt，因為它支援動態調整加密係數因子，所以<br/>
可以根據硬體的能力增加Hash的所需時間。

### 使用配置
config/hashing是主要的設定檔，預設提供了Bcrypt跟Argon2兩種演算法，但Argon2需要PHP版本<br/>
7.2.0以上才有支援。
```
// 設定預設使用的雜湊演算法
'driver' => 'bcrypt',

// Bcrypt的預設係數設定
'bcrypt' => [
    // Adjust Time Cost
    'rounds' => env('BCRYPT_ROUNDS', 10),
],

// Argon的預設係數設定
'argon' => [
    'memory' => 1024, // 調節演算時可用記憶體量
    'threads' => 2,   // 調解演算時使用的threads
    'time' => 2,      // Adjust Time Cost
],
```
PS. 雖然雜湊演算提供了唯一且不可逆的加密過程，但在現今硬體效能蓬勃發展的環境下，<br/>
      暴力破解的成本可能越來越低，所以一些雜湊都會額外提供因子來提高演算時間，藉<br/>
      此緩慢電腦解密速度以提高破解所需成本。

### 基本用法
Facade method: **make**
```
public function update(Request $request)
{
    // Validate the new password length...

    $request->user()->fill([
        'password' => Hash::make($request->newPassword)
    ])->save();
}
```

### 執行期指定使用的加密係數
**Bcrypt**
```
$hashed = Hash::make('password', [
    'rounds' => 12
]);
```
**Argon2**
```
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

### Hashing驗證檢查
使用Hash Facade的check方法檢測未加密的明文字串與hashing string是否等價
```
if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}
```
### 檢測密碼是否需要重新Hash
使用Hash Facade的needsRehash方法可以檢查加密係數是否有異動，進而決定是否重新<br/>
進行雜湊演算產生新的Hash String
```
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```
