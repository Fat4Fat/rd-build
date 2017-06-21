开发环境搭建教程
依赖环境

操作系统

Mac OS

其他软件


安装指导

安装VirtualBox

软件安装

参考mac软件安装方式，使用安装包按照指引进行安装。

查看安装结果

安装完成后在终端窗口下执行命令 VBoxManage --version 查看版本信息。若显示版本号，则表示VirtualBox安装成功。

安装Vagrant

软件安装

参考mac软件安装方式，使用安装包按照指引进行安装。

查看安装结果

安装完成后在终端窗口下执行命令 vagrant --version 查看版本信息。若显示版本号，则表示Vagrant安装成功。
环境搭建

下载vagrant box文件

下载地址参考本文章节：依赖环境－软件－Vagrant Box。假设下载到本地的文件的绝对路径为/tmp/centos7_dev.box。

导入vagrant box文件

导入文件

执行命令 vagrant box add centos7_dev /tmp/centos7_dev.box 。
其中：
/tmp/centos7_dev.box 为上一步骤中下载到本地/tmp目录下的box文件；
centos7_dev 为导入后的box命名，可自行更改。本文中使用centos7_dev作为示例。

查看导入结果

导入完成后执行命令 vagrant box list 可查看已导入的box文件列表，其中包括刚导入的 centos7_dev。

初始化虚拟机环境

切换目录

切换到项目目录的父目录。
例：假设项目路径为 /home/user1/projects/demo，则切换当前路径为 /home/user1/projects

初始化

执行命令：vagrant init centos7_dev。
执行成功后系统默认在当前目录下生成Vagrantfile文件。

编辑Vagrantfile文件

在config.vm.box = "centos7_dev"的下一行开始添加以下内容：
config.ssh.username = "vagrant"
config.ssh.password = "vagrant"
config.vm.network "forwarded_port", guest: 80, host: 8080
config.vm.network "public_network"

启动虚拟机

执行命令 vagrant up，提示选择桥接网卡时需要根据提示选择当前可用的网卡

登录虚拟机

执行命令 vagrant ssh 登录到虚拟机开发环境

使用共享目录

/vagrant 目录对应用户真实操作系统下初始化环境的目录（即执行 vagrant up 的目录。本文示例中步骤3的 /home/user1/projects）。所有此目录下的文件在虚拟机环境下都可以直接访问，无需使用ftp或scp等文件传输工具进行上传。因此建议将项目代码的父目录作为共享目录，以方便开发。
开发环境介绍

预安装软件


常用目录



常用命令
