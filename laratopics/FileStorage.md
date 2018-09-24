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

## 檔案存取
### 取得檔案
### 儲存檔案
### 刪除檔案

## 目錄操作

## 客製檔案系統
