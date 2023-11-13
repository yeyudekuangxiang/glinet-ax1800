# glinet-ax1800
## 以下教程内容转自[OpenWRT下安装和配置shadowsocks](https://douxinchun.github.io/posts/install-shadowsocks-on-openwrt/) 有部分修改

<div class="content">
    <p>本文主要记录在openWRT下安装和配置shadowsocks的简要过程，便于日后查找和备忘。成功安装后可以实现透明代理，分流和防DNS污染。</p>

<h2 id="environment"><span class="me-2">Environment</span><a href="#environment" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>
<ul>
  <li>路由器型号：gl.inet ax1800</li>
  <li>固件版本：OpenWrt 21.02-SNAPSHOT r16399+159-c67509efd7 / LuCI openwrt-22.03 branch git-21.284.67084-e4d24f0</li>
</ul>

<h2 id="工作原理"><span class="me-2">工作原理</span><a href="#工作原理" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>

<ol>
  <li>
    <p>dnsmasq是openwrt自带的一个软件，提供dns缓存，dhcp等功能。dnsmasq会将dns查询数据包转发给chinadns。</p>
  </li>
  <li>
    <p>chinadns的上游DNS服务器有两个，一个是<code class="language-plaintext highlighter-rouge">国内DNS</code>，一个是<code class="language-plaintext highlighter-rouge">可信DNS</code>（国外DNS）。</p>
    <ul>
      <li>chinadns会同时向上游的DNS发送请求</li>
      <li>如果<code class="language-plaintext highlighter-rouge">可信DNS</code>先返回, 则直接采用<code class="language-plaintext highlighter-rouge">可信DNS</code>的结果</li>
      <li>如果<code class="language-plaintext highlighter-rouge">国内DNS</code>先返回, 分两种情况: 如果返回的结果是国内IP,则采用;否则丢弃并等待采用<code class="language-plaintext highlighter-rouge">可信DNS</code>的结果</li>
    </ul>
  </li>
</ol>

<p>3.dns-forwarder 支持DNS TCP查询, 如果ISP的UDP不稳定, 丢包严重,可以使用dns-forwarder来代替<code class="language-plaintext highlighter-rouge">ss-tunnel</code>来进行DNS查询.</p>

<p>4.shadowsocks 用于转发数据包, 科学上网. 关于shadowsocks的科普文章可查看这里: https://www.css3er.com/p/107.html</p>

<h2 id="相关的ipk软件包下载地址"><span class="me-2">相关的ipk软件包下载地址</span><a href="#相关的ipk软件包下载地址" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>

[shadowsocks-libev_3.3.5-1_arm_cortex-a7.ipk](shadowsocks-libev_3.3.5-1_arm_cortex-a7.ipk)  
[ChinaDNS_1.3.3-1_arm_cortex-a7.ipk](ChinaDNS_1.3.3-1_arm_cortex-a7.ipk)  
[dns-forwarder_1.2.1-2_arm_cortex-a7.ipk](dns-forwarder_1.2.1-2_arm_cortex-a7.ipk)   
[luci-app-shadowsocks_2.1.1-1_all.ipk](luci-app-shadowsocks_2.1.1-1_all.ipk)  
[luci-app-chinadns_1.6.3-1_all.ipk](luci-app-chinadns_1.6.3-1_all.ipk)  
[luci-app-dns-forwarder_1.6.3-1_all.ipk](luci-app-dns-forwarder_1.6.3-1_all.ipk)  

<h3 id="openwrt-shadowsocks"><span class="me-2">openwrt-shadowsocks</span><a href="#openwrt-shadowsocks" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3>
<p><strong>GitHub</strong>: https://github.com/shadowsocks/openwrt-shadowsocks  <br>
<strong>luci-app-shadowsocks</strong>: https://github.com/shadowsocks/luci-app-shadowsocks</p>

<ul>
  <li>
    <p>shadowsocks-libev</p>


