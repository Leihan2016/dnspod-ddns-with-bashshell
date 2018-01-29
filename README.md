# Dnspod-DDNS-with-BashShell
利用Dnspod的api和shell脚本搭建自己的动态域名服务。  
应friends要求写的，dnspod的限制比较多，对调用次数比较小气，频次高了（API文档说的是一小时5次）就会冻结API一小时，返回API usage is limited的报错。所以本脚本进行多次对比确保减少API调用。  
CloudXns另见：https://github.com/lixuy/CloudXNS-DDNS-with-BashShell  
https://github.com/lixuy/CloudXNS-DDNS-with-PowerShell  
## 使用方法
本脚本分为两个版本，一个是获取自己外网ip的版本dnspod_ddns.sh，一个是直接获取自己网卡设备上的ip的版本dnspod_ddns_line.sh（对于多拨或者路由器网关用户适用）。
### 获取api的ID和Token
api的ID和Token可以在后台获取：  
>创建一个 Token，依次点击 用户中心 -> 安全设置 -> API Token：
![创建一个 Token](https://support.dnspod.cn/Uploads/api-tokens-1.png)
>点击创建一个 Token，输入 Token 名称即可，名称仅用来标记 Token，方便用户管理 Token ，不参与鉴权。
![输入 Token 名称](https://support.dnspod.cn/Uploads/api-tokens-2.png)     
>点击 “确定” 之后，Token 创建成功，会弹出如下提示框：
![Token 创建成功](https://support.dnspod.cn/Uploads/api-tokens-3.png)
### dnspod_ddns.sh
#### 参数说明  
参数|填写说明
:-:|:-
|API_ID | 在个人中心后台的安全设置里面获取ID|
API_Token|在个人中心后台的安全设置里面获取Token
domain| 你所注册的主域名，例如baidu.com，qq.com，china.edu.cn，example.com
host|主机记录名，例如www.baidu.com的主机记录名是www，image.www.weibo.com的主机记录是image.www，home.example.com的主机记录名是home
Email|填写你的邮箱。（根据API规范要求）
CHECKURL|用于检查自己的外网IP是什么的网址，注释掉该参数会跳过所有检查（仅验证域名记录是否存在）直接执行更新记录（会导致高频率调用更新）；建议的备选CHECKURL：```http://ip.3322.org``` ```http://myip.ipip.net``` ```http://ip.xdty.org```
OUT|指定使用某个网卡设备进行联网通讯（默认被注释掉）。注意，一些系统的负载均衡功能可能会导致该参数无效。推荐使用```ip a```命令查看网卡设备名称。
#### 推荐的部署方法
把如上所述的参数填好即可。  
本脚本没有自带循环，因为linux平台几乎都有Crontab（计划任务），利用计划任务可以实现开机启动循环执行脚本并设定循环频率。  
>**命令参考**  
新建计划任务输入```crontab -e```  
按a进入编辑模式，输入   
 ```*/1 * * * * /root/dnspod_ddns.sh &> /dev/null```  
意思是每隔一分钟执行/root/dnspod_ddns.sh并屏蔽输出日志。当然，如果你需要记录日志可以直接重定向至保存路径。 
然后按Esc，输入:wq回车保存退出即可。  
更多关于Crontab的使用方法此处不再详述。  
另外对于一些带有Web管理界面嵌入式系统（比如openwrt），有图形化的计划任务菜单管理，可以直接把脚本粘贴进去。

#### 工作过程
1、用CHECKURL检查自己的外网ip和本地解析记录是否相同，相同则退出；  
2、使用API获取域名在Dnspod平台的ip记录，如果ip记录和本地解析记录或者外网解析记录相同则退出；获取记录异常也会退出并返回错误信息（例如域名不存在No Record）；  
3、执行DNS更新，并返回执行结果。
#### 注意事项
本脚本**不会**自动创建子域名，请务必先到后台添加一个随意的子域名A记录，否则会提示No Record；  
如果你看到**API usage is limited**的报错，是由于调用API频率过高账号被冻结（一小时后解封）

### dnspod_ddns_line.sh
仅说明与上面脚本参数不同的地方。  
因该脚本是用于获取网卡设备ip，所以没有CHECKURL参数。  
#### 参数说明  
参数|填写说明
:-:|:-
|DEV | 从网卡设备（例如eht0）上获取ip，并与DNS记录比对更新|

### 关于
https://03k.org/dnspod-ddns-with-bashshell.html

