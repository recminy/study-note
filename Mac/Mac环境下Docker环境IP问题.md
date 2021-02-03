# Mac环境下Docker环境

最近开始从windows开发环境切换到Mac，Mac下使用docker用的是**docker for mac**。

通过``docker-compose``编排web应用，因为使用的是本地域名，故将域名全部解析为``127.0.0.1``,后面发现这样的方式其实不可行，因为此时的容器间的Network是独立的，127.0.0.1指向的是容器自身的网络名称空间。

故打算为每个容器分配一个固定IP，将本本地域名解析至容器固定IP，这样就可以通过域名访问具体的应用，配置如下

```yaml
version: '2'
services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: nginx-proxy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /Users/recminy/docker/logs/nginx:/var/log/nginx
      - /Users/recminy/docker/ssh/certs:/etc/nginx/certs:ro
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - /Users/recminy/docker/server/nginx-proxy/my_proxy.conf:/etc/nginx/conf.d/my_proxy.conf:ro
      
  pma: #phpmyadmin
    image: centos/php:7.2
    container_name: ma
    restart: always
    networks:
      default: #使用默认的bridge网络
        ipv4_address: 172.18.0.100
    environment:
      - PHPINI=production
      - VIRTUAL_HOST=pma.localhost.com
    volumes:
      - /Users/recminy/web/pma:/var/www/app
      
  oauth-api:
    image: centos/php:7.2
    container_name: account-api
    restart: always
    networks:
      default:
        ipv4_address: 172.18.0.101
    hostname: oauth-api.com
    privileged: true
    environment:
      - PHPINI=production
      - VIRTUAL_HOST=oauth-api.com
    volumes:
      - /Users/recminy/web/oauth-api:/var/www/app
      
  web-api:
    image: centos/php:7.2
    container_name: web-api
    restart: always
    hostname: api.com
    networks:
      default:
        ipv4_address: 172.18.0.102
    privileged: true
    environment:
      - PHPINI=production
      - VIRTUAL_HOST=api.com
      - DOCUMENTROOT=/var/www/app/public
    volumes:
      - /Users/recminy/web/web-api:/var/www/app
      
  shopStore:
    image: centos/php:7.2
    container_name: shopStore
    restart: always
    hostname: school.com
    privileged: true
    networks:
      default:
        ipv4_address: 172.18.0.103
    environment:
      - PHPINI=production
      - VIRTUAL_HOST=www.shop.com
      - DOCUMENTROOT=/var/www/app/public
    volumes:
      - /Users/recminy/web/shop:/var/www/app
```

结果发现使用宿主机始终无法用IP访问容器，倒腾了半天原以为是网络配置问题，后来怀疑是否是mac的限制，因为``ifconfig``根本查不到docker0桥，结果查询资料发现是Mac系统受限的问题：**Docker 是利用 Linux 的 Namespace 和 Cgroups 来实现资源的隔离和限制，容器共享宿主机内核，所以 Mac 本身没法运行 Docker 容器，Docker for Mac 也是在本地跑了一个虚拟机来运行 Docker容器，且这个容器是非常轻量级的**

但是也不是没有办法解决，很多网友给了多种方式

- ```Docker Toolbox```不推荐，因为```docker for mac```即设计用来替换Toolbox，

- OpenVPN server:在 Docker for Mac 的虚拟机里跑一个 OpenVPN Server，然后从本地连过去

- SOCKS,docker实验室功能，且每次运行都要配置一次，麻烦等官网stable了再考虑，不过可能也是趋势

- docker-connector：这里我也是通过这种方式配置的

  通过 brew 安装 docker-connector

  ```shell
  brew install wenjunxiao/brew/docker-connector
  ```

  把 docker 的所有 `bridge` 网络都添加到路由中

  ```shell
  docker network ls --filter driver=bridge --format "{{.ID}}" | xargs docker network inspect --format "route {{range .IPAM.Config}}{{.Subnet}}{{end}}" >> /usr/local/etc/docker-connector.conf
  ```

  >也可通过直接改配置```/usr/local/etc/docker-connector.conf```方式来修改
  >
  >```shell
  >route 172.17.0.0/16 EXPOSE #EXPOSE
  >```

  启动docker-connector

  ```shell
  sudo brew update -v
  sudo brew services start docker-connector
  ```

  这里我将docker-connector编排至docker-compose配置文件中，编辑yml文件，加入docker-connector

  ```yaml
  services:
    connector:
      image: wenjunxiao/mac-docker-connector
      container_name: connector
      network_mode: "host"
      restart: always
      cap_add:
        - NET_ADMIN
    #...
  ```

  也可使用以下命令在 docker 端运行 wenjunxiao/mac-docker-connector，需要使用 `host` 网络，并且允许 `NET_ADMIN`

  ```shell
  docker run -it -d --restart always --net host --cap-add NET_ADMIN --name connector wenjunxiao/mac-docker-connector
  ```

  

