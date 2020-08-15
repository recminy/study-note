## 安装Ubuntu16.04服务器版

### 下载发行版

下载地址：https://mirrors.aliyun.com/ubuntu-releases/16.04/ubuntu-16.04.6-server-i386.iso

如上地址失效则自行前往https://developer.aliyun.com/mirror/选择对应server-amd64版本

这里选择16.04的原因主要是因为16.04默认的python版本还是2，后期docker-compose相关工具安装起来方便，省去更多麻烦

### 安装系统

#### 创建虚拟机

VmWare 创建新的虚拟机，注意光盘映像时候选择稍后安装操作系统，如果直接选择ios文件的话，则可能出现操作系统使用简易安装方式导致后续开发部署可能出现问题，等待虚拟机创建完成后，编辑虚拟机再选择对应的映像文件即可

#### 启动并安装

- 系统语言当前版本选择英文，如果选择中文，磁盘分区完安装系统时候会出现bug

- 在选择安装过程语言时候可以选择中文
- 而后直接默认下一步
- 选择输入主机名时候建议不要输入特殊字符，而后输入用户密码，选择不加密主目录，这里当学习使用，所以方便
- 从网络同步时间，保证有网络吧，没有试过断网的情况下是否发生异常
- 磁盘分区直接选默认LVM，默认按照Vm创建虚拟机时候大小直接下一步，确认将修改写入磁盘并配置LVM->继续,下一步->选择是等待1-2分钟
- 安装系统，这里需要等待一段时间
- 弹出节目提示是否配置HTTP代理，选择继续，配置apt,耐性等待软件下载完成，等待时间视网络速度而定，这时候可以来首音乐
- 下一步默认不选择安装更新，直接继续选择安装的软件，光标上下按钮高亮并使用空格选择或者取消要安装的软件，我这里安装M anual package selection、DNS、Samba、OpenSSH Server，等待安装软件
- 将GRUB安装至硬盘：选择默认是继续
- 安装完成后点击继续将会重启系统

#### 配置系统

- 选择时区：tzselect，根据提示调整至北京时间Asia/Shanghai

- 同步时间：nptdate，同步时间，防止后续安装各种服务因为时间出现各种诡异问题

- 配置crontab等默认的编辑器：editor-select

- 更新apt源

  ```shell
  # 先备份
  recminy@master:~$ sudo cp /etc/apt/source.list /etc/apt/source.list.bak
  recminy@master:~$ sudo vim /etc/apt/source.list
  #内容替换成如下
  deb http://mirrors.aliyun.com/ubuntu/ xenial main
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial main
  
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main
  
  deb http://mirrors.aliyun.com/ubuntu/ xenial universe
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
  deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
  
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
  deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
  deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
  
  #保存退出
  recminy@master:~$ sudo apt-get update
  ```

  

