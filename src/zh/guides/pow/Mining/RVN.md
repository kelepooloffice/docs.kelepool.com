
*本指南针对通过注册矿池账户进行挖矿的用户，如需使用链上地址进行挖矿，请参考* [匿名挖矿教程](../anonymous.md)。


# 🪙 RVN挖矿教程

### 挖矿地址

**URL1：** stratum+tcp://rvn.kelepool.com:80

### 支付计划

结算方式：PPLNS       费率：1.00%      起付额：10.00RVN  

收益账单于次日早8点生成，**如果余额达到支付门槛，链上支付**将在次日完成。

## 1.注册一个可乐矿池账号

注册及设置收款地址流程参考[新手入门](../../)，注册完成记录下子账户名称备用；

**网站地址（需VPN开全局代理）：** [**`https://www.kelepool.com/`**](https://www.kelepool.com/)

## 2.使用矿梯

可乐矿池推荐国内用户使用矿梯代理线路，更加隐蔽安全，更加高效，查看[矿梯使用教程](../ladder.md)

## **3. 配置挖矿设备(不使用矿梯)**

> **GPU(NVIDIA或AMD)**

### **a. 常用软件**

**常用RVN挖矿软件：**

* AceMiner：[**`https://www.aceminer.io/`**](https://www.aceminer.io/)**``**
* Hiveon：[**`https://hiveon.com/`**](https://hiveon.com/)**``**
* 开源矿工：[**`http://dl.ntminer.top/`**](http://dl.ntminer.top/)**``**
* MinerOS：[**`https://mineros.info/`**](https://mineros.info/)**``**

**常用RVN挖矿原创内核：**

* PhoenixMiner：[**`https://phoenixminer.org/download/`**](https://phoenixminer.org/download/)**``**
* Bminer：[**`https://www.bminer.me/zh/releases/`**](https://www.bminer.me/zh/releases/)**``**
* NBminer：[**`https://github.com/NebuTech/NBMiner/releases`**](https://github.com/NebuTech/NBMiner/releases)**``**
* DamoMiner：[**`http://www.damominer.com/`**](http://www.damominer.com/)**``**

### **b. 常用RVN挖矿软件设置**

可以根据各软件官方教程进行设置，如HiveOS需建立RVN币种 的飞行表，自定义矿池设为可乐矿池，填写用户的矿池子账户/用户名，应用到矿机即可。

### **c. 常用原创内核挖矿设置**

通过上述途径获取内核挖矿软件，选择“将文件解压缩到当前文件夹”，找到或新建一个批处理文件。 右键点击此批处理文件，选择“编辑”，各项参数设置如下：


> **Bminer：** bminer -uri raven://username.worker_name@rvn.kelepool.com:80
>
> **NBminer：** nbminer -a kawpow -o stratum+tcp://rvn.kelepool.com:80 -u username.worker\_name
>
> **Gminer：** miner.exe --algo kawpow --server rvn.kelepool.com:80 --user username.worker\_name

其中“**rvn.kelepool.com:80”** 为可乐矿池地址，其他可选RVN挖矿地址为

**“username”** 要替换成你的矿池注册用户名（若是匿名挖矿则为链上地址）；

**“worker\_name”** 为矿机编号，自定义即可如编号或字母组合（不能有汉字或特殊字符）；

修改时只把对应的字符修改掉，空格、标点均不要增减。

核实所填信息无误后，保存并退出此批处理文件，然后双击运行此修改过的批处理文件即开始挖矿。

## 4.ASIC矿机设置(不使用矿梯)

### **a. 进入矿机设置**

启动矿机，将矿机与电脑置于同一网络，然后通过矿机IP查找软件获取矿机IP，在电脑浏览器中输入矿机IP进行设置，或使用各类挖矿软件完成设置，具体流程参照各矿机官方给出的教程。

### b. 填写挖矿地址

**挖矿地址即对应矿池的矿池地址**，可前往**可乐矿池-矿池挖矿-总览右下角（左上角切换币种）**查看最新地址，需注意的是可能因为杀毒软件/防火墙等原因会导致地址无法显示，关闭杀毒软件/防火墙即可，可乐矿池目前**ETC地址如下**（不同币种地址不一样）：

**通用挖矿地址：**

* **URL1：stratum+tcp://rvn.kelepool.com:80**

### c. **填写**矿工名

矿工名填写模式为：**矿池里的子账号/用户名.编号**

子账户/用户名即子账户/用户名即“矿池挖矿-切换账户”中显示的挖矿账户名，编号是区分矿机的编号，可按照自己需要自由编写，如有多台矿机，可以子账户名XXX为开头按XXX.01，XXX.02这个规律给机器编号方便管理。

### d. 注意事项

* 是否使用矿梯作为代理的区别主要体现在矿机上设置时是填写可乐矿池地址还是填写矿梯地址；
* 因为矿池的挖矿地址可能出现问题，一般一个矿池会提供多个挖矿地址做备用；
* 同一台矿机在填写了同一个矿池的多个挖矿地址时，使用同一个矿工名（也就是同一个子账户/用户名.编号）；
* 同一台矿机若设置了不同矿池的挖矿地址，则矿工名要填写对应矿池的子账户/用户名；
* 多台矿机在设置矿工名时应在编号部分进行区分，如填写成同样的矿工名则只能在矿池里看到一台，但真实算力不受影响，仍旧是多台矿机的叠加；
* **挖矿地址和矿工名的填写非常重要**，如填错了则可能导致收益受损，若不确定请联系客服。

## 5.添加收款信息

**1) 全节点钱包**

以太坊经典全节点钱包占用硬盘空间较大，如有需要可自行搜索下载。

**2) 交易所钱包**

[币安](https://www.binance.com/cn)、[OKEx](https://www.okex.com/)等。大部分交易所都支持ETC充值和交易，注册交易所并找到ETC充值即可获取钱包地址。

**3) App钱包：**

也可选用[Cobo](https://cobo.com/)等App钱包。

获取到钱包地址后，请在账户中绑定钱包地址。登录可乐矿池帐户，并在**账户配置-收款设置**中按照提示完成钱包地址绑定。

## 6.验证连接成功

设置完成后，可以稍等一会儿，再通过以下方式确认矿梯和矿机是否连接成功正常工作：

1、查看挖矿软件的日志或活动面板，确认矿机正常工作且算力显示正常&#x20;

2、前往可乐矿池的总览页面查看活跃矿机数量和算力是否正常，预估收益可能出现偏差，以第二天实际结算的收益为准。

![](<../../.gitbook/assets/image(303).png>)