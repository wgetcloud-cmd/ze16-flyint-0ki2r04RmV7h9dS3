
# 一、背景内容


其实就是接了一单，有人需要我帮忙配置一下树莓派开机启动热点。这边做个记录，该方式树莓派4B、3B都可以使用。


# 二、实际操作


## 1、使用网线连接路由器和树莓派


树莓派的网络接口一共有三个，分别是：


* eth0：有线网络接口（以太网接口）
* wlan0：无线网络接口（WiFi接口）
* lo：本地回环接口（用于本地通信，localhost：127\.0\.0\.1）


由于需要配置热点，所以需要对wlan0进行配置，故其WiFi功能需要被关闭，这里使用eth0进行网络的连接，其已经默认配置为通过DHCP来自动获取IP地址。连上网线之后通过



```


|  | ifconfig |
| --- | --- |


```

查看eth0是否有固定的IP地址，用于判断网络是否连接。
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108163239098-1059214342.png)


## 2、关闭wlan0的网络连接


终端输入：



```


|  | ip route |
| --- | --- |


```

可以看到：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108163943281-461674004.png)
说明我的设备通过eth0和wlan0连接到了网络，第一点已经提到了需要用到wlan0来开启热点，故这里需要断开wlan0的wifi连接功能再进行配置。
终端输入：



```


|  | sudo nano /etc/wpa_supplicant/wpa_supplicant.conf |
| --- | --- |


```

删除里面的全部内容，然后保存退出重启服务，一键三连\~\~\~


## 3、树莓派换源


因为后续需要使用apt install来安装Linux软件包，请确保自己的树莓派的apt源是没问题的。这里不过多展开，后续会添加一些其他链接。


## 4、hostapd


hostapd可以将设备的无线网络接口配置为热点模式，使其成为一个软AP，接受其他设备的连接。


### （1）安装且停止服务


终端输入指令进行安装：



```


|  | sudo apt install hostapd |
| --- | --- |


```

停止hostapd的服务：



```


|  | sudo systemctl stop hostapd |
| --- | --- |


```

### （2）热点参数的配置


终端输入：（如果没用这个文件，在这个路径下新建一个即可）



```


|  | sudo nano /etc/hostapd/hostapd.conf |
| --- | --- |


```

填入：



```


|  | interface=wlan0 |
| --- | --- |
|  | driver=nl80211 |
|  | ssid=??? |
|  | hw_mode=g |
|  | channel=7 |
|  | wmm_enabled=0 |
|  | macaddr_acl=0 |
|  | auth_algs=1 |
|  | ignore_broadcast_ssid=0 |
|  | wpa=2 |
|  | wpa_passphrase=??? |
|  | wpa_key_mgmt=WPA-PSK |
|  | wpa_pairwise=TKIP |
|  | rsn_pairwise=CCMP |


```

ssid是热点名称；wpa\_passphrase是热点密码，根据需要修改。
填完之后如图:
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108165105867-853290625.png)


### （3）给hostapd指定热点配置文件的路径


终端输入：



```


|  | sudo nano /etc/default/hostapd |
| --- | --- |


```

去掉DAEMON\_CONF的注释，并配置成/etc/hostapd/hostapd.conf，如图所示。意思就是告诉hostapd要从/etc/hostapd/hostapd.conf读取配置参数。具体如图：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108165247207-549271255.png)
最后重启hostapd服务,终端输入：



```


|  | sudo systemctl unmask hostapd |
| --- | --- |
|  | sudo systemctl enable hostapd |
|  | sudo systemctl start hostapd |


```

稍等就可以看到产生的热点信号了。但是此时热点无法连接，因为此热点信号没有连接网络，也无法给客户端分配IP。
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108165356209-547705725.png)


## 5、dhcpcd


dhcpcd是用于进行IP地址相关操作的软件包，这里我们用这个软件包来进行热点IP地址的固定。
终端输入指令进行安装：


### （1）安装dhcpcd



```


|  | sudo apt install dhcpcd |
| --- | --- |


```

### （2）编辑配置文件


编辑dhcpcd配置文件，终端输入：



```


|  | sudo nano /etc/dhcpcd.conf |
| --- | --- |


```

删除当中的全部内容，然后输入：



```


|  | interface wlan0 |
| --- | --- |
|  | static ip_address=192.168.4.1/24 |
|  | nohook wpa_supplicant |


```

**这里设置的static ip\_address最好不要和你周围的无线网络在同一个网段，比如你家无线网络的网段是192\.168\.2\.X，那么这里的静态IP的第三位就设置成其他的就好**。


保存好之后，重启dhcpcd 服务，终端输入：



```


|  | sudo systemctl restart dhcpcd |
| --- | --- |


```

### （3）检查


之后检查wlan0的IP地址，终端输入



```


|  | ifconfig |
| --- | --- |


```

可以看到IP地址被固定了，如图：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108170105235-1778312204.png)


## 6、dnsmasq


dnsmasq软件包用于给连接热带的设备自动分配IP地址


### （1）安装dnsmasq


终端输入：



```


|  | sudo apt install dnsmasq |
| --- | --- |


```

停止其服务：



```


|  | sudo systemctl stop dnsmasq |
| --- | --- |


```

### （2）配置参数


终端输入：



```


|  | sudo nano /etc/dnsmasq.conf |
| --- | --- |


```

删除其全部内容，填入：



```


|  | interface=wlan0 |
| --- | --- |
|  | dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h |


```

含义：dhcp 服务会给客户端分配 192\.168\.4\.2 到 192\.168\.4\.20 的 IP 空间，24 小时租期。
如图：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108170608406-1327204098.png)
之后重启服务。终端输入：



```


|  | sudo systemctl reload dnsmasq |
| --- | --- |


```

如果报错了，就执行如下指令：



```


|  | sudo systemctl unmask dnsmasq |
| --- | --- |
|  | sudo systemctl enable dnsmasq |
|  | sudo systemctl start dnsmasq |


```

### （3）尝试连接热点


服务重启之后，热点就可以连接了。因为树莓派给连接热点的设备分配了IP。不过此时还无法上网。
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108170812987-249534090.png)


## 7、启用IP转发


终端输入：



```


|  | sudo nano /etc/sysctl.conf |
| --- | --- |


```

找到并且取消注释以下行：



```


|  | net.ipv4.ip_forward=1 |
| --- | --- |


```

如图：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108171120235-1881044226.png)


## 8、配置树莓派防火墙


终端输入：



```


|  | sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE |
| --- | --- |


```

之后保存防火墙规则：



```


|  | sudo sh -c "iptables-save > /etc/iptables.ipv4.nat" |
| --- | --- |


```

之后让设备每次重启都重载这个防火墙规则：



```


|  | sudo nano /etc/rc.local |
| --- | --- |


```

将iptables\-restore \< /etc/iptables.ipv4\.nat加到最后一行exit 0的前面，如图：
![image](https://img2024.cnblogs.com/blog/3089600/202411/3089600-20241108171249598-357169845.png)


## 9、重启设备，enjoy yourself\~


重启全部服务：



```


|  | sudo systemctl unmask hostapd |
| --- | --- |
|  | sudo systemctl unmask dnsmasq |
|  | sudo systemctl enable hostapd |
|  | sudo systemctl enable dnsmasq |
|  | sudo systemctl start hostapd |
|  | sudo systemctl start dnsmasq |


```

重启设备：



```


|  | sudo reboot |
| --- | --- |


```

 本博客参考[FlowerCloud机场](https://hushicha.org)。转载请注明出处！
