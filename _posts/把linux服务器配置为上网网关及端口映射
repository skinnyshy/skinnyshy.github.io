# 7.把linux服务器配置为上网网关和端口映射功能

把服务器配置成上网网关，意思就是把服务器当做路由器活者网关，实现其他机器通过这个服务器实现上网等功能。实际上这个linux服务器就相当于我们买的路由器一样，只不过我们把廉价的linux主机来当成了路由器。把linux配置为上网网关，我们需要iptables的nat表

## 7.1 nat表相关名词

- 1.DNAT DNAT全称为Destination Network Address Translation，意思是目的网络地址转换。DNAT是一种该表数据包的目的的ip地址技术，它可以使服务器能共享一个IP地址连入internet，并且继续对外提供服务（注意是对外提供服务）。通过对同一个外部ip地址分配不同的端口，映射到不同的内部服务器的端口，从而实现提供各种服务的目的，除了进行端口映射外，还可以配合SNAT的功能实现类似防火墙设备才能实现的DMZ功能，即IP的一对一映射。 例如：

```bash
iptables -t nat -A PREROUTING -d 203.81.17.88 -p tcp -m tcp --dport 80 -j DNAT --to-destination 10.1.0.16:80
```

将所有访问ip 203.81.17.88端口是80的请求，地址改写为内部web服务器10.0.0.16的80端口，从而实现端口映射。

- 2.SNAT SNAT全称为Source Network Addres Translation，意思是源网络地址转换。这是一种改变数据包源ip地址的技术，经常用来使多台计算机分享一个internet地址访问互联网，如办公室内部上网，IDC机房的内部机器上网等。 例如：

```bash
iptables -t nat -A **POSTROUTING** -s 10.0.0.0/255.255.255.0 -o eth0 -j SNAT --to-source 203.81.17.88
```

- 3.MASQUERADE MASQUERADA为动态源地址转换，即当外部IP非固定ip时的场合经常使用的选项，例如ADSL拨号上网的情况，当然对于固定单个或多个IP地址的情况，也可以使用MASQUERADE来完成共享上网。 例如：

```bash
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -j MASQUERADE
```

## 7.2 生产环境实战案例

### 7.2.1 办公室路由网关架构图

说明：这是一个实验测试的路基土，在本架构图中，我们只需要关注linux网关B，我们就把linux网关B配置成网关来做测试，实现下面的172.16.1.0/24局域网内的机器可以共享上网及对外提供服务，而把10.0.0.0/24段内的用户当成是外网用户，在实际工作中，我们需要把路由器A配置成上网网关。

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac0b05bd-014d-4100-b261-f01af26c0503/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac0b05bd-014d-4100-b261-f01af26c0503/Untitled.png)

### 7.2.2 根据上网图来实现如下要求

1. 实现C经过B，通过A上internet网
2. 在10段的足迹可以通过访问B的外网卡地址10.0.0.3:80即可访问到172.16.1.7:80 C提供的web服务 
3. .实现172段机器和10段机器互相访问（这就是前面讲过生产环境大于254台机器如何扩展网段）

```bash
iptables -t nat -A POSTROUTING -s 172.16.1.7/255.255.255.255 -o eth0 -j SNAT --to-source 10.0.0.3
iptables -t nat -A PREROUTING -d 10.0.0.3 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.16.1.7:80
```

## 7.3 演示前期准备工作

### 7.3.1 服务器网关B需具备如下条件

- 1.B需要物理条件双网卡，eth0外网网址 10.0.0.3(外网网卡配置默认网关)，eth1内网地址172.16.1.3（不需要配置网关），并且内外处于联通状态
- 2.确保服务器B自身可以上网，这样才能共享给下面机器上网。（B的默认网关指向A即可）
- **3.网关B内核文件`/etc/sysctl.conf`需要开启转发功能。** 在B上开启转发功能过程如下: 编辑/etc/sysctl.conf文件，修改`net.ipv4.ip_forward = 1`，然后执行`sysctl -p`使修改生效。 **注意：**如果之前将forward表drop的话，需要修改为accept`iptables --policy FORWARD ACCEPT`

```bash
echo "net.ipv4.ip_forward = 1">>/etc/sysctl.conf
sysctl -p #
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55b6e283-6682-405d-acc6-4e42f503b33b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55b6e283-6682-405d-acc6-4e42f503b33b/Untitled.png)

db54999f-ae64-4938-bf5b-553d1c3e7bd8.png

### 7.3.2 网关B下面局域网的机器

- 1.确保局域网172.16.1.0/24的机器，默认网关设置为服务器B的eth1的内网网卡IP
- 2.检查手段，在C上ping网关服务器B的内外网网卡IP，都通就是对的。
- 3.在C上出公网检查，可以ping 百度测试。

### 7.3.3 配置并检查上面环境的准备情况

- 1.登录C主机查看能否访问公网（注意配置好dns），ping www.biadu.com测试，当前情况不同
- 2.在笔记本上分别测试telnet 172.16.1.3 22 看能否联通，结果：当前情况通（打开网关B的转发功能）
- 3.在笔记本上分别测试ping 17.16.1.3 看能否联通，结果：当前情况通
- 4.测试登录172.16.1.3看能否访问外部页面，如ping [www.baidu.com](http://www.baidu.com/) 结果：当前情况通
- 5.在笔记本上测试telnet 172.16.1.17 22 结果：当前情况不通
- 6.在笔记本上ping 172.16.1.17 ，结果： 当前情况不通

检查完毕，下面我们进行实战测试。 另外由于iptables是在内核中运行的，我们需要价差活载入如下基本的相关内核模块。

```bash
modprobe ip_tables
modprobe iptable_filter
modprobe iptable_nat
modprobe ip_conntrack
modprobe ip_conntrack_ftp
modprobe ip_nat_ftp
modprobe ipt_state
```

将以上命令复制到B机器上运行后检查模块是否加载

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2263f741-d7d9-4233-89b0-88fcd39cc16e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2263f741-d7d9-4233-89b0-88fcd39cc16e/Untitled.png)

### 7.3.4 实现C经过B ，通过A上因特网

问题1：实现C经过B，通过A访问因特网 方法1：

```bash
iptables -t nat -A POSTROUTING -s 172.16.1.0/255.255.255.0 -o eth0 -j SNAT --to-source 10.0.0.3
#实验中是：
iptables -t nat -A POSTROUTING -s 172.16.0.134 -j SNAT --to-source 192.168.154.131
```

方法2：

```bash
iptables -t nat -A POSTROUTING -s 172.16.1.0/255.255.255.0 -o eth0 -j MASQUERADE
#注意实验中是：
iptables -t nat -A POSTROUTING -s 172.16.0.134 -o bond0 MASQUERADE
#删除指定的nat链
iptables -t nat -D POSROUTING 1
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6eef4108-15e5-4b6d-a422-29b819fb879b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6eef4108-15e5-4b6d-a422-29b819fb879b/Untitled.png)

