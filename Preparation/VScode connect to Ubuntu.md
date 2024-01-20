# 使用VScode连接虚拟机

## 1.下载需要的插件

1. Remote-SSH
2. C/C++ Extension Pack
3. Live Preview
4. Live Server
5. Remote Development

## 2.检查主机与虚拟机能否互通
* 打开终端输入`ifconfig`，如果出现报错可能是没有安装`ifconfig`，输入相关的命令进行安装即可。
* 输入后效果如下：其中标注处即为虚拟机的IP地址。![image-20240120204654307](https://github.com/Fengtao4428/MIT-6.S081/assets/88192248/d9e6783a-71b5-4725-8db2-faca5e76b644)
* 在主机(windows)的终端中输入命令`ipconfig`，效果如下：IPV4地址即为主机的IP地址。![image-20240120204834585](https://github.com/Fengtao4428/MIT-6.S081/assets/88192248/b819b2e2-d2b4-4a81-8b85-b70a3c11fa83)
* 能够互相ping通就准备好了。

## 3.使用VScode连接虚拟机

1. 使用VScode，打开下载的插件Remote Development![image-20240120205136588](https://github.com/Fengtao4428/MIT-6.S081/assets/88192248/5fae5922-7cc7-4808-95e5-aee15b0a7b77)
2. 在窗口中输入：`ssh user@ip`进行连接![image-20240120205219839](https://github.com/Fengtao4428/MIT-6.S081/assets/88192248/a441bb9a-cb9d-4313-b032-fcf1cf42c717)
3. 最后输入虚拟机的密码，选择Linux即可。
