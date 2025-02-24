命令行安装virtualbox

```bash
1. 下载、安装当前版本扩展或者apt安装

wget https://download.virtualbox.org/virtualbox/$(VBoxManage -v | cut -d_ -f1)/Oracle_VM_VirtualBox_Extension_Pack-$(VBoxManage -v | cut -d_ -f1).vbox-extpack

VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-$(VBoxManage -v | cut -dr -f1).vbox-extpack

apt安装
apt install virtualbox-ext-pack


2.配置host-only网络
VBoxManage list hostonlyifs  列表
VBoxManage hostonlyif create 创建
VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0 配置
VBoxManage dhcpserver add --ifname vboxnet0 --ip 192.168.56.100 --netmask 255.255.255.0 --lowerip 192.168.56.101 --upperip 192.168.56.254 --enable 启用dhcp
VBoxManage modifyvm <VM_NAME> --nic1 hostonly --hostonlyadapter1 vboxnet0 虚拟机中使用hostonly网络
VBoxManage hostonlyif remove vboxnet0 删除

3.配置NAT网络
VBoxManage list natnetworks 列表
VBoxManage natnetwork add --netname NatNetwork --network "10.0.2.0/24" --dhcp on 创建
VBoxManage natnetwork modify --netname NatNetwork --port-forward-4 "SSH:tcp:[127.0.0.1]:2222:[10.0.2.5]:22" 端口转发
VBoxManage natnetwork start --netname NatNetwork 启用
VBoxManage modifyvm "wifi" --nic2 natnetwork --nat-network1 NatNetwork 虚拟机连接到NAT网络
VBoxManage natnetwork remove --netname NatNetwork 删除

4.dhcp
VBoxManage list dhcpservers dhcp 服务器列表
VBoxManage dhcpserver add --netname HostInterfaceNetworking-vboxnet0 --ip 192.168.56.100 --netmask 255.255.255.0 --lowerip 192.168.56.101 --upperip 192.168.56.254 --enable 创建dhcp
VBoxManage dhcpserver remove --netname HostInterfaceNetworking-vboxnet0 删除


5.镜像操作
VBoxManage import xxx.ova  导入镜像
VBoxManage unregistervm "wifi" --delete 删除

6.虚拟机操作
VBoxManage controlvm wifi poweroff 关闭虚拟机
VBoxManage startvm "wifi" --type headless  启动虚拟机
VBoxManage showvminfo wifi 查看虚拟机i信息
```





ubuntu设置开机启动

```bash
# 下载安装VirtualBox7.0.10
https://download.virtualbox.org/virtualbox/7.0.10/virtualbox-7.0_7.0.10-158379~Ubuntu~focal_amd64.deb
dpkg -i virtualbox-7.0_7.0.10-158379~Ubuntu~focal_amd64.deb

# 创建虚拟机，配置网络，usb3.0启用，添加wifi
...

# 设置开机启动

1.设置vbox目录的权限，添加当前用户到vboxusers用户组
sudo chgrp vboxusers /etc/vbox
sudo chmod 1775 /etc/vbox
sudo adduser USERNAME vboxusers

2. 创建自启动服务新建文件，注意替换自己的用户名及虚拟机信息
vim /etc/systemd/system/vm.service

[Unit]
Description=vm-server
After=network.target virtualbox.service
Before=runlevel2.target shutdown.target
[Service]
User=USERNAME
Group=vboxusers
Type=forking
Restart=no
TimeoutSec=5min
IgnoreSIGPIPE=no
KillMode=process
GuessMainPid=no
RemainAfterExit=yes
ExecStartPre=/usr/bin/sleep 5 # 休眠几秒等待network启动
ExecStart=/usr/bin/VBoxManage startvm 虚拟机名 --type headless
ExecStop=/usr/bin/VBoxManage controlvm 虚拟机名  acpipowerbutton

[Install]
WantedBy=multi-user.target

3. 设置开机启动服务
systemctl daemon-reload
systemctl enable vm.service
```