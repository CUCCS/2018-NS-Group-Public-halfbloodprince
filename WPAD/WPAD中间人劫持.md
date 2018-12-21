# WPAD中间人劫持

## WPAD功能

- WPAD ： Web代理自动发现协议，该协议的功能是可以使局域网中的用户浏览器可以自动发现内网中的代理服务器，并使用已发现的代理服务器连接互联网或企业内网

## WPAD工作原理

- 当系统开启了代理自动发现功能后，用户使用浏览器上网时，浏览器就会在当前局域网中自动查找代理服务器，如果找到了代理服务器，则会从代理服务器中下载一个名为PAC(Proxy Auto-Config)的配置文件。该文件中定义了用户在访问一个URL时应该使用的代理服务器。浏览器会下载并解析该文件，将相应的代理服务器设置到用户的浏览器中。因此正确PAC文件的获取至关重要

- 一个PAC文件至少定义了一个 ` FindProxyForURL(url, host) ` 函数，该函数的返回值指定了URL的访问方式

        function FindProxyForURL(url, host) {
        if (url== 'http://Her0in.org/') return 'DIRECT';
        if (shExpMatch(host, "*.wooyun.org")) return "DIRECT";
        if (host== 'wooyun.com') return 'SOCKS 127.1.1.1:8080';
        if (dnsResolve(host) == '10.0.0.100') return 'PROXY 127.2.2.2:8080;DIRECT';
        return 'DIRECT';
        }
    - 该文件定义了当用户访问 http://Her0in.org/ 时，将不使用任何理服务器直接（DIRECT）访问 URL

    - PROXY 127.2.2.2:8080;DIRECT 指定了使用 127.2.2.2:8080 的 HTTP 代理进行 URL 的访问，如果连接 127.2.2.2:8080 的 HTTP 代理服务器失败，则直接（DIRECT）访问 URL。

- 原理图

    ![](image/WPAD.png)

## WPAD实现方式

- 通过DHCP服务器

    ![](image/DHCP.png)

    - web浏览器通过向DHCP服务器发送 `DHCP INFORM` 查询PAC文件位置

    - DHCP服务器返回 `DHCP ACK` 数据包，包含PAC文件的位置

    - 目前内网中已不再使用DHCP服务器来对客户端的WPAD配置

- 通过DNS查询

    1. 客户端向DNS服务器发起了 `WPAD+X` 的查询请求

    2. DNS 服务器对WPAD 主机的名称进行解析，返回WPAD主机的IP地址

    3. 客户端主机通过WPAD主机的IP地址的80端口下载并解析PAC文件

- 通过LLMNR协议(DNS查询失败)


    - LLMNR名称解析过程

        ![](image/LLMNR_1.png)

    - 缺陷
        1. 整个过程走UDP协议，主机A根本不知道它得到的IP是不是它想要解析的主机的IP地址

        2. 无论局域网中存不存在主机B，只要主机A请求主机B，都会进行一次LLMNR解析过程

- NBNS 查询(LLMNR查询失败)

## WPAD中间劫持原理

-  攻击者将自己加入到组播组中，当主机A发起对主机B的名称解析，使用UDP协议广播，攻击者收到这个请求后，对主机A进行“恶意”应答

- 若主机A的浏览器设置为`自动检测代理`，攻击者宣称自己就是wpad服务器，那么主机A就会从攻击者这里下载攻击者事先准备好的PAC文件，通过攻击者为它“准备”的代理服务器访问url

    ![](image/LLMNR_2.png)

    ![](image/show.png)



### 顺序
    DNS >= LLMNR >= NBNS

## WPAD劫持实验(未成功)

### 工具 ：metaspliot  framework

### 实验环境

- 攻击者  kali IP : 10.0.2.22

- 靶机  win7  IP : 10.0.2.20

    - 开启自动检测设置

    ![](image/auto_detect.png)

### 监听NBNS查询

![](image/monit_nbns.png)

### 设置WPAD服务器

![](image/wpad_server.png)

### 靶机查询WPAD

- 会产生NBNS广播

### wireshark抓包分析

- 靶机广播NBNS

    ![](image/find_wpad.png)

- 攻击主机“回复”

    ![](image/attack_response.png)

- 靶机从攻击主机回复的IP处请求下载PAC文件

    ![](image/request_pac.png)

    - request URI 处为空白，未能获得PAC文件的地址,目标端口并没有错

- 攻击主机PAC文件

    ![](image/response_pac.png)


    

    









    


