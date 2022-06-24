# Air-Universe-to-V2borad

>手头的机器比较多，除了正产建站之外，也顺便搭建了节点，节点多了不好管理，所以想搭建个机场面板管理自己的节点。**本文为小白学习笔记，大佬轻喷。**

###### 本文要实现的效果:

Air-Universe后端对接V2borad面板，实现Xray+vmess+TLS+WS，宝塔面板安装Nginx处理TLS，伪装站也通过宝塔面板搭建，全程可视化操作，降低安装和维护门槛，443端口实现后端节点与网站共存。

***

##### 特点：

1.  Nginx进行TLS加解密，不经Xray内核进行TLS加解密，可能能增加隐蔽性，但是流量大了什么方法都隐藏不住。
2.  Nginx部署伪装站处理TLS，增强伪装性，或许能降低被主动探测到的可能性。

* * *

##### 前期准备：

1.  如何安装v2borad面板请看[官方文档](https://www.blogger.com/blog/post/edit/1535019839350003004/3600798552222405078 "https://www.blogger.com/blog/post/edit/1535019839350003004/3600798552222405078#")，这里假设已经搭建好v2borad面板。
2.  域名解析到节点的IP，推荐Cloudflare托管域名。
3.  v2boar面板添加一个Vmess节点，记下节点ID，开启TLS，连接端口443，服务端口自定义；TLS选项内，Server Name填写节点的域名，下方Allow Insecure关闭；传输协议选项选websocket，编辑协议配置：
```shell
{
  "path": "/china/no1",  ##path 这里可自定义路径,不要填写v2flay官方文档默认的路径，会被主动探测,记住这里，后面要用到
  "headers": {
   "Host": "你的域名"
  }
}
```
下图来自网络，有文中有不同之处，请按照文中设置配置节点。

![v2bord设置](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/v2bord%E8%AE%BE%E7%BD%AE-202206231507317.jpeg)


* * *


##### Air-Universe后端对接V2borad面板：
​
1.  升级内核，开启bbr加速，用下列脚本，选36 `5.10LTS` 内核。
​
```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/install_kernel.sh && chmod +x ./install_kernel.sh && ./install_kernel.sh
```
![](https://raw.githubusercontent.com/github-office/png-hub/main/img/%E8%84%9A%E6%9C%AC%E5%9B%BE1.jpg)
​
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/2开启bbr202206231018487.jpg)
​
​
2.  centoOS内核太老，必须的升级内核，ubuntu20.04自带bbr加速，根据脚本菜单提示，选择2开启bbr加速即可，算法全部默认，根据提示可能需要重启vps。
​
​
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/3检测是否开启bbr202206231019069.jpg)
​
3.  安装宝塔面板（开启面板SSL更安全，不过小白开启面板SSL之后可能会打不开面板），之后安装Nginx，并且新建一个网站，在宝塔面板中为网站一键申请SSL证书，部署证书并强制开启https。

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/证书申请-202206231913912.jpg)


4.  使用官方推荐的第三放脚本安装Air-Universe后端 ，选择 `51`，按照提示安装。

```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/linux_install_software.sh && chmod +x ./linux_install_software.sh && ./linux_install_software.sh
```
​
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/3一键安装后端脚本202206231019189.jpg)
5.  如果检测到80、443端口被Nginx占用，选择y继续操作。
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/4安装错误提示端口占用202206231020144.jpg)
​
6.  依次输入v2borad面板的地址（https://不能省略）、token、节点ID、面板类型（选v2borad）、节点协议类型（选vmess）。

```
########Air-Universe config#######
Enter panel domain(Include https:// or http://): 
Enter panel token: token12345678token12345678
Enter node_ids, (eg 1,2,3): 
Choose panel type:
	1. SSPanel
	2. V2board
	3. Django-sspanel
Choose panel type: 

Please select node type[0-2]:
	0. VMess
	1. ShadowSocks
	2. Trojan 

```

7.  继续安装airu，根据提示选择，我这里依次选择 Air-Universe版本：不降级 使用最新版本》Xray版本： 1. 不降级 使用最新版本》请选择SSL证书申请方式：n不申请。

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/6-airu安装过程2-202206231021045.jpg)

至此，airu已经安装完毕，会弹出如下菜单。下面需要配置airu和Nginx。

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/8-airu安装完毕202206231021958.jpg)

8.  配置airu，查看Xray配置文件，记录下Xray api的监听端口。

```shell
cat /usr/local/etc/xray/config.json
```
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/记录api端口号-202206231611939.jpg)

9.  修改airu的配置文件
```
文件修改成如下配置并保存：
vi /usr/local/etc/au/au.json #如果不会vi可以直接在宝塔面板的文件管理双击打开修改，改完了别忘了保存
{
  "panel": {
   "type": "v2board",
   "url": "https://你的域名",
   "key": "你的token",
   "node_ids": [13],                       ##这里是你的节点ID
   "nodes_type": ["vmess"]
  },
  "proxy": {
   "type": "xray",
       "auto_generate": true,
       "in_tags":  ["vmess"],
       "api_address": "127.0.0.1",
       "api_port": 10085,               ##这里填写上一步的端口
       "force_close_tls": true,       ##这里填true，关闭Xray的tls功能
       "log_path": "/root/air-universe-access.log",
       "cert": {
           "cert_path": "/www/server/panel/vhost/cert/保密/fullchain.pem",
           "key_path": "/www/server/panel/vhost/cert/保密/privkey.pem"
       },
       "speed_limit_level": [0, 10, 30, 100,  300, 1000]  ##这里是限速参数，不用管
  }
}

```
10.  重启airu后端

```
airu ##选择6 重启 
```

11.  修改Nginx配置，通过 `/path`分流，实现伪装。在宝塔面板网站>设置，`error_log` 下面增加如下内容（注意缩进）并保存：
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/网站设置-202206231937808.jpg)
```
    location /china/no1 { 
      proxy_redirect off;
      proxy_pass http://127.0.0.1:44444;      ##这里填写v2borad中设置的服务端口
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_read_timeout 300s;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
```

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/nginx配置-202206231940003.jpg)

12.  访问 你的域名/china/no1 ，如果浏览器显示Bad Request，则说明配置成功

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/返回错误请求202206231945122.png)

13.  配置伪装站。此时网站还是宝塔默认的网页，为了让网站看上去更正常一点，可以随便找个网站伪装一下。宝塔面板点击网站根目录，单机全部文件，有个文件删不掉不用管。

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/删除文件202206231947878.png)

我用的是这个矩阵计算的网站用作伪装
`https://github.com/staltz/matrixmultiplication.xyz.git`
```
网站根目录下下载网站内容:
网站根目录:/www/wwwroot/你的域名/

git clone https://github.com/staltz/matrixmultiplication.xyz.git
或者直接下载了手动用宝塔面板上传就行

```
![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/下载了伪装网站202206231959713.png)

把文件夹内的文件剪贴到网站根目录

![](https://cdn.jsdelivr.net/gh/github-office/png-hub/img/page1/剪贴202206232001703.png)

刷新一下你的网站，至此完结撒花。
