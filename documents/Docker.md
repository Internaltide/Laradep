# Docker Installation Steps


## 安裝Docker
#### 新增Docker源
```
vim /etc/yum.repos.d/docker.repo
將文件dockerRepo.txt的內容同步進去再儲存
```
Get Docker Repository Content From [dockerRepo.txt](https://github.com/Internaltide/Laradep/blob/master/documents/dockerRepo.txt)

#### Yum 快取更新
```
yum clean all
yum list
```
#### 使用官方提供的自動安裝指令
```
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```
#### 檢查docker版本訊息，確認安裝
```
docker -v
```
#### 設定開機啟用
```
systemctl enable docker( chkconfig --level 345 docker on )
service docker start
```

## 安裝docker-compose
```
yum -y install epel-release
yum -y install python-pip
pip install --upgrade pip
pip install docker-compose
docker-compose --version
```



<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laravel development environment documents
------
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Document Home&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/README.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux OS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Linux%20OS.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Samba&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Samba.md)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Laradock&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;](https://github.com/Internaltide/Laradep/blob/master/documents/Laradock.md)[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VSCode](https://github.com/Internaltide/Laradep/blob/master/documents/VSCode.md)