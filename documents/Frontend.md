# Laravel Frontend Package Installation
主要說明Laravel前端開發所需的套件安裝方式

## 安裝nodejs
安裝完nodejs後，會自動也安裝上前端套件管理工具npm
#### 添加官方庫
```
curl -sL https://rpm.nodesource.com/setup_8.x | sudo -E bash -

PS. 其中setup_8.x的數字表示nodejs的大版本，建議使用官網上的穩定版本
```
#### 使用yum安裝
```
yum install -y nodejs
```

## 安裝前端管理套件yarn
Facebook發表的新一代前端JS套件管理工具，據說安裝速度比npm快。<br/>
可用來取代npm，但非必要套件，你仍可以使用既有的npm來管理。<br/>
Laravel也是預設使用npm，目前不確定laravel環境下是否可直接由yarn取代，或許需要進行調整

#### 添加官方庫
```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
```
#### 使用yum安裝
```
yum install -y yarn
```