############################################################
# 带ssh的centos
# Based on Centos 6.8
#
############################################################
# Set the base image to centos6.8
FROM centos:6.8

# File Author / Maintainer
MAINTAINER lvyalin lvyalin.yl@gmail.com

##################### 常用基础软件 START #####################
RUN yum install -y net-tools vsftpd vim wget crontabs gcc make openssh-server git && \
 service sshd start && \
 echo "root:Root1.pwd" | chpasswd
##################### 常用基础软件 END #####################

################## openresty start ######################
### 安装依赖
RUN yum -y install readline-devel pcre-devel openssl-devel && \
### 安装软件
 wget https://openresty.org/download/openresty-1.11.2.3.tar.gz && \
 tar -xzvf openresty-1.11.2.3.tar.gz && \
 cd openresty-1.11.2.3 && \
 ./configure && make && make install && \

 ln -s /usr/local/openresty/nginx/conf /etc/nginx && \
 mkdir /etc/nginx/vhosts
###添加配置
ADD conf/nginx/* /etc/nginx/
ADD conf/nginx/vhosts/* /etc/nginx/vhosts/
ADD conf/nginx/nginx /etc/init.d/nginx
##################### openresty END #####################


ADD rc.local /etc/rc.local
RUN chmod +x /etc/rc.local
CMD ["/etc/rc.local"]