</pre></td><td class="rouge-code"><pre> 客户端/
 └── usr/
     └── bin/
         ├── ss-local       // 提供 SOCKS 正向代理, 在透明代理工作模式下用不到这个.
         ├── ss-redir       // 提供透明代理, 从 v2.2.0 开始支持 UDP
         └── ss-tunnel      // 提供端口转发, 可用于 DNS 查询
</pre></td></tr></tbody></table></code></div>    </div>
  </li>
  <li><p>shadowsocks-libev-server</p>

</pre></td><td class="rouge-code"><pre>服务端/
└── usr/
    └── bin/
        └── ss-server      // 服务端可执行文件
</pre></td></tr></tbody></table></code></div>    </div>
  </li>
</ul>

<h3 id="chinadns"><span class="me-2">ChinaDNS</span><a href="#chinadns" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3>
<p><strong>GitHub</strong>: https://github.com/aa65535/openwrt-chinadns<br>
<strong>原版ChinaDNS地址, 被请喝茶后已不再维护</strong>:https://github.com/shadowsocks/ChinaDNS<br>
<strong>luci-app-chinadns</strong>: https://github.com/aa65535/openwrt-dist-luci</p>

<p>更新 /etc/chinadns_chnroute.txt</p>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre> wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' &gt; /etc/chinadns_chnroute.txt
</pre></td></tr></tbody></table></code></div></div>

### dns-forwarder
<p> 
<strong>GitHub</strong>: https://github.com/aa65535/openwrt-dns-forwarder<br>
<strong>luci-app-dns-forwarder</strong>: https://github.com/aa65535/openwrt-dist-luci</p>

<h3 id="dnsmasq"><span class="me-2">dnsmasq</span><a href="#dnsmasq" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3>
<p>openWRT自带, 无需自行下载安装.<br>
<strong>GitHub</strong>: https://github.com/aa65535/openwrt-dnsmasq</p>

<h2 id="install"><span class="me-2">Install</span><a href="#install" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>

<p>下载完成有两种方式安装<br>
方式一(建议): 通过web使用luci安装:
路径: 系统 -&gt; Software -&gt; Upload Package… -&gt; Install</p>

<p>方式二: 直接在线通过opkg命令来安装(注意使用方式需要提前更新好软件源, <code class="language-plaintext highlighter-rouge">opkg update</code>):</p>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre>opkg install luci-compat
</pre></td></tr></tbody></table></code></div></div>
<h2 id="config"><span class="me-2">Config</span><a href="#config" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>

<h3 id="方式一-使用luci来配置"><span class="me-2">方式一, 使用luci来配置</span><a href="#方式一-使用luci来配置" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3>
<p>登录luci.</p>

