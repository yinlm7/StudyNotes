apt相关命令

```

apt update // 更新资源
apt upgrade // 执行更新
apt install --reinstall xxx // 重新安装
apt --fix-broken install // 修复依赖
apt autoremove // 删除未使用包
apt remove xxx // 卸载
apt purge xxx // 卸载清除配置文件

apt install --print-uris --yes xxx > vim.txt   // 输出依赖包地址
awk '{print $1}' ../vim.txt | grep http | sed "s/'//g" | wget -i -  下载依赖包


dpkg 

dpkg -i xxx.deb // 安装
dpkg -i *.deb  // dpkg安装所有deb文件
dpkg --get-selections | grep -v deinstall // 查看已安装的包
dpkg -l | grep xxx // 查看指定包的信息 


```

