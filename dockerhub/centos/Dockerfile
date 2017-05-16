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


ADD rc.local /etc/rc.local
RUN chmod +x /etc/rc.local
CMD ["/etc/rc.local"]