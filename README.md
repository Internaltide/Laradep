# Laradep
Laravel development environment - using laradock<br/>

適用的Linux版本：CentOS7<br/>
環境：Windows + Linux (所以編輯器部分會安裝在Windows下面)

## Installation
### [Linux OS](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md)
 * [安裝虛擬機軟體](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E5%AE%89%E8%A3%9D%E8%99%9B%E6%93%AC%E6%A9%9F%E8%BB%9F%E9%AB%94)
 * [開始安裝Linux](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E9%96%8B%E5%A7%8B%E5%AE%89%E8%A3%9Dlinux)
 * [網路設定](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E7%B6%B2%E8%B7%AF%E8%A8%AD%E5%AE%9A)
 * [Yum源更新](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#yum%E6%BA%90%E6%9B%B4%E6%96%B0)
 * [Linux套件更新](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#linux%E5%A5%97%E4%BB%B6%E6%9B%B4%E6%96%B0)
 * [常用套件安裝](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E5%B8%B8%E7%94%A8%E5%A5%97%E4%BB%B6%E5%AE%89%E8%A3%9D)
 * [Git版本更新](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#git%E7%89%88%E6%9C%AC%E6%9B%B4%E6%96%B0)
 * [關閉防火牆](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E9%97%9C%E9%96%89%E9%98%B2%E7%81%AB%E7%89%86)
 * [關閉SELinux](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#%E7%B7%A8%E8%BC%AF-etcselinuxconfig%E9%97%9C%E9%96%89selinux)
  * [chronyc自動校時](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md#chronyc%E8%87%AA%E5%8B%95%E6%A0%A1%E6%99%82)
### [Samba](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md)
 * [安裝Samba相關套件](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#%E5%AE%89%E8%A3%9Dsamba%E7%9B%B8%E9%97%9C%E5%A5%97%E4%BB%B6)
 * [設定Samba開機啟動](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#%E8%A8%AD%E5%AE%9Asamba%E9%96%8B%E6%A9%9F%E5%95%9F%E5%8B%95)
 * [建立與windows登入用戶同名的Samba使用者](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#%E5%BB%BA%E7%AB%8B%E8%88%87windows%E7%99%BB%E5%85%A5%E7%94%A8%E6%88%B6%E5%90%8C%E5%90%8D%E7%9A%84samba%E4%BD%BF%E7%94%A8%E8%80%85)
 * [Samba設定](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#samba%E8%A8%AD%E5%AE%9A)
 * [重啟Samba](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#%E9%87%8D%E5%95%9Fsamba)
 * [防火牆設定放行](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md#%E9%98%B2%E7%81%AB%E7%89%86%E8%A8%AD%E5%AE%9A%E6%94%BE%E8%A1%8C)
### [Docker](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md)
 * [安裝Docker](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md#%E5%AE%89%E8%A3%9Ddocker)
 * [安裝docker-compose](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md#%E5%AE%89%E8%A3%9Ddocker-compose)
### [Laradock](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md)
 * [安裝Laradock](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E5%AE%89%E8%A3%9Dlaradock)
 * [設定Laradock](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E8%A8%AD%E5%AE%9Alaradock)
 * [啟動Laradock容器服務](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E5%95%9F%E5%8B%95laradock%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8B%99)
 * [建立Laravel專案](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E5%BB%BA%E7%AB%8Blaravel%E5%B0%88%E6%A1%88)
 * [建立虛擬主機](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E5%BB%BA%E7%AB%8B%E8%99%9B%E6%93%AC%E4%B8%BB%E6%A9%9F)
 * [建立DNS對應](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md#%E7%B7%A8%E8%BC%AFwindows%E7%9A%84hosts%E5%BB%BA%E7%AB%8Bdns%E5%B0%8D%E6%87%89)
### [VSCode](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md)
 * [Visual Studio Code 相關環境安裝](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#visual-studio-code-%E7%9B%B8%E9%97%9C%E7%92%B0%E5%A2%83%E5%AE%89%E8%A3%9D)
 * [Visual Studio Code 相關外掛安裝](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#visual-studio-code-%E7%9B%B8%E9%97%9C%E5%A4%96%E6%8E%9B%E5%AE%89%E8%A3%9D)
 * [Visual Studio Code 相關設定](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#visual-studio-code-%E7%9B%B8%E9%97%9C%E8%A8%AD%E5%AE%9A)
 * [Visual Studio Code工作區建立](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#visual-studio-code%E5%B7%A5%E4%BD%9C%E5%8D%80%E5%BB%BA%E7%AB%8B)
 * [Visual Studio Code Debug設定](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#visual-studio-code-debug%E8%A8%AD%E5%AE%9A)
  * [使用powershell建立windows的公私鑰](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#%E4%BD%BF%E7%94%A8powershell%E5%BB%BA%E7%AB%8Bwindows%E7%9A%84%E5%85%AC%E7%A7%81%E9%91%B0)
  * [遠端建立Repository(以Gitlab為例)](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md#%E9%81%A0%E7%AB%AF%E5%BB%BA%E7%AB%8Brepository%E4%BB%A5gitlab%E7%82%BA%E4%BE%8B)
### [Frontend Required](https://github.com/Internaltide/Laradep/blob/master/documents/Frontend.md)
 * [安裝nodejs](https://github.com/Internaltide/Laradep/blob/master/documents/Frontend.md#%E5%AE%89%E8%A3%9Dnodejs)
 * [安裝yarn](https://github.com/Internaltide/Laradep/blob/master/documents/Frontend.md#%E5%AE%89%E8%A3%9yarn)

## Laravel Topic
### Getting Started
#### [ - Installation](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingStarted.md#Installation)
#### [ - Configuration](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingStarted.md#Configuration)
#### [ - 目錄結構](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingStarted.md#目錄結構)
#### [ - Deployment](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingStarted.md#Deployment)
### Architecture Concepts
#### [ - Request Lifecycle](https://github.com/Internaltide/Laradep/blob/master/laratopics/Lifecycle.md)
#### [ - Service Container](https://github.com/Internaltide/Laradep/blob/master/laratopics/ServiceContainer.md)
#### [ - Service Providers](https://github.com/Internaltide/Laradep/blob/master/laratopics/ServiceProvider.md)
#### [ - Facades](https://github.com/Internaltide/Laradep/blob/master/laratopics/Facades.md)
#### [ - Contracts](https://github.com/Internaltide/Laradep/blob/master/laratopics/Contracts.md)
### Basic
#### [ - Routing](https://github.com/Internaltide/Laradep/blob/master/laratopics/routing.md)
### Frontend
#### [ - Laravel Mix](https://github.com/Internaltide/Laradep/blob/master/laratopics/LaravelMix.md)
#### [ - Localization](https://github.com/Internaltide/Laradep/blob/master/laratopics/Localization.md)
### Security
#### [ - Authentication](https://github.com/Internaltide/Laradep/blob/master/laratopics/Authentication.md)
#### [ - Encryption & Hashing](https://github.com/Internaltide/Laradep/blob/master/laratopics/Encryption.md)
#### [ - Password Reset](https://github.com/Internaltide/Laradep/blob/master/laratopics/PasswordReset.md)
### Database
#### [ - Getting Started](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingDatabaseStart.md)
#### [ - Query Builder](https://github.com/Internaltide/Laradep/blob/master/laratopics/QueryBuilder.md)
#### [ - Pagination](https://github.com/Internaltide/Laradep/blob/master/laratopics/Pagination.md)
#### [ - Migrations](https://github.com/Internaltide/Laradep/blob/master/laratopics/Migrations.md)
#### [ - Seeding](https://github.com/Internaltide/Laradep/blob/master/laratopics/Seeding.md)
#### [ - Redis](https://github.com/Internaltide/Laradep/blob/master/laratopics/Redis.md)
### Eloquent ORM
#### [ - Getting Started](https://github.com/Internaltide/Laradep/blob/master/laratopics/GettingORMStarted.md)
#### [ - Relationships](https://github.com/Internaltide/Laradep/blob/master/laratopics/Relationships.md)
#### [ - Mutators](https://github.com/Internaltide/Laradep/blob/master/laratopics/Mutators.md)

## 常用快捷
 - Alt+滑鼠左鍵 可同時編輯多行
 - Ctrl+Shift+V 預覽Markdown File
 - Ctrl+F Search
 - Ctrl+H Search and Replace
 - Ctrl+Shift+L 可同時修改多行的相同處
 - Ctrl+Shift+P 開啟命令選擇區
 - Ctrl+/ 單行註解
 - Shift+Alt+A 區塊註解

## 其他主題