---
layout: post
title:  "SS usages in detail"
date:   2017-10-28 14:12:30 +0800
categories: blog
---
To use shadowsocks, first need to get a clone of ss from [github][ss-github].

After that, compile and get 5 components of ss: ss-local, ss-redir, 
ss-tunnel, ss-server and ss-manager.

`ss-server`

This is the server side of ss, normally it runs background.
Normal usage: `ss-server -c -u &`

`ss-local`

This is the client side of ss, normally it runs background.
Normal usage: `ss-local -s -p -k -m -t -l`
Normally the ss-local & ss-server together could solve some problem of
connecting to http/https sites. Only http/s requests are forwarded to remote
server if only set up ss-local and ss-server. The two do not solve the
problem that dns is polluted. So some application might request things other 
than http/https might fail, because they send dns requests themselves.

`ss-tunnel`

This tool is very useful at forwarding dns requests. It often works together
with iptables to forward packets to destination port 53(dns) to the remote
server and let server send the dns packet to a pre-destined dns resolver and
sends back the dns reply to the application who sends the dns request. Thus
the correct ip address of a domain could be solved. And again we use iptables
to determine that the ip address is not from mainland and forward the following
packets to this ip address to locally binded ss-redir port.
Normal usage: `ss-tunnel -s -p -m -k -b -l -L` where -L sets the forwarded dstip/port.

`ss-redir`

This tool is used together with ss-tunnel at client side. 
Shadowsocks-libev gives us at least 2 ways of connection to remote server. 
The one with ss-local/ss-server is easier to configure and use with certain 
proxy tools(In my POV, it is polipo). However, the one with 
ss-redir/ss-tunnel let client solves the dns pollution. So that none
http/https requests can also connect to remote server.
Normal usage: `ss-redir -s -p -m -k -b -l -u -v &`

How to use iptables together with ss-tunnel & ss-redir ?

Quote from [他山之石][ss-manual].
{% highlight bash %}
# 安装 ipset
## ArchLinux
pacman -S --need ipset
## CentOS
yum install ipset -y

# 获取大陆地址段
curl -sL http://f.ip.cn/rt/chnroutes.txt | egrep -v '^$|^#' > chinaip.txt

# 添加 chinaip 表
ipset -N chinaip hash:net
for i in `cat chinaip.txt`; do echo ipset -A chinaip $i >> chinaip.sh; done
bash chinaip.sh

# 持久化 chinaip 表
ipset -S > /etc/ipset.chinaip

# 新建 shadowsocks 链
iptables -t nat -N shadowsocks

# 放行保留地址、环回地址、特殊地址
iptables -t nat -A shadowsocks -d 0/8 -j RETURN
iptables -t nat -A shadowsocks -d 127/8 -j RETURN
iptables -t nat -A shadowsocks -d 10/8 -j RETURN
iptables -t nat -A shadowsocks -d 169.254/16 -j RETURN
iptables -t nat -A shadowsocks -d 172.16/12 -j RETURN
iptables -t nat -A shadowsocks -d 192.168/16 -j RETURN
iptables -t nat -A shadowsocks -d 224/4 -j RETURN
iptables -t nat -A shadowsocks -d 240/4 -j RETURN

# 放行发往 ss 服务器的数据包
iptables -t nat -A shadowsocks -d 1.2.3.4 -j RETURN # 替换1.2.3.4为你的服务器地址

# 放行大陆地址
iptables -t nat -A shadowsocks -m set --match-set chinaip dst -j RETURN

# 重定向 tcp 数据包至 1080 监听端口
iptables -t nat -A shadowsocks -p tcp -j REDIRECT --to-ports 1080

# 重定向除 dns 外的 udp 数据包至 1080 监听端口
iptables -t nat -A shadowsocks -p udp ! --dport 53 -j REDIRECT --to-ports 1080

# 本机 tcp 数据包流经 shadowsocks 链
iptables -t nat -A OUTPUT -p tcp -j shadowsocks

# 本机除 dns 外的 udp 数据包流经 shadowsocks 链
iptables -t nat -A OUTPUT -p udp ! --dport 53 -j shadowsocks

# 内网 tcp 数据包流经 shadowsocks 链
iptables -t nat -A PREROUTING -p tcp -s 192.168/16 -j shadowsocks

# 内网除 dns 外的 udp 数据包流经 shadowsocks 链
iptables -t nat -A PREROUTING -p udp ! --dport 53 -s 192.168/16 -j shadowsocks

# 开启内核网卡转发功能
## 查看是否已开启：
cat /proc/sys/net/ipv4/ip_forward # 如果值为1，表示已开启
## 若为0，则进行此步骤：
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

# 内网数据包源NAT
iptables -t nat -A POSTROUTING -s 192.168/16 -j MASQUERADE

# 持久化 iptables 规则
iptables-save > /etc/iptables.shadowsocks
{% endhighlight %}

[ss-github]:https://github.com/shadowsocks/shadowsocks-libev
[ss-manual]:https://www.zfl9.com/ss-redir.html