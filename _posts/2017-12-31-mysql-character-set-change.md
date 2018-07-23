---
layout: post
title:  "MYSQL character set utf8로 변경하기"
tags:
  - MYSQL
hero: https://source.unsplash.com/collection/427433/
overlay: gray
---
[환경]
mysql : 5.5.57

php 파일을 통해 insert문과 select문을 하여 mysql에 데이터를 넣거나 가져오는데 한글이 물음표로 나오거나 깨지는 현상이 일어났다. 때문에 이 문제를 해결한 과정을 적어보려고 한다. 이 문제를 해결하기 위해 php파일 내에서의 character set과 파일을 저장할때 인고딩 형식과 mysql의 character set을 모두 utf8로 통일시켜줘야 한다. 이 게시물에서는 mysql의 character set을 설정하는 과정을 담아보려고 한다.
{: .lead}

# 1. my.cnf 파일수정

먼저 mysql의 character set 설정을 바꿔주기 위해
<pre><code>/etc/mysql/my.cnf
</code></pre>

경로에 있는 설정파일을 수정하기로 했다.

<pre><code>[client]
default-character-set = utf8

[mysqld]
init_connect="SET collation_connection = utf8_general_ci"
init_connect="SET NAMES utf8"
default-character-set = utf8
character-set-server = utf8
collation-server = utf8_general_ci

[mysql]
default-character-set = utf8

[mysqldump]
default-character-set = utf8
</code></pre>

위의 내용들을 추가해준 후
<pre><code>service mysql restart</code></pre>

명령어를 사용하여 mysql을 restart를 했더니
*Job for mysql.service failed. See 'systemctl status mysql.service' and 'journalctl -xn' for details.*
이라는 오류문구가 나왔다.
<pre><code>service mysql status</code></pre>

를 통해 mysql이 동작하고 있는지 확인해 보았다.  


<pre><code>● mysql.service - LSB: Start and stop the mysql database server daemon
   Loaded: loaded (/etc/init.d/mysql)
   Active: failed (Result: exit-code) since Thu 2017-12-28 04:10:16 KST; 1m                                                                                  in 13s ago
  Process: 18829 ExecStop=/etc/init.d/mysql stop (code=exited, status=0/SUC                                                                                  CESS)
  Process: 19761 ExecStart=/etc/init.d/mysql start (code=exited, status=1/F                                                                                  AILURE)

Dec 28 04:10:16 cinnamonpi_4 /etc/init.d/mysql[20601]: 0 processes alive...
Dec 28 04:10:16 cinnamonpi_4 /etc/init.d/mysql[20601]: [61B blob data]
Dec 28 04:10:16 cinnamonpi_4 /etc/init.d/mysql[20601]: error: 'Can't con...
Dec 28 04:10:16 cinnamonpi_4 /etc/init.d/mysql[20601]: Check that mysqld...
Dec 28 04:10:16 cinnamonpi_4 /etc/init.d/mysql[20601]:
Dec 28 04:10:16 cinnamonpi_4 mysql[19761]: Starting MySQL database serv...!
Dec 28 04:10:16 cinnamonpi_4 systemd[1]: mysql.service: control process...1
Dec 28 04:10:16 cinnamonpi_4 systemd[1]: Failed to start LSB: Start and....
Dec 28 04:10:16 cinnamonpi_4 systemd[1]: Unit mysql.service entered fai....
</code></pre>

mysql이 오류로 인해 동작을 안하고있었다. 오류로그를 확인해보기로 했다.  
**/var/log/mysql/error.log** 경로의 오류로그를 살펴보니
*171228  4:27:27 [ERROR] /usr/sbin/mysqld: unknown variable 'default-character-set=utf8'* 이라는 오류로그를 확인할 수 있었다.  
my.cnf 설정파일에서 [mysqld]의 **default-caracter-set=utf8** 이 부분이 문제가 된것으로 보인다.  
여러 자료들을 살펴보니 5.5.3버전 부터 mysqld의 **default-caracter-set** 설정이 없어진 것을 알게 되었다.  

<https://dev.mysql.com/doc/refman/5.5/en/server-options.html#option_mysqld_default-character-set>

my.cnf 설정파일에서 [mysqld]의 **default-caracter-set=utf8** 부분을 지운 후에 다시 mysql을 start시키니 정상적으로 작동이 되었다.  

--------------------------------------------------------------

# 2. mysql db 및 table character set 변경

mysql 내에서 status를 통해 character set을 확인해 보니  

<pre><code>Connection id:          37
Current database:       test
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.5.57-0+deb8u1 (Raspbian)
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /var/run/mysqld/mysqld.sock
Uptime:                 9 min 45 sec
</code></pre>

위와 같이 **Db characterset**이 **latin1**로 되있는 것을 확인하였고, 다시 insert문과 select문을 해봤지만 여전히 한글이 깨지는 현상이 일어났다.  

<pre><code>alter database test default character set utf8</code></pre>

을 통해 db의 character set을 바꿔주고  

<pre><code>alter table attend convert to charset utf8</code></pre>

로 table의 character set 또한 바꿔주었다.
