# 使docker不同网络互通
## https://stackoverflow.com/questions/36035595/communicating-between-docker-containers-in-different-networks-on-the-same-host
不同网络中的容器无法相互通信，因为iptables会丢弃此类数据包。这在iptables的DOCKER-ISOLATION-STAGE-1和DOCKER-ISOLATION-STAGE-2链中显示。
##  
    sudo iptables -vL DOCKER-ISOLATION-STAGE-2
    sudo iptables -vL DOCKER-ISOLATION-STAGE-1
需要先找到网络的桥接接口名称（MyNetWork1和MyNetWork2）。 它们的名称通常看起来像BR-07D0D51191DF或BR-85F51D1CF6，它们可以使用命令“Ifconfig”或“IP链路显示”找到。 由于有多个桥接器接口，以识别用于感兴趣网络的正确桥接接口（IFConfig中所示的INET地址应匹配命令'Docker Network Insetwork1'中所示的子网地址

```
192.168.227.0/24 dev br-23816ec6d0e3 proto kernel scope link src 192.168.227.12 
192.168.228.0/24 dev br-f610130a1df5 proto kernel scope link src 192.168.228.12 
br1=$(ip r |grep 227 |awk '{print $3}')
br2=$(ip r |grep 228 |awk '{print $3}')
```
## 加了之后就能ping通了
    sudo iptables -I DOCKER-USER -i $br1 -o $br2 -j ACCEPT
    sudo iptables -I DOCKER-USER -i $br2 -o $br1 -j ACCEPT
    # 需要自己加hosts 的hostname 和ip的关联

## 列出指定的链的规则的编号来。 
    ###列出刚才加的两条规则
    sudo iptables -vL DOCKER-USER 

## 取消ping通
    ## iptable -D chain rulenum
    sudo iptables -D DOCKER-USER 1 
    sudo iptables -D DOCKER-USER 1 #执行两次一即可 如果加的位置不对要改rulenum的










docker-compose down && docker-compose up -d