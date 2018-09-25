# Laravel File Storage

> Laravel提供了一個強大的檔案系統抽象層，整合了本地、Amazon S3、Rackspace雲儲存等，<br/>
> 並提供簡單易用的驅動程式。更棒的是Laravel為每種檔案系統都提供了一致的接口，所以<br/>
> 在各檔案系統間換變得非常的簡單。

## 配置方式
config/filesystems為檔案儲存系統的配置檔案。裡面可以設定你的所有磁碟，每個磁碟都代表了<br/>
一個儲存驅動跟儲存位置。各類型儲存系統的配置方式都有範例在裡頭，只需根據自身的需求<br/>
調整設定即可。

### 公開磁碟
公開磁碟預期就是用於那些可以公開訪問的文件。預設，公開磁碟會使用local類型的驅動並且將<br/>
檔案存在目錄storage/app/public下。如果希望能夠在web上被存取，可以透過建立對應的軟連結到<br/>
public/storage下來達成。

你可以透過指令來為公開目錄建立軟連結
```
php artisan storage:link
```

透過asset helper 方法來取得公開目錄文件的訪問URL
```
echo asset('storage/file.txt');
```

### 本地驅動
使用local本地驅動時，所以的文件操作都是相對於設定中檔中所配置的根目錄，預設是使用<br/>
storage/app，因此以下面例子而言就是將內容儲存到storage/app/file.txt。
```
Storage::disk('local')->put('file.txt', 'Contents');
```

### 使用各類驅動的先決條件
部分驅動需要對應的底層套件，所以需要先使用Composer安裝對應的套件。另外，下述套件是為了提升效能用的，<br/>
為所有驅動的必要套件。驅動各自的專屬套件則列在各自的敘述區塊中。<br/>
**Require CachedAdapter**
```
league/flysystem-cached-adapter ~1.0
```

#### FTP
預設，config/filesystem並沒有FTP Driver的範例設定，但Laravel本身其實就對FTP就有很好的支持。<br/>
若要使用FTP，只需參考下面的設定進行配置即可。
```
'ftp' => [
    'driver'   => 'ftp',
    'host'     => 'ftp.example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // Optional FTP Settings...
    // 'port'     => 21,
    // 'root'     => '',
    // 'passive'  => true,
    // 'ssl'      => true,
    // 'timeout'  => 30,
],
```

#### SFTP
所需套件
```
league/flysystem-sftp ~1.0
```
同樣地，與FTP Driver一樣，SFTP也沒有範例設定。若要使用SFTP，請參考下例範例進行配置
```
'sftp' => [
    'driver' => 'sftp',
    'host' => 'example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // Settings for SSH key based authentication...
    // 'privateKey' => '/path/to/privateKey',
    // 'password' => 'encryption-password',

    // Optional SFTP Settings...
    // 'port' => 22,
    // 'root' => '',
    // 'timeout' => 30,
],
```

#### Amazon S3
所需套件
```
league/flysystem-aws-s3-v3 ~1.0
```
預設，config/filesystem裡頭就有S3的配置範例，只需要將各項目調整成自己的設定即可。另外，Laravel也特別設計<br/>
讓環境變數名稱符合AWS CLI使用的命名，以其方便使用。
```
        's3' => [
            'driver' => 's3',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('AWS_BUCKET'),
            'url' => env('AWS_URL'),
        ],
```

#### Rackspace
所需套件
```
league/flysystem-rackspace ~1.0
```
Rackspace也沒有範例設定，請參考下列範例進行配置
```
'rackspace' => [
    'driver'    => 'rackspace',
    'username'  => 'your-username',
    'key'       => 'your-key',
    'container' => 'your-container',
    'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
    'region'    => 'IAD',
    'url_type'  => 'publicURL',
],
```

### 快取
你可以為每一個磁碟來配置其快取設定，方式就是在磁碟設定陣列內部再附加一個<br/>
快取的設定陣列，如下
```
's3' => [
    'driver' => 's3',

    // Disk Cache Options...
    'cache' => [
        'store' => 'memcached',    //
        'expire' => 600,                // 快取時間
        'prefix' => 'cache-prefix', // 快取名稱前綴
    ],
],
```