<ol>
  <li>配置ss-server <br>
 <code class="language-plaintext highlighter-rouge">服务</code> -&gt; <code class="language-plaintext highlighter-rouge">影梭</code> -&gt; <code class="language-plaintext highlighter-rouge">服务器管理</code>, 添加自己的shadowsocks server</li>
  <li>配置dnsmasq
    <ul>
      <li><code class="language-plaintext highlighter-rouge">网络</code> -&gt; <code class="language-plaintext highlighter-rouge">DHCP/DNS</code> -&gt; <code class="language-plaintext highlighter-rouge">常规设置</code> -&gt; <code class="language-plaintext highlighter-rouge">本地服务器</code>, 设置为 <code class="language-plaintext highlighter-rouge">127.0.0.1#5353</code></li>
      <li><code class="language-plaintext highlighter-rouge">网络</code> -&gt; <code class="language-plaintext highlighter-rouge">DHCP/DNS</code> -&gt; <code class="language-plaintext highlighter-rouge">HOSTS和解析文件</code>, 勾选: <code class="language-plaintext highlighter-rouge">忽略解析文件</code></li>
    </ul>
  </li>
  <li>配置ChinaDNS<br>
 <code class="language-plaintext highlighter-rouge">服务</code> -&gt; <code class="language-plaintext highlighter-rouge">ChinaDNS</code><br>
 监听端口: <code class="language-plaintext highlighter-rouge">5353</code><br>
 上游服务器修改为: <code class="language-plaintext highlighter-rouge">114.114.114.114,127.0.0.1#5300</code><br>
 这样<code class="language-plaintext highlighter-rouge">国内DNS</code>: <code class="language-plaintext highlighter-rouge">114.114.114.114</code>, <code class="language-plaintext highlighter-rouge">可信DNS</code>: <code class="language-plaintext highlighter-rouge">127.0.0.1#5353</code>, 勾选 <code class="language-plaintext highlighter-rouge">启用</code>, 保存设置</li>
  <li>配置dns-forwarder<br>
 <code class="language-plaintext highlighter-rouge">服务</code> -&gt; <code class="language-plaintext highlighter-rouge">DNS转发</code><br>
 监听端口: <code class="language-plaintext highlighter-rouge">5300</code> 
 监听地址: <code class="language-plaintext highlighter-rouge">0.0.0.0</code><br>
 上游 DNS: <code class="language-plaintext highlighter-rouge">8.8.8.8 </code>
 勾选, <code class="language-plaintext highlighter-rouge">启用</code> 保存</li>
  <li>
    <p>配置shadowsocks 透明代理 + 访问控制<br>
 <code class="language-plaintext highlighter-rouge">服务</code> -&gt; <code class="language-plaintext highlighter-rouge">影梭</code> -&gt; <code class="language-plaintext highlighter-rouge">常规设置</code> -&gt; <code class="language-plaintext highlighter-rouge">透明代理</code><br>
 <code class="language-plaintext highlighter-rouge">主服务器</code>, 选择setp1中配置的ss-server, 保存.<br>
 <code class="language-plaintext highlighter-rouge">服务</code>-&gt; <code class="language-plaintext highlighter-rouge">影梭</code> -&gt; <code class="language-plaintext highlighter-rouge">常规设置</code> -&gt; <code class="language-plaintext highlighter-rouge">访问控制</code>-&gt; <code class="language-plaintext highlighter-rouge">外网区域 </code> <br>
 <code class="language-plaintext highlighter-rouge">被忽略IP列表</code>, 选择 <code class="language-plaintext highlighter-rouge">ChinaDNS路由表</code>, 保存设置.  注意这里的优先级: (走代理IP列表 = 强制走代理IP) &gt; (额外被忽略IP = 被忽略IP列表)</p>
  </li>
  <li><code class="language-plaintext highlighter-rouge">保存并应用</code> 所有配置, reboot openWRT</li>
</ol>

<h3 id="方式二-直接编辑etcconfig目录下的文件"><span class="me-2">方式二, 直接编辑/etc/config目录下的文件</span><a href="#方式二-直接编辑etcconfig目录下的文件" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h3>
<p>课外阅读: UCI System
<a href="https://oldwiki.archive.openwrt.org/doc/uci">UCI system</a></p>

<blockquote>
  <p>The abbreviation UCI stands for Unified Configuration Interface and is intended to centralize the configuration of OpenWrt.</p>
</blockquote>

<h4 id="etcconfigshadowsocks"><span class="me-2">/etc/config/shadowsocks</span><a href="#etcconfigshadowsocks" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h4>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
</pre></td><td class="rouge-code"><pre>root@OpenWrt:~# cat /etc/config/shadowsocks

config general
	option startup_delay '0'

config transparent_proxy
	option udp_relay_server 'nil'
	option local_port '1234'
	option mtu '1492'
	list main_server 'cfg054a8f'

config socks5_proxy
	option local_port '1080'
	option mtu '1492'
	list server 'nil'

config port_forward
	option local_port '5300'
	option mtu '1492'
	option destination '8.8.8.8:53'
	list server 'nil'

config servers
	option fast_open '0'
	option no_delay '0'
	option timeout '60'
	option server '服务器地址,注意luci下这里只能是ip'
	option server_port '端口'
	option password '密码'
	option encrypt_method '加密方式'
	option alias 'ss服务别名'

