############################################################
# 带ssh的centos
# Based on Centos 6.8
#
############################################################
# Set the base image to centos6.8
FROM centos:6.8

# File Author / Maintainer
MAINTAINER zhangce zhangce5413@gmail.com

##################### 常用基础软件 START #####################
RUN yum install -y net-tools vsftpd vim wget crontabs gcc make openssh-server git && \
 service sshd start && \
 echo "root:Root1.pwd" | chpasswd
##################### 常用基础软件 END #####################

##################### 安装java-1.8.0-openjdk-headless #########
RUN yum install java-1.8.0-openjdk-headless
##################### 安装java-1.8.0-openjdk-headless end #####

###################### redis START #####################
ADD conf/redis/redis /etc/init.d/redis
RUN chmod +x /etc/init.d/redis && \
 yum -y install epel-release && \
 yum -y install redis && \
 sed -i "s/daemonize[ ]*no/daemonize yes/g" /etc/redis.conf
###################### redis END #####################

###################### mysql START #####################
RUN rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm && \
 yum -y install mysql-community-server && \
 service mysqld start && \
 defaultmysqlpwd=`grep 'A temporary password' /var/log/mysqld.log | awk -F"root@localhost: " '{ print $2}' ` && \
 /usr/bin/mysql -uroot -p${defaultmysqlpwd} --connect-expired-password -e "SET PASSWORD = PASSWORD('Root1.pwd');grant all privileges on *.* to root@'%' identified by 'Root1.pwd';"
###################### mysql END #####################

ADD rc.local /etc/rc.local
RUN chmod +x /etc/rc.local
CMD ["/etc/rc.local"]