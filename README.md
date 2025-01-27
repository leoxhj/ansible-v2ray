# V2Ray for Ansible

本 Playbook 将利用 Ansible 在服务器上自动化部署 V2Ray。目前，仅在 Debian
GNU/Linux 10 上测试通过。Debian GNU/Linux 更低版本或 Ubuntu
系列大概也没有问题，其它 Linux
发行版本有待进一步测试。使用本仓库请自行承担风险。

根据 V2Ray 大多数用户推荐，本 Playbook 采用 [WebSocket+TLS+Web](https://toutyrater.github.io/advanced/wss_and_web.html) 部署方式。换句话说，它将在服务器上安装下列软件：

* Certbot：用于请求 SSL 证书
* NGINX：用于提供 Web 服务
* V2Ray

## 使用要求

为了正常使用 V2Ray，应当满足以下要求：

* 一台云主机VPS，如 [Vultr](https://www.vultr.com/?ref=7599369)等等，我自己用的是3.5美金一个月的VPS。
```bash
Deploy New instance: ($3.50/month)
* Choose server: Cloud Compute
* CPU & Storage Technology: Regular Performance
* Server Location: America -> New York(NJ)
* Server Image: CentOS 7 x64
* Server Size: 10GB SSD 
Turn off auto backups, otherwise the price is $4.2/month
```
* 一个域名(比如Google domains)且已经绑定到已经部署好的云主机的IP地址。
* 推荐使用centos7，测试ok，机器安装完后
  - 注意关闭firewalld服务，不然会有问题 `systemctl disable firewalld && systemctl stop firewalld`
  - `setsebool -P httpd_can_network_connect 1`
* Ansible，可参考[官方文档](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-node)安装。
* 修改roles/v2ray/templates/config.json.j2中的域名为你已经绑定DNS的域名
* 设置本机到云主机VPS的SSH远程免密登录，把本地ssh的is_ras.pub上传到VPS的authorized_keys file (permission 600)，并测试本地ssh能免密登录成功

### 使用方法

1. 将服务器的 IP 添加到 `inventory/selfhost` 文件：

        $ echo 'your.ip.address' >> inventory/selfhost

2. 安装ansible插件
 
        $ ansible-galaxy collection install ansible.posix
 

2. 执行 Playbook 并按提示操作：

        $ ansible-playbook v2ray.yml

3. Enjoy.

### Issues: v2ray client connection error "v2ray handshake timeout"
Nginx Error Log:

```
2023/02/27 11:49:31 [crit] 4137#4137: *1812 connect() to 127.0.0.1:1080 failed (13: Permission denied) while connecting to upstream, client: 116.230.163.217, server: jump.sunlandgreen.com, request: "GET /ws2/ HTTP/1.1", upstream: "http://127.0.0.1:1080/ws2/", host: "jump.sunlandgreen.com"
```

Solution: 
```bash
setsebool -P httpd_can_network_connect 1
```
