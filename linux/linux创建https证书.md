# Linux创建https证书

1. 生成密钥server.key，需要输入密码

``` shell
openssl genrsa -des3 -out server.key 2048
```

2. 去除server.key的密码

```shell
openssl rsa -in server.key -out server.key
```

3. 创建服务器证书的申请文件server.csr

```shell
openssl req -new -key server.key -out server.csr
```

4. 创建CA证书

```shell
openssl req -new -x509 -key server.key -out ca.crt -days 3650
```

5. 创建有效期10年的服务器证书server.crt

```shell
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```