config access_control
	option self_proxy '1'
	option lan_target 'SS_SPEC_WAN_AC'
	option wan_bp_list '/etc/chinadns_chnroute.txt'
</pre></td></tr></tbody></table></code></div></div>

<h4 id="etcconfigdhcp"><span class="me-2">/etc/config/dhcp</span><a href="#etcconfigdhcp" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h4>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
</pre></td><td class="rouge-code"><pre>root@OpenWrt:~# cat /etc/config/dhcp

config dnsmasq
	option domainneeded '1'
	option localise_queries '1'
	option rebind_protection '1'
	option rebind_localhost '1'
	option domain 'lan'
	option expandhosts '1'
	option authoritative '1'
	option readethers '1'
	option leasefile '/tmp/dhcp.leases'
	option localservice '1'
	option local '127.0.0.1#5353'
	option noresolv '1'
...
</pre></td></tr></tbody></table></code></div></div>

<h4 id="etcconfigchinadns"><span class="me-2">/etc/config/chinadns</span><a href="#etcconfigchinadns" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h4>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre></td><td class="rouge-code"><pre>root@OpenWrt:~# cat /etc/config/chinadns

config chinadns
	option chnroute '/etc/chinadns_chnroute.txt'
	option addr '0.0.0.0'
	option port '5353'
	option bidirectional '1'
	option server '114.114.114.114,127.0.0.1#5300'
	option enable '1'
</pre></td></tr></tbody></table></code></div></div>

<h4 id="etcconfigdns-forwarder"><span class="me-2">/etc/config/dns-forwarder</span><a href="#etcconfigdns-forwarder" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h4>
<div class="language-plaintext highlighter-rouge"><div class="code-header">
        <span data-label-text="Plaintext"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre></td><td class="rouge-code"><pre>root@OpenWrt:~# cat /etc/config/dns-forwarder

config dns-forwarder
	option listen_addr '0.0.0.0'
	option listen_port '5300'
	option enable '1'
	option dns_servers '8.8.8.8'
</pre></td></tr></tbody></table></code></div></div>

<p>验证配置是否生效</p>
<div class="language-bash highlighter-rouge"><div class="code-header">
        <span data-label-text="Shell"><i class="fas fa-code fa-fw small"></i></span>
      <button aria-label="copy" data-title-succeed="Copied!"><i class="far fa-clipboard"></i></button></div><div class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
</pre></td><td class="rouge-code"><pre>root@OpenWrt:~# netstat <span class="nt">-lpn</span> | <span class="nb">grep </span>ss
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:1234            0.0.0.0:<span class="k">*</span>               LISTEN      13469/ss-redir
root@OpenWrt:~# netstat <span class="nt">-lpn</span> | <span class="nb">grep </span>5353
udp        0      0 0.0.0.0:5353            0.0.0.0:<span class="k">*</span>                           1438/chinadns
root@OpenWrt:~# netstat <span class="nt">-lpn</span> | <span class="nb">grep </span>5300
udp        0      0 0.0.0.0:5300            0.0.0.0:<span class="k">*</span>                           12993/dns-forwarder
root@OpenWrt:~# netstat <span class="nt">-lpn</span> | <span class="nb">grep </span>53
tcp        0      0 127.0.0.1:53            0.0.0.0:<span class="k">*</span>               LISTEN      2254/dnsmasq
...

root@OpenWrt:~# nslookup google.com 127.0.0.1#5353
Server:		127.0.0.1
Address:	127.0.0.1#5353

Name:      google.com
Address 1: 142.250.72.238
Address 2: 2607:f8b0:4007:80d::200e
root@OpenWrt:~#
</pre></td></tr></tbody></table></code></div></div>

<h2 id="issues"><span class="me-2">Issues</span><a href="#issues" class="anchor text-muted"><i class="fas fa-hashtag"></i></a></h2>
<ul>
  <li>luci-app-shadowsocks 不支持domain的方式配置ss-server, 需要使用IP地址</li>
</ul>



  </div>
