
启动tomcat没有java权限
chmod +x /home/user/java/jdk1.7.0/bin/java
chmod 777 /home/jdk.bin


启动
./startup.sh

停止
./shutdown.sh

关闭端口占用
kill -9 port

查看tomcat是否停止
ps -ef|grep java
ps -ef |grep tomcat

ZIP
unzip file.zip
zip -r file.zip dirname

tar -czvf ***.tar.gz
tar -cjvf ***.tar.bz2
解压
tar -xzvf ./text.tar.gz -C /home/app/test/
tar –xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar –xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
unzip test.zip -d /root/

总结
1、*.tar 用 tar –xvf 解压
2、*.gz 用 gzip -d或者gunzip 解压
3、*.tar.gz和*.tgz 用 tar –xzf 解压
4、*.bz2 用 bzip2 -d或者用bunzip2 解压
5、*.tar.bz2用tar –xjf 解压
6、*.Z 用 uncompress 解压
7、*.tar.Z 用tar –xZf 解压
8、*.rar 用 unrar e解压
9、*.zip 用 unzip 解压

删除所有文件夹
rm -rf /var/log/httpd/access
删除文件
rm -f /var/log/httpd/access.log

复制
cp redis.conf redis-bak.conf
重命名
mv redis-bak.conf bak-redis.conf

查看文件
du -sh ./*

运行tomcat没有权限
chmod u+x *.sh


查看系统时间
date
同步时间
ntpdate 
例如：# ntpdate 1.cn.pool.ntp.org
http://www.pool.ntp.org是NTP的官方网站,在这上面我们可以找到离我们国家的NTP Server cn.pool.ntp.org.它有3个服务器地址：
服务器一：        1.cn.pool.ntp.org
服务器二：        2.asia.pool.ntp.org
服务器三：        3.asia.pool.ntp.org

grep   '2018-Nov-06 11:3[4-7]' shop-bussiness.log.2018-11-06


jps -lv
jmap -heap 8080
jstat -gc 23055

