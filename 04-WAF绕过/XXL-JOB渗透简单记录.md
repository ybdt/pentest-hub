同事已经打完，给了我一个截图，我去验证一遍  

admin   123456进入后台  
照着网上的教程尝试反弹Shell发现没反应，进一步查阅资料得知，目标服务器可能不出网，针对不出网情况，可通过日志查看结果，于是执行whoami等命令并通过调度日志查看执行结果，可怎么也验证不成功，一直如下错误：  

Cannot run program "bash": CreateProcess error=2, 系统找不到指定的文件。  

可同事给我的截图分明是linux系统啊  

抱着试试看的态度，选择了powershell，竟然成功了，还真是一切皆有可能，猜测后端是集群，有的executor是linux，有的executor是windows  

参考文章：  
https://mp.weixin.qq.com/s/t0aDztXr9KNwop_Je5ot4Q  
https://mp.weixin.qq.com/s/vUr4kLQ88coHxxLbb-ZwxA  
https://mp.weixin.qq.com/s/AKufROJaT6DLDqyykslrAg  