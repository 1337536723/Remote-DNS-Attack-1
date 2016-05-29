# Remote-DNS-Attack

远程DNS攻击是指攻击者和受害者不在同一个局域网下，攻击者无法嗅探到受害者发出的包。不过在这个项目中，没有刻意去制造这样的环境，攻击者、受害者、DNS服务器实际上都在同一局域网下的，只是我们默认攻击者不可以像上一节的本地攻击一样嗅探到受害者发出的包。

## 配置

### 网络配置

同样是用三台虚拟机，一台Apollo(本地DNS服务器，同时也是受害DNS服务器，Victim DNS server)，一台用户(User，或者说客户端)，一台攻击者(Attacker)，下文中统一称呼都是Apollo，用户和攻击者。 不过我这次没有配置静态IP，所以实验时IP地址会与下图不一致。依次是192.168.153.130、192.168.153.133和192.168.153.129。

![组网](https://raw.githubusercontent.com/familyld/Remote-DNS-Attack/master/graph/image28.png)

### 配置本地DNS服务器
第一步：安装bind9服务器，由于SEEDUbuntu已经预装了，所以无须操作。

第二步：修改配置文件named.conf.options，这个配置文件是DNS服务器启动时需要读取的，其中dump.db是用来转储DNS服务器的缓存的文件。

![local server](https://raw.githubusercontent.com/familyld/Local-DNS-Attack/master/graph/image11.png)

特别地，以下两条命令可以帮助我们更好地进行实验：

![local server](https://raw.githubusercontent.com/familyld/Remote-DNS-Attack/master/graph/image29.png)

其中第一条命令是把DNS缓存清空，第二条命令是把缓存转储到dump.db文件。

第三步：移除example.com zone，前面的本地DNS攻击实验是假设我们拥有example.com域名并负责其应答，而在这次的远程DNS缓存投毒攻击实验中，本地DNS服务器不负责这件事情，example.com的应答交给互联网上专门的example.com DNS 服务器来负责。这里把原来加入到named.conf文件的zone删掉即可。

第四步：启动DNS服务器

![bind9](https://raw.githubusercontent.com/familyld/Local-DNS-Attack/master/graph/image15.png)

或者使用sudo service bind9 restart 语句也可以，至此本地DNS服务器配置完毕。

### 配置客户端

在客户端我们需要配置使得Apollo (192.168.153.130) 成为客户端(192.168.153.133)的默认DNS服务器，通过修改客户端的DNS配置文件来实现。

第一步：修改etc文件夹下的resolv.conf文件，加入红框内的内容。

![client](https://raw.githubusercontent.com/familyld/Remote-DNS-Attack/master/graph/image30.png)

注意，因为Ubuntu系统中，resolv.conf会被DHCP客户端覆写，这样我们加入的内容就会被消除掉，为了避免这个状况我们要禁止DHCP。

第二步：禁止DHCP

![client](https://raw.githubusercontent.com/familyld/Local-DNS-Attack/master/graph/image17.png)

点击All Settings，然后点击Network，在Wired选项卡点击Options按钮，然后在IPv4 Settings选项卡中把Method更改为Automatic(DHCP) addresses only，然后把DNS servers更改为前面设置的本地DNS服务器的ip地址。

因为没有找到实验指导中Network Icon的Auto eth0选项，所以这里直接手动从命令行重启一下网卡：

![client](https://raw.githubusercontent.com/familyld/Remote-DNS-Attack/master/graph/image31.png)

为了确保万无一失，重启一次虚拟机来使修改生效也行。再查看一下此时的DNS服务器，确实就我们刚刚配置的，DHCP被禁止了，没有对我们的修改进行覆盖，这样就证明配置成功了。

![client](https://raw.githubusercontent.com/familyld/Remote-DNS-Attack/master/graph/image32.png)

## 配置攻击者

攻击者并没有需要特别配置的地方，它可以是网络上任意一台主机，我们可以通过raw socket编程来伪造DNS包进行攻击。


