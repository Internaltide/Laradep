# Linux OS Installation Steps


## 安裝虛擬機軟體
可擇一安裝。這邊的安裝範例以visualbox為主

 - Vmware Workstaion Player<br/>
   一開始也是用這個，但vmware更新時卻發生了嚴重死當，重開後，Vmware再也回不來了。<br/>
  嘗試重新安裝、修正、其他許多Google來的方法都無法挽救。猜測可能是win10環境造成。<br/>
  所以若是 Windows10系統的使用者，請小心服用。

 - Visualbox<br/>
   本開發環境會使用VBox進行建置

 - HyperV<br/>
   注意!! HyperV跟其他的虛擬軟體會有衝突。如果你不知道如何解除衝突也沒這種需求，建議擇一安裝就好。

## 開始安裝Linux
這邊裝的是CentOS7最小安裝，安裝過程已簡化許多。<br/>
如果沒有特殊需求，基本上，就是一直Next用預設值就可以了。<br/>
只有在用戶設定頁面，可以視需要建立用戶跟root密碼。

## 網路設定
#### 編輯 /etc/hosts，設定網址與IP的對應
```
<Linux Host IP> <Hostname>
192.168.1.124 whatsoft.com
```
#### 編輯 /etc/resolv.conf，加入下列設定
```
search whatsoft.com (註1)
name server 168.95.1.1
name server 8.8.8.8
```
#### 編輯 /etc/sysconfig/network，加入下列設定
```
NETWORKING=yes (註2)
HOSTNAME=www.whatsoft.com
GATEWAY=192.168.1.1(好像沒用了)
```
#### 編輯 /etc/sysconfig/network-scripts/ifcfg-{網卡編號}，異動下列項目(其餘既存項目可保留不動)
```
NAME={網卡編號}
DEVICE={網卡編號} (註3)
BOOTPROTO=static (註4)
ONBOOT=yes (註5)
IPADDR=192.168.1.124
NETMASK=255.255.255.0
NETWORK=192.168.1.0 (註6)
GATEWAY=192.168.1.1
DNS1=168.95.1.1
DNS2=8.8.8.8
USERCTL=no (註7)
```
#### 連線開通(啟用網卡連線 或 重啟網路服務)
```
ifup {網卡編號}                  => 啟用網卡連線
service NetworkManager restart  => 重啟網路服務
```

## Yum源更新
預設可能是流量大且緩慢的國外源或是無效源。更新一下會讓往後套件安裝較為順暢
#### 更改源
```
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.bak
vi CentOS-Base.repo
將文件yumRepo.txt的內容同步進去後再儲存
```
Get Repository Content From [yumRepo.txt](https://github.com/Internaltide/Laradep/blob/master/documents/yumRepo.txt)

#### Yum 快取更新
```
yum clean all
yum list
```

## Linux套件更新
```
yum update  (只針對已安裝且未Deprecated套件)
yum upgrade (無差別全數更新，較常用在版本升級)
```

## 常用套件安裝
```
yum -y install vim
yum -y install wget
yum -y install telnet
```

## Git版本更新
#### 移除舊版
```
yum remove git
```
#### 安裝依賴
```
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
yum -y install gcc perl-ExtUtils-MakeMaker
```
#### 安裝新版Git
```
wget https://github.com/git/git/archive/v2.17.1.tar.gz
tar -xzvf v2.17.1.tar.gz
cd git-2.17.1
make prefix=/usr/local/git all
make prefix=/usr/local/git install
```
#### 添加環境變量
```
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/bashrc
source /etc/bashrc
```
#### 檢查版本
```
git --version
```
#### 仍為舊版時，需再執行一次移除
```
yum remove -y git
source /etc/bashrc
git --version
```
PS. 雖然已移除舊版，但安裝新版Git後，/usr/bin/下會再度出現舊版git應用程式。<br/>
      原因不明，只能再執行一次yum移除予以解決。

#### 設定Git使用者資訊
```
git config --global user.name {User Name}
git config --global user.email {User Email}
```

## 關閉防火牆
因為是內部自己的開發環境，關掉後會比較省事<br/>
如果您想要玩玩Centos的新式防火牆firewalld，那可選擇忽略此步驟，
只是需要自行設定規則放行，本文件不會著墨太多
```
systemctl stop firewalld
systemctl disable firewalld
```

## 編輯 /etc/selinux/config，關閉SELinux
原因同防火牆，關掉會比較少問題
```
SELINUX=Enforcing 改成 SELINUX=disabled
setenforce 0

執行getenforce檢查是否關閉成功(Enforcing:啟用；Permissive:臨時停用；Disabled:停用)
```

<br/><br/>
### 註解
1. 例如查詢名稱"www2"解析失敗時，系統會檢查search設定值，假若設定為 whatsoft.com，則會嘗試改用www2.whatsoft.com進行二次查詢
2. 用以表示網路正處於運作中
3. 需同網卡編號，否則設定無法生效
4. static表示使用固定IP，可設定為dhcp改用浮動IP
5. 宣告是否開機啟動網卡
6. 設定所在網段
7. 是否允許非root使用者控制



<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laravel development environment documents
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Document Home&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/README.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux OS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Samba&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Docker.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laradock&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VSCode](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md)