# 使用mcrcon定时向服务器所有玩家播报全体消息

## 概述

使用mcrcon以及rconMessager该脚本实现

## 前置要求

确保服务器已启用rcon,具体参考官方[Server Admin Guide](https://sandstorm-support.newworldinteractive.com/hc/en-us/articles/360049211072-Server-Admin-Guide)

## 安装mcrcon

从仓库拉取mcrcon

```bash
$git clone https://github.com/Tiiffi/mcrcon.git
```

使用gcc编译(没有请先安装,sudo apt install gcc)

```bash
$cd mcrcon
$gcc -std=gnu11 -pedantic -Wall -Wextra -O2 -s -o mcrcon mcrcon.c
```

将以下两文件移至以下两个目录

```bash
sudo mv mcrcon /usr/local/bin
sudo mv mcrcon.1 /usr/local/share/man/man1
```

完成后输入

```bash
mcrcon -h
```

并得到以下输出说明mcrcon已可正常使用

```bash
Usage: mcrcon [OPTIONS] [COMMANDS]

Send rcon commands to Minecraft server.

Options:
  -H            Server address (default: localhost)
  -P            Port (default: 25575)
  -p            Rcon password
  -t            Terminal mode
  -s            Silent mode
  -c            Disable colors
  -r            Output raw packets
  -w            Wait for specified duration (seconds) between each command (1 - 600s)
  -h            Print usage
  -v            Version information

Server address, port and password can be set with following environment variables:
  MCRCON_HOST
  MCRCON_PORT
  MCRCON_PASS

- mcrcon will start in terminal mode if no commands are given
- Command-line options will override environment variables
- Rcon commands with spaces must be enclosed in quotes

Example:
        mcrcon -H my.minecraft.server -p password -w 5 "say Server is restarting!" save-all stop

```

举个例子

```
$mcrcon -H 你的主机IP -P 你的rcon端口 -p 你的rcon密码 "listplayers" save-all stop
将会向服务器发送listplayers命令，列出当前服务器所有玩家信息
(当然，我们肯定是要在服务器同一台主机上使用脚本，那么-H参数 就不需要了,-H参数默认值是localhost,
故
mcrcon -P 你的rcon端口 -p 你的rcon密码 "listplayers" save-all stop
)
```

## 使用rconMessager脚本

从仓库拉取

```bash
$git clone https://github.com/kentone/rconMessager/blob/master/rconMessager.sh
```

```bash
cd rconMessager
```

首先编辑rconMessager.sh

```sh
#!/bin/bash
#把你的家目录放到下面这行的HOME后面(/home/用户名)
export HOME=/home/用户名
#把你的全体消息存储文本路径放入(比如 /home/用户名/rconMessager/Adverts.txt,Adverts.txt就是存储我们写好的全体消息的文本)
FILE=/home/用户名/rconMessager/Adverts.txt

#RCON 配置
##################################
#Ip地址,在沙暴服务器本机运行脚本就直接输localhost
HOST=localhost
#Rcon密码
PASSWORD=密码
#Rcon端口号
PORT=端口号
#MCRCON命令行(不要更改下面)
COMMAND="mcrcon -H $HOST -P $PORT -p $PASSWORD"

#FTP Config 
#如果你的服务器不允许执行脚本,你可以在自己的主机上运行该脚本然后从游戏服务器下载adverts文件来编辑
#如果你在游戏服务器本机执行脚本，那么后面这些行都别动就完事,只需要更改下后面的消息发送时间间隔，脚本就编辑完成了
##################################
#Put a 1 to enable FTP fetching, put a 0 to disable
ENABLEFTP=0
#FTP user
FTPUSER=
#FTP password
FTPPASS=
#FTP PORT (Fill this even if it uses the default port)
FTPPORT=
#This is the file that will be downloaded from the FTP server (ex. /directory1/directory2/Adverts.txt)
FTPFILE=
#消息发送时间间隔,每多少秒发送一条消息
INTERVAL=90
#Initial value for counter
i=1

#后面的全都不要动!脚本到这里就编辑完毕了
#The main loop
while true
do
	#Retrieves the Adverts.txt so we can change it from the server
	if [ $ENABLEFTP = 1 ]
	then
	wget --timeout 10 -t 3 ftp://"$FTPUSER":"$FTPPASS"@"$HOST":"FTPPORT""$FTPFILE" -O "$FILE"
	fi
	#refreshes the number of lines on every main loop iteration.
	FILELINES=$(wc -l < $FILE)
	#Counter loop, sends every line of the Adverts.txt one by one to the server via rcon
	until [ $i -gt $FILELINES ]
	do
		#I use cut with the counter to get every line one by one on every iteration, until it reaches maximum number of lines and resets the counter to the first one
		echo "RCONING!!!!!"
		$COMMAND "say $(cut -f$i -d$'\n' $FILE)"
        #Sums one to the counter
		i=$(( i+1 ))
		sleep $INTERVAL
	done
	#Resets the counter when the counter loops reaches $FILELINES
	i=1
done
```

然后编辑你的全体消息存储文本(之前脚本里定义的这个FILE=/home/用户名/rconMessager/Adverts.txt),比如默认的是Adverts.txt

每一行一条消息，往里面写就完事，比如

```
[Server酱]欢迎进入本服务器游玩,请勿在语音频道嘴臭
[Server酱]社区服在此次更新后不加经验,该问题仍在等待官方确认修复
```

然后直接

```
#新建screen会话rcon
$screen -S rcon
#执行脚本
$./rconMessager.sh
#输出显示RCONING!!!!，已经在发送消息了，此时在游戏里可以看到了
#再按ctrl+A+D detach挂在后台就完事了
#不会用screen就自行百度,很简单的
```

效果样例

![ex](https://pic.imgdb.cn/item/601e81ec3ffa7d37b37bae8a.jpg)



