# Laradock Installation Steps


## 安裝Laradock
此處會於Home下面建立一個主要目錄laradep，並將Laradock跟專案目錄置於其下
#### Clone Laradock From Github
```
mkdir /home/laradep
cd /home/laradep
git clone https://github.com/laradock/laradock.git Laradock
```
#### 建立專案目錄
```
mkdir /home/laradep/projects
```

## 設定Laradock
#### 建立環境設定檔 .env
```
cd /home/laradep/Laradock
cp env-example .env
```
#### 修改設定檔內容
請依據需求，自行增加異動項目
```
APP_CODE_PATH_HOST=../projects/

DOCKER_HOST={DOCKER主機IP}

WORKSPACE_TIMEZONE=Asia/Taipei

PHP_VERSION={Laradock支援PHP版本}

WORKSPACE_INSTALL_XDEBUG=true

PHP_FPM_XDEBUG=true
```
#### 容器個別設定
##### Xdebug
```
修改workspace/xdebug.ini 與 php-fpm/xdebug.ini

xdebug.remote_host={IDE所在主機IP}
xdebug.remote_connect_back=1 (當IDE發起請求時，自動返回結果到IDE所在主機)
xdebug.remote_port=9000
xdebug.idekey=VSCODE
xdebug.remote_log="/var/log/xdebug.log"

其餘項目維持原先預設，不予更動
```
##### memcached
```
無特別設定，預設即可
```
##### nginx
```
無特別設定，預設即可
```
##### mysql
```
修改.env檔

MYSQL_VERSION={Mysql Version}
MYSQL_DATABASE={Database Name}
MYSQL_USER={User Name}
MYSQL_PASSWORD={User Password}
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD={Root Password}
```
#### 重啟Docker
```
service docker restart
```

## 啟動Laradock容器服務
首次啟動的容器需要先進行編譯，所需時間會較久。日後若有設定異動，需要重新編譯。
```
docker-compose up -d nginx mysql memcached
docker-compose ps 查詢各容器啟動狀況
```

## 建立Laravel專案
注意!! docker-compose指令需在Laradock根目錄下執行
####  進入workspace工作環境
```
docker-compose exec workspace bash
```
#### 開始建立專案
##### vis composer command
可以指定版本安裝，此處指定為5.4，未指定時會自動安裝適用的最新版本
```
composer create-project laravel/laravel {Project Name} 5.4.* --prefer-dist
```
##### vis laravel command
使用laravel安裝包來建立專案，速度上會比較快。只是似乎無法指定版本。
```
先下載安裝包(僅需下載一次)
composer global require "laravel/installer"

建立專案
laravel new {Project Name}
```
#### 調整目錄權限
主要是要提供web server有可寫的權限，這邊直接將目錄權限設為777。<br/>
線上環境或安全性需求高的開發環境則不建議這樣做。
```
cd {Project Name}
chmod -R 777 storage
chmod -R 777 bootstrap/cache
```

## 建立虛擬主機
#### 使用Laradock提供的nginx設定樣板
```
先離開 workspace 工作環境
exit

建立虛擬主機設定檔
cp /home/laradep/Laradock/nginx/sites/laravel.com.example /home/laradep/Laradock/nginx/sites/laravel.{Project Name}.conf

設定檔異動項目
server_name {Server Name, Ex. www.xxx.com}
root /var/www/{Project Name}/public
```
#### 重啟容器
```
docker-compose down
docker-compose up -d nginx mysql memcached
```

## 編輯windows的hosts，建立DNS對應
```
加入
{DOCKER 主機IP} {連結網址}
Ex. 192.168.1.124 dm.whatsoft.com
```


<br/><br/>
### 其他實用指令
1. docker-compose down      關閉所有容器
2. docker-compose build --no-cache {container-name}     重建docker映像檔
3. docker-compose stop {container-name}     關閉單一容器
4. docker-compose exec {container-name} bash         進入單一容器
5. docker-compose logs {container-name}     查看容器Log
    (NGINX Log file is stored in the logs/nginx directory)
6. docker-compose -f production-docker-compose.yml up -d {container-list}        使用客製設定建立容器
7. docker-compose exec mysql mysql -u homestead -psecret        驗證進入Mysql容器

### 參考
 > https://medium.com/@yfancc20/laradock-%E8%BC%95%E9%87%8F-laravel-%E7%92%B0%E5%A2%83%E7%9A%84%E5%98%97%E8%A9%A6-%E5%B8%B8%E8%A6%8B%E9%8C%AF%E8%AA%A4-2fc6f0c21433
 > https://hk.saowen.com/a/f1f0180051f2d3b81004047f5b4434f09e66ef4ab679f21ff600fc695b2abc8d



 <br/><br/>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laravel development environment documents
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Document Home&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/README.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux OS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Samba&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laradock&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VSCode](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md)