## 獲取磁碟實例
Laravel提供Storage Facade來與底層的磁碟做溝通。例如，你可以使用put方法來將檔案儲存到預設的磁碟中。<br/>
若你需要儲存到特定磁碟而不是預設磁碟，則需要再掉用put方法前先調用disk方法來指定互動的磁碟。
```
use Illuminate\Support\Facades\Storage;

// 儲存到預設磁碟
Storage::put('avatars/1', $fileContents);

// 儲存到 S3磁碟
Storage::disk('s3')->put('avatars/1', $fileContents);
```

## 檔案存取控制
### 文件檢索
#### 取得檔案內容
```
$contents = Storage::get('file.jpg');
```
#### 判斷檔案是否存在
```
$exists = Storage::disk('s3')->exists('file.jpg');
```
#### 下載檔案
download方法被用來產生強制用戶下載之回應，第一參數為指定下載的文件路徑<br/>
第二參數為用戶下載檔案所看到的文件名稱，第三參數則為欲指定的Header，該參數須以陣列型態傳入。
```
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```
#### 檔案網址
當使用的是驅動是local或s3時，可以使用url方法來取得文件網址。local驅動通常只是在給定的路徑前加上/storage<br/>
並返回相對的URL；s3驅動則返還完整的遠端URL。
```
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```
#### 臨時網址
當使用的是s3或rackspace驅動時，可以使用temporaryUrl為文件建立一個臨時性的存取網址，該方法接受文件路徑<br/>
為第一參數，DateTime實例為第二參數。DataTime實例則用來宣告網址何時過期。
```
$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);
```
#### 自訂主機
使用local驅動時，若要自定義主機位置，可以在public配置區塊中添加url項來定義
```
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
],
```
#### 文件Metadata
除了檔案讀寫以外，Laravel也提供方法來取得文件的資訊。例如，<br/>
size方法可以得到文件的大小。
```
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```
lastModified方法可以到到文件最後更改時間(時間格式為UNIX timestamp)
```
$time = Storage::lastModified('file.jpg');
```

### 儲存檔案
使用put方法可以將原始數據內容存進文件中。但傳入put方法的是PHP的resource類型變數，則會使用到<br/>
底層的串流機制，一般檔案較大時會建議使用串流的方式。
```
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```
#### 自動串流
使用putFile 或 putFileAs兩種方法時，Laravel會自動管理將指定文件內容串流到指定位置的行為，<br/>
該方法接受Illuminate\Http\File 或 Illuminate\Http\UploadedFile這兩種檔案實例。
```
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// 會自動產生惟一ID作為檔名
Storage::putFile('photos', new File('/path/to/photo'));

// 手動指定檔名為photo.jpg
Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```
關於putFile方法，還有幾項重要提醒是，該法僅需要指定要儲存的目錄，因為檔名會自動產生。<br/>
而其副檔名則根據檔案的MIME TYPE來決定。而最後該方法會回傳包含檔名的完整路徑。<br/>

putFile 與 putFileAs還能夠指定檔案的可視程度。例如，將文件存入S3並想設定為公開時，就特別有用。
```
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

#### 數據附加與數據Prepend
```
// 從文件開頭寫入數據
Storage::prepend('file.log', 'Prepended Text');

// 從文件尾端開始附加數據
Storage::append('file.log', 'Appended Text');
```

#### 檔案複製與移動
```
// 複製檔案到新位置
Storage::copy('old/file.jpg', 'new/file.jpg');

// 移動檔案到新位置(亦可用來更改檔名)
Storage::move('old/file.jpg', 'new/file.jpg');
```

### 檔案上傳
在web應用中，檔案上傳是非常常見的實作案例。Larave提供了store這個方法，可以<br/>
更為方便的處理檔案上傳，常為uploaded file實例所調用。
```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class UserAvatarController extends Controller
{
    /**
     * Update the avatar for the user.
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```
上述例子中，要注意的是，我們一樣只需指定目錄即可，因為store方法也會自動產生惟一ID來做為檔名，<br/>
並以MIME TYPE來決定副檔名，同樣store也會返回完整路徑。<br/>

使用putFile來等價完成上述例子的行為
```
$path = Storage::putFile('avatars', $request->file('avatar'));
```

#### 指定檔名
#### 指定磁碟

### 檔案可視性

### 刪除檔案

## 目錄操作

## 客製檔案系統
