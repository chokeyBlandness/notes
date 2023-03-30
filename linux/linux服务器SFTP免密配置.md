# Linux服务器SFTP免密配置

此案例由A服务器免密登陆B服务器

1. 在A服务器上生成私钥
2. 将A服务器上的私钥scp到B服务器对应的目录上
3. 将B服务器上目录权限放开

具体操作：

两台服务器

A：1.1.1.1

B：2.2.2.2

### 1、生成密钥，在A服务器执行指令

```shell
ssh-keygen -t rsa
```

该指令会在```~/.ssh```目录下生成密钥。

同时确保B服务器上也有```.ssh```文件夹

### 2、发送密钥，在A服务器上执行指令

```shell
cd ~/.ssh
scp id_rsa.public root@2.2.2.2:~/.ssh/id_rsa.public.a # 该指令需要输入B服务器root用户的密码
```

将A服务器上的```id_rsa.public```文件推送到B服务器对应用户的```.ssh```文件夹下。

### 3、赋权，在B服务器上执行指令

```shell
cd ~/.ssh
cat id_rsa.public.a >> authorized_keys
chmod 600 authorized_keys
cd ..
chmod 700 .ssh
```

### 4、测试结果，在A服务器上执行指令

```shell
sftp root@2.2.2.2
```

如果不用输入密码就登陆进去了，即表示免密成功。