ip操作

```
ifocnfig

ifconfig // 显示网络接口配置
ifconfig eth0 192.168.1.100 // 设置网卡ip
ifconfig eth0 arp // 开启arp/down
ifconfig eth0 -arp // 关闭arp
ifconfig eth0 up/down // 开启关闭网卡


ip link 查看ip
ip addr add 192.168.1.100/24 dev eth0 设置临时ip
ip link set dev eth0 up  启用网卡
ip route add default via <网关IP地址> dev eth0 设置默认网关
```

