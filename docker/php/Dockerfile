############################################################
# 创建开发环境的dockerfile
# Based on Centos 6
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

################## openresty start ######################
### 安装依赖
RUN yum -y install readline-devel pcre-devel openssl-devel && \
### 安装软件
RUN wget https://openresty.org/download/openresty-1.11.2.3.tar.gz && \
 tar -xzvf openresty-1.11.2.3.tar.gz && \
 cd openresty-1.11.2.3 && \
 ./configure && \
 make && \
 make install && \

 ln -s /usr/local/openresty/nginx/conf /etc/nginx && \
 mkdir /etc/nginx/vhosts

###添加配置
ADD conf/nginx/* /etc/nginx/
ADD conf/nginx/vhosts/* /etc/nginx/vhosts/
ADD conf/nginx/nginx /etc/init.d/nginx
##################### openresty END #####################


###################### php7 START #####################

RUN cd /usr/src && \
 ###安装php7
 yum -y install libxml2-devel curl-devel libpng-devel gd-devel autoconf xz && \
 wget http://cn2.php.net/get/php-7.1.2.tar.xz/from/this/mirror && \
 mv mirror php-7.1.2.tar.xz && xz -d php-7.1.2.tar.xz && tar xvf php-7.1.2.tar && \
 cd php-7.1.2 && \
 ./configure --prefix=/usr/local/php --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-gd --with-jpeg-dir=/usr/lib64/ --with-iconv --with-openssl --with-curl --enable-pcntl --enable-bcmath --enable-json --enable-fpm --enable-mbstring --enable-soap --enable-opcache && \
 make -j4 && make install && \
 echo -e 'export PATH=/usr/local/php/bin:usr/local/php/sbin:$PATH' >> ~/.bashrc && \
 source ~/.bashrc

 ### yaf
 cd /usr/src && \
 wget http://pecl.php.net/get/yaf-3.0.4.tgz && tar zxvf yaf-3.0.4.tgz && \
 cd yaf-3.0.4 && phpize && ./configure && make -j4 && make install && \

 ### yac
 cd /usr/src && \
 wget http://pecl.php.net/get/yac-2.0.1.tgz && tar zxvf yac-2.0.1.tgz && \
 cd yac-2.0.1 && phpize && ./configure && make -j4 && make install && \

 ### redis
 cd /usr/src && \
 wget http://pecl.php.net/get/redis-3.1.1.tgz && tar zxvf redis-3.1.1.tgz && \
 cd redis-3.1.1 && phpize && ./configure && make && make install && \

 ### phpunit
 cd /usr/src && \
 wget https://phar.phpunit.de/phpunit-6.1.phar && \
 cp phpunit.phar /usr/local/bin/phpunit && \
 chmod +x /usr/local/bin/phpunit && \

 echo -e 'export PATH=/usr/local/php/bin:usr/local/php/sbin:$PATH' >> ~/.bashrc && \
 source ~/.bashrc

ADD conf/php/php.ini /usr/local/php/lib/
ADD conf/php/php-fpm.conf /usr/local/php/etc/
ADD conf/php/www.conf /usr/local/php/etc/php-fpm.d/
###################### php7 END #####################

ADD rc.local /etc/rc.local
RUN chmod +x /etc/rc.local
CMD ["/etc/rc.local"]