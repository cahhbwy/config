# network configuration and tips

## ssh登陆不上去

1. 检查用户名和密码

2. 检查sshd_config的允许登陆方式

3. 检查hosts.deny和iptables、denyhosts
```bash
sudo iptables -F
sudo service denyhosts stop
sudo denyhosts --purge-all
sudo service denyhosts start
```

## iptables

### 不同ip的端口转发，如通过ssh到192.168.31.8:30022达到ssh 192.168.31.88的目的
```bash
iptables -t nat -A PREROUTING -p tcp -d 192.168.31.8 --dport 30022 -j DNAT --to-destination 192.168.31.88:22
iptables -t nat -A POSTROUTING -p tcp -d 192.168.31.88 --dport 22 -j SNAT --to-source 192.168.31.8
iptables-save
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 查看nat表
```bash
iptables -nvL --line-number
```

_加上-t nat，查看nat表_
```bash
iptables -L -t nat --line-number
```

### 删除
```bash
iptables -D POSTROUTING 1 -t nat
```

### 清空
```bash
iptables -F
```

### 替换 -R
```bash
iptables -t nat -R POSTROUTING 2 ......
```

### 插入 -I
```bash
iptables -t nat -I POSTROUTING 1 ......
```
