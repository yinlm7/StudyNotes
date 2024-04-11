ubuntu设置开机启动

```Go
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