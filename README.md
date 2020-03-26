# 面试经历分享和题目环境复现

笔者最近参加了很多面试，想通过这个项目将面试的经历和面试官提出的问题分享一下，同时复现一部分面试官的题目环境。这里有很多漏洞环境在网上都存在了很多，所以在这里也没有过多的再去自个复现，于是就将前辈们的成果结果面试题目整理了一下。特别感谢vulhub提供的众多渗透测试环境，有兴趣的伙伴们还可以去大佬的项目下膜拜https://github.com/vulhub/vulhub

## 经典不变的场景问题：给你一个后台登陆站点，说说你的渗透思路。

此类问题一般围绕渗透测试的基本流程展开，不过个人有其他思路也可以，这里笔者主要是为了可以做到条例清晰，让面试官觉得很有逻辑性。当然回答过程中还有面试官提出的其他问题。

### 信息搜集

#### whois站长查询

查注册人信息，邮箱。主要是可以拿着这些信息通过goole，或github搜索一些其他的敏感信息，扩大搜索面。效果就不多说了，在github泄漏一些账号或源码的事件简直不要太多。

#### 查子域名

这里不仅仅是只针对这个场景的渗透了。当然有可能子域名下的数据库和正式的用的一样呢，例如测试站点，开发者为了方便利用数据测试，自然而然的采用了正式数据库（这个我遇到过），测试站点的问题理论上比正式上线的会多一些，拿到权限的可能也就越大，这是再到正式的登陆接口测试。（扩散思维）

#### 服务器相关信息（真实IP，系统类型、waf、开放端口）

问：获取获取真实ip方法，

ping方法去ping域名，

问：如何判断目标站点使用了CDN，如果目标站点使用了CDN，如何绕过判断真实ip？

判断：有条件的使用全国不同地区的服务器ping这个网址，（当然线上也有工具！）如果得到的ip结果不同，即可判断使用了CDN。

绕过：查询域名解析记录；使用国外服务器ping，因为国内很少对国外做CDN。当然还可以使用nslookup，dns服务器地址尽量找冷门的，热门的会和国内查询的结果相同。

Nmap 扫描，探测当前服务器是否开放了其他端口，如果其他端口存在历史遗留漏洞，可以直接利用。nmap扫描服务器进行搜集，我认为也是至关重要的一点，不能遗漏。

问：nmap默认以什么方式探测

ping方式

问：如何使用禁ping的方式扫描

-p0



### 漏洞挖掘
对应下面的一些列渗透环境请点击https://github.com/yaunsky/Iproject/tree/master/%E6%BC%8F%E6%B4%9E%E6%8C%96%E6%8E%98/

一个登陆站点可能存在的漏洞：sql注入、万能密码（sql注入）、弱口令，敏感信息泄漏，csrf，url跳转，xss，命令执行，越权访问，未授权访问，短信轰炸

#### sql注入

问：注入类别

get、post、cookie、盲注

sqlmap的使用方法

问：获得当前数据库使用的用户

—current-user

问：post注入方法、cookie注入方法。

抓包，放到txt文件，使用sqlmap -r *.txt实现post注入

cookie注入，--cookie .. —level 2

使用文件时，制定某特定选项注入的方法

在文件中，将指定的参数值设为星号（*）

risk级别、level级别

risk 1-3

level 1-5

默认都是1.测试cookie注入时最低level为2

#### 文件上传

绕过的方式有哪些

前端验证绕过，

#### 网站指纹识别,中间件识别

1、常见的cms漏洞

2、weblogic、nginx、iis、apache

问：weblogic你知道那些漏洞

类似问题尽量答新一点的或经典的（我认为这样面试官会觉得你关注最新的安全事件），任意文件上传。

又问：有哪些限制呢？

用户名和密码

设置web测试

问：谈一谈nginx、iis和apache解析漏洞

nginx解析：从右往左解析

iis解析漏洞： 通过；绕过

apache解析：将jpg文件当作执行文件解析

### 提权或拿shell

#### mysql写shell

问：拥有数据库的访问权限如何写shell

union select 写入

问：还有其他的吗？

不会，肯定是有的

问：如果你拥有一个sa用户，如何写shell

1、通过sa权限回复xp_cmdshell

2、查网站真实路径

3、写入shell

问：sql server和oracal默认端口

1433 1521 mysql3306

问：phpmyadmin下如何写shell

1、查询是否开启日志记录，没有则开启

show global variables like "genera";

2、设置日志保存地址（前提得到网站真实路径）

3、select "shell"这样就写进了日志，菜刀连日志文件即可

或者

1、创建一个表，设置一个字段

2、将字段内容为一句话

3、查询表的内容，倒入到php文件

4、菜刀连接php文件

或者

1、直接通过select 一句话 into outfile "PHP文件路径"

2、连接一句话

问：mysql获得shell的时候，有哪些限制条件

GOP关闭

Secure-file-priv没有配置

root权限

有绝对路径

#### udf提权

利用root权限，创建带有调用cmd函数的udf.dll文件，然后可以直接输入cmd

linux提权方式

1、利用内核漏洞提权

脏牛

2、sudo提权

3、数据库提权

4、suid提权

#### ssrf结合redis获得shell

使用ssrf探测内网是否开放了6379端口（对应redis服务），如果开放，即可结合redis未授权漏洞，通过gopher协议写入shell。

又问：redis未授权都有哪几种方式获得shell？

1、直接写shell到网站的真实路径下，当然必须知道网站真实目录

2、将密钥写入.ssh/目录下

3、写计划任务

又问：执行写shell的操作时，应该具备哪些条件。

1、未授权访问

2、redis命令没有禁止，例如config

禁用命令的使用去找redis.conf。里面的security项

rename-command CONFIG ""

又问：如果内容禁止使用ip如何探测内网端口

1、使用dns解析

2、127。0。0。1

3、http://[::]:80/

4、进制转换

### 权限维持

#### windows方法

1、shift后门

将自己的cmd.exe替换掉C'\windows\system32\dllcache\sethc.exe

在开机后未登陆情况下按5下shift执行后门。

2、计划任务

没研究过

#### linux方法

1、ssh后门

2、suid后门

3、inetd服务后门

4、vim后门

#### MSF权限维持

1、persistence模块

2、metsvc模块

#### powershell权限维持

资料查的，暂未研究，不过面试官有时侯也会挺喜欢问这些。

## 挖洞过程中，你觉得挖到最有意思的漏洞

阿里积分活动

ueditor的ssrf漏洞



## 内网渗透知识

### 域控

域控hash文件存放位置

C:\Windows\NTDS\NTDS.dit

### 内网信息搜集

查看域：new user /domain

看共享：net view

看hosts文件：可能通过这个文件进行ip-域名解析

浏览器中记录的密码

敏感文件

服务器中的配置文件

历史命令

ssh私钥

xshell中的session文件，可导出

工具扫描：msf ms17010

## 应急响应

应急如何查找挖矿

Netstat -anp查异常pid看端口信息，然后根据端口信息定位文件 ls -l /proc/PID

服务器入侵报警的怎么做？

自写的脚本，实现的功能：查登陆日志（成功与失败）、查规定时间内修改的文件、查最新添加的文件、查历史命令，查异常进程、查/etc/passwd、查各服务的最新日志（每天的日志都会备份一份到指定文件夹下）。