在主机134上测试：

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/88293d94-35f0-4df4-8856-4489222b80da/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/88293d94-35f0-4df4-8856-4489222b80da/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5bd8cc1a-17af-415e-8967-503f1d24ddc7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5bd8cc1a-17af-415e-8967-503f1d24ddc7/Untitled.png)

问题2：实现外部IP地址端口到内部服务器IP和端口映射

```bash
iptables -t nat -A PREROUTING -d 10.0.0.3 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.16.1.17:80
实验中如下配置：
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 8090 -j DNAT --to-destination 172.16.0.134:80
```

以上方法仅仅是端口之间的映射，实际上我们也可以实现IP一对一的映射，即DMZ功能。

```bash
iptables -t nat -A PREROUTING -d 124.42.60.112 -j DNAT --to-destination 10.0.0.8
iptables -t nat -A PREROUTING -s 10.0.0.8 -o eth0 -j SNAT --to-source 124.42.60.112
iptables  -t nat -A POSTROUING -s 10.0.0.0/255.255.255.0 -d 10.0.0.8 -j SNAT --to-source 10.0.0.254  #注意这个
提示，以上内容需要放在配置文件中
```

在笔记本上添加路由，首先测试直接访问134主机的情况

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eaabe944-e670-49ee-8659-37e42c44382f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eaabe944-e670-49ee-8659-37e42c44382f/Untitled.png)

在131主机上完成DNT配置并检查

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c63a34ef-d672-4448-9fac-bcc990c9044f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c63a34ef-d672-4448-9fac-bcc990c9044f/Untitled.png)

再次测试通过映射方式访问

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f8985ebc-c144-422a-aa9c-da962ddd4614/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f8985ebc-c144-422a-aa9c-da962ddd4614/Untitled.png)

问题3（附加题）：实现linux网关B 对客户端C的关键字过滤及限速等操作

**注意：**

这里到操作都需要filter表中的FORWARD链来完成 这里我们使用到了filter表中的FORWARD链！

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4852102d-770e-446a-a499-7df1f4d58bd0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4852102d-770e-446a-a499-7df1f4d58bd0/Untitled.png)

理解了上图我们就明白FORWARD链的使用方式了

```bash
#过滤baidu关键字的访问
iptables -t filter -A FORWARD -s 172.16.0.134 -p tcp -m string --algo kmp --string "baidu" -j DROP
#限速需要用到TC，具体参考：[https://blog.plotcup.com/2012/08/24/linux-xia-shi-yong-iptables-he-tc-xian-zhi-liu-liang-bi-ji/](https://blog.plotcup.com/2012/08/24/linux-xia-shi-yong-iptables-he-tc-xian-zhi-liu-liang-bi-ji/)
```

实际效果如下：可以看到在134上的curl访问已经失败了

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f133b458-3529-478e-991d-7f48bbae26c8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f133b458-3529-478e-991d-7f48bbae26c8/Untitled.png)

```bash
#在主机B上删除规则：
iptables -t filter -D FORWARD 1
#再次测试客户端可以curl通了
#也可以封掉nds解析
iptables -t filter -A FORWARD -p udp --dport 53 -j DROP
#封掉客户端的icmp
iptables -t filter -A FORWARD -p icmp -j DROP
```

问题4：实现172段和10段互访（规模小不用使用三层交换的情况下） 在10段的机器添加如下路由：

```bash
route add -net 172.16.1.0/24 gw 10.0.0.3 (linux )
route add 172.16.1.0 mask 255.255.255.0 10.0.0.3 -p (windows)
```

### 7.3.5 iptables 网关生产应用说明

问题1的生产环境应用： - a.用于办公网共享上网,所有内部电脑地址通过网关公网IP或PPP共享上网 - b.IDC相房内网服务器通过网关出 提示：之所以选用liux网关,因为廉价和可自行定义很多功能,如:局域网限速,映射IP 甚至负载均衡,高可用等等,完全可以达到数万元的防火墙或路由器的效果

问题2的生产环境应用： 用于局域网没有外网地址的内网服务器，通过映射为公网IP不同的端口后对外提供服务，甚至一对一的IP映射，相当于在内部服务器上绑定外部IP意向的效果

### 7.3.6 映射多个外网ip上网 (防火墙路由器的地址池)

```bash
iptables -t nat -A POSTING -s 10.0.0.0/22 -o eth0 -j SNAT --to-source 124.42.60.103-124.42.60.115
iptables -t nat -A POSTING -s 172.16.1.0/22 -o eth0 -j SNAT --to-source 124.42.60.103-124.42.60.115
```
