# 1. Ubuntu从PPA安装以太坊

    sudo apt-get install software-properties-common
    sudo add-apt-repository -y ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install ethereum






# XX. Ubuntu基础

## XX.1 设置静态IP

    # 打开文件，/etc/network/interfaces

    # 添加以下内容到文件尾

    address 192.168.1.95
    netmask 255.255.255.0
    gateway 192.168.1.1


    # enp0s3
    
    
    # 重启网卡
    
    sudo /etc/init.d/networking restart
    
    # 开启和关闭网卡

    sudo ifup enp0s3

    sudo ifdown enp0s3

## XX.2 安装OpenSSH

    sudo apt-get install openssh-server

    # 检查服务是否启用
    ps -e | grep ssh

    netstat -antp | grep 22

    # 修改端口号，/etc/ssh/sshd_config

    # 重启ssh服务
    sudo /etc/init.d/ssh restart
