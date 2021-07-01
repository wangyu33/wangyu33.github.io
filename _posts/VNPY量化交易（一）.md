# VNPY量化交易（一）

## 一、平台选取与环境搭建

​	交易平台选取为VNPY，VNPY框架主要基于c++与python3进行开发，因此，VNPY可以支持tensorflow等深度学习框架的使用，方便后续构建深度学习量化模型。

​	接下来简单介绍下VNPY的几个名词:

- VN Studio：相当于针对vn.py量化框架的Python发行版，类似于Anconda是用于科学计算，数据分析的Pyhon发行版，好处就是可以省去了手动安装vn.py所依赖的库的步骤，并且可以与最新版进行同步。
- VN Station：用于管理VN Trader以及其他Python量化交易应用的图形化管理工具，相当于一个集成很多量化App的软件，同时也是vn.py进行开发的IDE。
- VN Trader：vn.py框架中的开箱即用专业量化交易平台，灵活加载各类交易接口（期货、股票、期权、外汇、数字货币），支持诸多量化交易用（CTA策略、算法交易、脚本策略、行情录制、RPC服务等等

​	笔者主要是基于window进行策略回测，因此选取带有图形界面的进行策略开发。接下来首先是进行VN station的下载，打开VNPY官网<https://www.vnpy.com/>，点击安装2.1.5进行下载。![量化交易1](C:\Users\wangyu\Desktop\博客\量化交易1.png)

​	该应用的环境需要保证python的版本大于等于3.7否则会出现安装后应用无法打开的情况。如果使用的是anaconda平台则可以使用如下命令进行更新：

```
# 升级conda
conda update conda
# 升级anconda
conda update anaconda
# 升级python
conda update python
```

​	首次登录需要到[vn.py社区](https://www.vnpy.com/)进行申请账号，然后运行vn station进行登录。

![量化交易2](C:\Users\wangyu\Desktop\博客\量化交易2.png)

​	便可以进入vn station界面，简单介绍VN station底部几个bar功能。

![量化交易3](C:\Users\wangyu\Desktop\博客\量化交易3.png)

- VN Trader Lite：一键启动针对国内期货CTA策略的轻量版VN Trader
- VN Trader Pro：支持灵活配置加载交易接口和策略模块的专业版VN Trader
- 提问求助：打开浏览器访问社区论坛的“提问求助”板块，掉坑了快速提问，这个模块是真的很nice，vn.py的创始人陈晓优大佬经常会亲自解答。
- 更新：傻瓜式更新vn.py和VN Station，按钮平时点不了，只在有更新时才会亮起，通过这个可以及时获取最新上线的功能。
- Jupyter Notebook：启动Jupyter Notebook交互式研究环境。



## 二、不同测试平台配置

### 2.1 国内期货CTP配置教程

​	模拟账户申请，CTP模拟账号可以去[SimNow官网](http://www.simnow.com.cn/)进行申请，账户申请需要在日内交易时间申请，否则会出现申请不成功的情况。

注册完成后，在SimNow首页的“投资者登录”即可看到账号信息：

![量化交易4](C:\Users\wangyu\Desktop\博客\量化交易4.png)

​	然后点击常用下载中的v3终端下载来获取其交易的服务器地址：

![量化交易5](C:\Users\wangyu\Desktop\博客\量化交易5.png)

​	下载安装后打开，点击测速获取服务器，第一次登录使用需要修改密码：![量化交易6](C:\Users\wangyu\Desktop\博客\量化交易6.png)

共计有三组服务器：

1. 交易服务器180.168.146.187：10101

   行情服务器180.168.146.187：10111

2. 交易服务器218.202.237.33：10102

   行情服务器218.202.237.33：10112

3. 交易服务器180.168.146.187：10130

   行情服务器180.168.146.187：10131

​	前两个服务器在日内交易时间登录，第三个服务器用于非交易时间登录，来获取行情信息。

最后是CTP接口配置，点击VN Trader Pro，选取CTP：

![量化交易7](C:\Users\wangyu\Desktop\博客\量化交易7.png)

选取第一个进行登录，（CTP与CTP测试只能选取一个），进入界面后选择左上角连接CTP，如图：

![量化交易8](C:\Users\wangyu\Desktop\博客\量化交易8.png)

上面各个字段的填写如下：

用户名：SimNow的investorId
密码：SimNow的登陆密码
经纪商代码：9999
交易服务器：180.168.146.187:10101
行情服务器：180.168.146.187:10111
产品名称：simnow_client_test
授权编码：0000000000000000（16个0）
产品信息：留空不用填

通过日志便可以查看登录信息。



### 2.2 火币实盘

​	点击API管理，然后创建API，获取自己的API密钥和密钥密码，然后保存：

![量化交易9](C:\Users\wangyu\Desktop\博客\量化交易9.png)

​	点击VN Trader Pro，勾选火币，点击进入，再点击左上角的huobi连接：

![量化交易10](C:\Users\wangyu\Desktop\博客\量化交易10.png)

​	输入对应的字段，点击连接前需要使用小飞机进行连接登录，否则会出现连接不成功的情况，笔者使用过sstap以及v2rayN-Core均出现登录失败的情况。

### 2.3 大A证券XTP

笔者使用的平台是中泰XTP，点击进入[中泰XTP主页](https://xtp.zts.com.cn/)点击注册，![量化交易11](C:\Users\wangyu\Desktop\博客\量化交易11.png)

​	根据实际情况进行编写，提交申请后一个工作日内便可以收到申请的邮件。

​	接下来是登录，同样是进入VN Trader Pro勾选中泰XTP，点击左上方的XTP连接，按照邮件信息填写，注意行情服务器和交易服务器不要填反。

![量化交易12](C:\Users\wangyu\Desktop\博客\量化交易12.png)

​	日志出现以下信息便说明登录成功。



这样便可以开展愉快♂的交易之旅了。