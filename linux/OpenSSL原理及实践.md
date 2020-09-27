# SSL/TLS

SSL(Secure Sockets Layer 安全套接字协议),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层与应用层之间对网络连接进行加密。

### SSL会话三步骤

- 客户端向服务端索要并验证证书
- 双方协商生成“会话秘钥“
- 双方采用“会话秘钥”进行加密通信

- 第一阶段：clientHello
- 第二阶段：ServerHello
  - 确认使用的加密通信版本
- 第三阶段：
  - 验证服务器证书（发证机构、证书完整性、有效期、证书持有者）

### Linux系统上的随机数生成

- /dev/random 仅从熵池中返回随机数，随机数用尽，阻塞；
- /dev/urandom 从熵池中返回随机数；随机数用尽，会利用软件生成伪随机数，非阻塞，伪随机数不安全

- 熵池中的随机数来源
  - 硬盘IO中断时间间隔
  - 键盘IO中断时间间隔

### openSSL构建私有CA、自签证书及吊销

#### 构建私有CA

在确定配置为CA的服务器上生成一个自签证书，并为CA提供所需要的目录及文件，具体步骤：

- 生成私钥

  ```shell
  root@node01:~/# (umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096) # 文件名及存放目录固定
  ```

- 生成自签证书

  ```shell
  root@node01:~/# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650
  #根据提示输入证书的相关信息email为可选，其他必须正确填写，否则后续的签署CA证书可能失败
  ```

  选项说明：

  - -new：生成新证书签署请求
  - -x509：生成自签格式证书，专用于创建私有CA
  - -key：生成请求时用到的私有文件路径
  - -out：生成的请求文件路径，如果自签操作将直接生成签署过程的证书
  - -days：证书的有效时长，单位天

- 为CA提供所需的目录及文件

  ```shell
  root@node01:~/# mkdir -pv /etc/pki/CA/{certs,crl,newcerts}
  root@node01:~/# touch /etc/pki/CA/{serial,index.txt}
  #申明第一个证书序列号号码，如从01开始编号
  root@node01:~/# echo 01 > /etc/pki/CA/serial 
  ```

#### 请求签署证书步骤

需要用到证书进行安全通信的服务器则需要向CA服务器请求签署，以http举例，步骤如下：

- 要用主机生成私钥

  ```shell
  root@node01:~/# mkdir /etc/httpd/ssl
  root@node01:~/# cd /etc/httpd/ssl
  root@node01:~/# (umask 077; openssl genrsa -out /etc/httpd/ssl/httpd.key 2048)
  ```

- 生成证书签署请求

  ```shell
  root@node01:~/# openssl req -new -key /etc/httpd/ssl/httpd.key -out /etc/httpd/ssl/httpd.csr -days 365
  #PS：由于是私有CA，这里的证书的服务器的公司名要保持一致，否则可能无法通过
  ```

- 将请求通过可靠方式发送给CA主机

  如:```scp /etc/httpd/ssl/httpd.csr root@CAHOST:/data/httpd.csr```

- 在CA主机签署证书

  ```shell
  root@node01:~/# openssl ca -in /data/httpd.csr -out /etc/pki/CA/certs/httpd.crt -days 365
  #PS：签署时候的证书相关的
  #如查看证书信息
  root@node01:~/# openssl x509 -in /etc/pki/CA/certs/httpd.csr -noout -serial -subject
  ```

  

#### 吊销证书步骤

- 客户端获取要吊销的证书的serial（在使用证书的主机上执行）

  ```shell
  root@node01:~/# openssl x509 -in /PATH/FILE.crt -noout -serial -subject
  ```

- CA主机吊销

- 生成吊销证书的吊销编号（首次吊销证书时执行）

  - 先根据客户的提交的serial和subject信息，对比其与本机数据库的index.txt中存储的是否一致

  - 执行吊销

    ```shell
    root@node01:~/# openssl ca -revoke /etc/pki/CA/newcerts/SERIAL.pem
    #SERIAL为要吊销证书的序列号
    ```

    

- 更新证书吊销列表

  ```shell
  root@node01:~/# openssl ca -gencl -out FILE.crl
  ## 查看crl文件
  root@node01:~/#  openssl crl -in /PATH/CRL_FILE.crt -noout -text
  ```

  