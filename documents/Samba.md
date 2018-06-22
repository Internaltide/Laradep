# Samba Installation Steps


## 安裝Samba相關套件
```
yum -y install samba samba-client samba-common
```

## 設定Samba開機啟動
```
systemctl enable smb
systemctl enable nmb
```

## 建立與windows登入用戶同名的Samba使用者
```
useradd {windows登入用戶名稱}(註1)
pdbedit -a {windows登入用戶名稱}(註2)(註3)
```

## Samba設定
#### 命令提示字元查詢所屬工作站群組(工作站網域)
```
net config workstation
```
#### 編輯/etc/samba/smb.conf
```
備份原始檔，開新檔編輯
mv smb.conf smb.conf.bak
vim smb.conf

將以下內容同步進去後再儲存
[global]
workgroup = {工作站群組或網域名稱}                    // 限15字元
server string = Samba Server Version %v           // 自訂主機說明字串
netbios name = laradep                            // Optional，預設會用主機名稱
unix charset = utf8                               // 支援utf8中文
dos charset = cp950                               // 支援big5中文
load printers = no
passdb backend = tdbsam                           // 設定使用者資料檔類型，windows用戶，此項必設

[homes]                                           // 不提供存取家目錄可忽略此區段
comment = Home Directories
browseable = no                                   // 設定是否可透過瀏覽器瀏覽
writeable = yes

[printers]                                        // 不提供印表機分享可忽略此區段
comment = All Printers
path = /var/spool/samba
browseable = no
guest ok = no
writable =no
printable =yes

[Projects]                                        // 公用目錄設定區段，可多個，名稱自訂
path = /home/laradep/projects                     // 自訂分享的目錄位置
comment = 專案工作區                               // 自訂描述說明
public = no                                       // 是否公開，否則須經過身份驗證(同guest ok)
browsable = no
printable = no                                    // 是否可分享列印
writable = yes                                    // 是否可寫，反向設置為read only
valid users = {windows登入用戶名稱}                // 允許的使用者，反向設置為invalid users
write list = root {windows登入用戶名稱}            // 設定可讀寫的使用者(read list為只可讀)
create mode = 0660                                // 檔案初始權限，預設為0744(同create mask)
directory mode = 2770                             // 目錄初始權限，預設為0775(同directory mask)
allow hosts = 192.168.1.                          // 允許的網段
```
PS. 經測試，當windows用戶名稱存在於valid users時，該用戶在已登入狀態下，可直接存取samba公用目錄，省去再一次身分驗證的手續。<br/>網路上很多資料都是設定為公開分享，但利用此特性，則不用犧牲安全性(public維持為no)，又可讓windows登入用戶無障礙存取samba資料。

## 重啟Samba
```
systemctl restart smb
systemctl restart nmb
```

## 防火牆設定放行
```
若您的防火牆是關閉的，此段則可忽略
firewall-cmd --permanent --zone=public --add-service samba
firewall-cmd --reload
```

<br/><br/>
### 註解
1. 先在Linux建立使用者的用意是因為本機需要一樣的帳號，否則samba新增使用者會失敗
2. 可以直接對Linux使用者資料檔進行轉換，快速建立samba的使用者資料<br/>
    cat /etc/passwd | mksmbpasswd.sh > /var/lib/samba/private/smbpasswd
3. 若已存在smbpasswd的舊版使用者資料檔，可利用指令轉換成新式的tdbsam資料格式。<br/>
    新式使用者資料庫格式為二進位，且可儲存更多的密碼政策原則<br/>
    pdbedit -i smbpasswd -e tdbsam
4. smbpasswd使用者資料檔預設放在/var/lib/samba/private/下面
5. 使用者密碼變更一律使用smbpasswd {username}
