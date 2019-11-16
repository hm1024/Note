# Linux 中关于防火墙

## Linux 中为某一个端口开启防火墙

首先打开防火墙配置文件

`vim /etc/sysconfig/iptables`

![Snipaste_2019-11-16_16-26-59.jpg](https://i.loli.net/2019/11/16/FLuphsKrnm62Wfc.jpg)

复制图中标红的哪一行，（在正常模式下按 `yy` ,然后按 `p`即可复制）

然后把 `22`改为相应的端口，

然后保存，重启防火墙 `service iptables restart`