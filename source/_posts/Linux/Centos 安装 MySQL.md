---
categories:
  - Linux
---
# Centos 安装 MySQL

## 下载并安装 MySQL

```sh
wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm &&
yum -y install mysql57-community-release-el7-10.noarch.rpm &&
yum install -y mysql-community-server --nogpgcheck
```

![image-20240301145145246](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403011451012.png)

## 启动 MySQL

```sh
systemctl start mysqld.service
```

## 查看 MySQL 初始密码

```sh
grep "password" /var/log/mysqld.log
```

![image-20240301145246431](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403011452787.png)

## 登录 & 修改密码

```sh
mysql -uroot -p
```

![image-20240301145437944](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403011454512.png)

```sh
set global validate_password_policy=0;  #修改密码安全策略为低（只校验密码长度，至少8位）。
ALTER USER 'root'@'localhost' IDENTIFIED BY '12345678';
```

![image-20240301145508549](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403011455043.png)

## 授予 root 用户远程管理权限

```sh
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '12345678';
```

![image-20240301145737027](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403011457305.png)