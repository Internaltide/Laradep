# Laravel Getting Started

## Installation

#### 安裝Composer
Laravel會利用Composr來管理所有的依賴套件，而Laradock的workspace環境<br/>
其實已準備好composer執行環境了。所以基本上，並不需要再重新安裝。<br/>
但若仍需於Linux下獨立安裝Composer，則可參考下面的安裝步驟
```
下載安裝檔
curl -sS https://getcomposer.org/installer > installer

建立composer.phar指令檔
php installer

移往/usr/local/bin方便全域使用
mv composer.phar /usr/local/bin/composer
```

#### Web Server 設定
主要是設定讓URL可以忽略index.php。<br/>
當然若使用的是Laradock，這些都會自動被包裝好。
##### Apache
需先啟用apache的mod_rewrite模組。<br/>
預設，專案public目錄下的.htaccess便已加入了相關設定<br/>
若無法生效，可嘗試使用下面設定進行取代。
```
Options +FollowSymLinks
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
##### Nginx
```
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

## Confiruration
## 目錄結構
## Deployment