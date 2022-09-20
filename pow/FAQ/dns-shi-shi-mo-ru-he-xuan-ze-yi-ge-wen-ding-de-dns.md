# DNS是什么、如何选择一个稳定的DNS？

**最近一年，不断有用户反馈挖矿地址解析失败导致无法正常挖矿的情况，经过排查出现问题的用户均使用了114.114.114.114为DNS。**

**挖矿的数据传输以网络作为基础，同时DNS在矿池与矿机的交互扮演了重要的角色，那么选择一个稳定且安全的DNS则显得尤为重要。**

**114.114.114.114目前其官网已经停止维护，除此之外，使用114DNS还会出现以下问题：**

\-不稳定，导致客户矿机不断重连挖矿地址，算力受到严重影响，折损机器寿命，增加矿场运维压力。

\-技术落后，服务器不能负载更高的DNS解析访问量带来的压力，易被攻击、更新较慢导致打不开网页。

\-无法快速同步最新的解析记录和智能的解析需求，矿机无法连接最优矿池节点，矿机连接不稳定，拒绝率升高，算力不足。

\-占用网络带宽为用户推送广告，且用户不易察觉。

**为避免矿工因为114DNS问题，而造成挖矿的影响，推荐您使用如下DNS：（如需直接查看修改步骤，请直接跳至文章步骤四）**

**1、国内用户（以下DNS任选其一即可）**

**119.29.29.29（腾讯DNS）119.28.28.28（腾讯DNS）**

推荐原因：

1.速度快。2.解析准确。3.用户基数大。

**223.5.5.5（阿里云DNS）223.6.6.6（阿里云DNS）**

推荐原因：

1.国内稳定。2.响应及时。3.传输稳定。4.出口优化。

**2、海外用户（以下DNS任选其一即可）**

**1.1.1.1（Cloudflare DNS）**

1.速度快。2.保护隐私。3.支持加密。

**8.8.8.8（Google Public DNS）**

1.安全稳定。2.速度快。

### **一、什么是域名？** <a href="#xoeu4" id="xoeu4"></a>

平常我们访问的www.dns.com这些地址就叫做域名。

### **二、什么是DNS？** <a href="#wijfm" id="wijfm"></a>

域名系统（Domain Name System缩写DNS）是因特网的一项核心服务，它作为可以将域名和IP地址相互映射的一个分布式数据库，能够使人更方便的访问互联网。大多数因特网服务依赖于 DNS 而工作，一旦 DNS 出错，就无法连接 Web 站点。

为了浏览网站，访问本网络之外的机器，必须要使用IP地址。通过DNS协议，我们可以建立域名和IP地址的一个映射关系。DNS协议能够帮助我们将域名解析为IP地址，而不用记住那些复杂的数字就可以上网。

例：

某网站的名称为www.zhenhao.com

其IP地址为192.168.176.19

DNS的作用即让我们记住zhenhao.com即可访问至192.168.176.19。

### **三、如何查看使用的DNS？** <a href="#xcvyz" id="xcvyz"></a>

**查看使用的DNS可以使用以下3种方法，分别为命令提示符工具、电脑网络设置及矿机后台查询。**

1.“开始”—“运行”—”输入CMD“，打开命令行窗口输入 nslookup，按回车。Address后的一串数字即为DNS。

![](<../../.gitbook/assets/image (183).png>)

2.“设置”—“网络和Internet”—“WLAN”/”以太网“—”网络和共享中心—“连接”—“详细信息”

![](<../../.gitbook/assets/image (188).png>)

3.矿机后台Network中，查看DNS Servers。

![](<../../.gitbook/assets/image (194).png>)

### **四、如何修改DNS？** <a href="#yn3cs" id="yn3cs"></a>

**修改DNS可以有如下3种方法，分别为路由器修改、电脑修改和矿机后台修改。**

**1.路由器修改（以小米路由器为例）**

（1）登录小米路由器网页版，点击“常用设置”。

![](<../../.gitbook/assets/image (148).png>)

（2）点击“上网设置”

![](<../../.gitbook/assets/image (179).png>)

（3）“手动配置DNS”选项

![](<../../.gitbook/assets/image (104) (1).png>)

\*其他路由器界面略有不同，找到“手动配置DNS”选项进行设置即可。

\*如果您修改路由器DNS，矿机为DHCP模式，则会自动分配。若为Static模式，则需要手动至矿机后台修改。

例：

矿机后台分为2种IP模式。（默认为DHCP模式）

* DHCP：俗称“动态IP”，自动获取IP地址，但矿机的IP会改变，修改路由器DNS后，将会自动为矿机分配IP。
* Static：俗称“静态IP”，手动设置IP地址，矿机的IP不会改变，修改路由器DNS后，矿机IP并不会随DNS的改变分配而分配。**也就是说如果矿机是Static，无论您是否修改DNS，矿机都不会做出任何改变。**

**2.电脑修改（以WLAN为例）**

“设置”—“网络和Internet”—“WLAN”/”以太网“—”网络和共享中心—“连接”（点击正在链接的网络）—“属性”—“Internet协议版本4（TCP/IPv4）”—“属性”—使用下面的DNS服务器地址“—填写推荐的DNS地址即可

![](<../../.gitbook/assets/image (160).png>)

**3.矿机修改（以蚂蚁机型为例）**

矿机IP分为2种。

* DHCP：俗称“动态IP”，自动获取IP地址，但矿机的IP会改变，不好定位问题机器
* Static：俗称“静态IP”，手动设置IP地址，矿机的IP不会改变，容易定位问题机器
* （矿机默认为DHCP模式）

（1）进入矿机后台，点击Network。

![](<../../.gitbook/assets/image (127).png>)

（2）点击DHCP，选择Static。

![](<../../.gitbook/assets/image (115).png>)

（3）填写矿机的网络信息。

![](<../../.gitbook/assets/image (166).png>)

修改时矿机后台会有默认IP、子网掩码、网关和DNS，修改DNS为**推荐**即可，也可参考电脑网络连接详细信息。（“设置”—“网络和Internet”—“WLAN”/”以太网“—”网络和共享中心—“连接”—“详细信息”）

![](<../../.gitbook/assets/image (102) (1).png>)

（4）点击”Save\&Apply“。

![](<../../.gitbook/assets/image (126).png>)