#如何用最低的成本转播PlayStation 4到国内的直播平台

自从肉壳和基友团晚上PS4的GTA之后，就一直想要试试把我们的游戏和语音直播到网上去，但是由于网络条件的限制，只能悻悻作罢。不过现在我的网络条件好了很多，所以这个想法就又蹦出来了。所以刚刚搞了6个小时，终于把我的信号推去了斗鱼上……

http://www.douyutv.com/rokeer

PS4自带的转播功能其实也是很强大的，可以直接转播到Twitch和另外两个不存在的网站上……不过，想转播到国内网站上就没这么容易了，一个比较直观的解决方法就是用HDMI采集卡，然后把采集卡收集到的信息Push去stream上去。但是HDMI采集卡价格从几百块到上千块，真的是贵的买不起，便宜的不敢买。所以只能找找替代方案，查了很多资料，看了很多网站，终于找到了一个相对比较实惠的解决方案。就是通过PS4自带的直播到Twitch上的功能，通过某些手段，劫持到视频流信号，然后再推到斗鱼上面去。想实现这个功能，你只额外需要一件物品：

树莓派 2

如果你不知道什么是树莓派也没关系，其实就是一台廉价的小电脑，只要不到40美元就可以买到一台，刷个Linux系统进去，接上显示器就可以用了。当然，如果你用电脑的话，可以装Linux进去也可以，或者虚拟机装Linux貌似也可以。Windows的话，直接拉到最下面看连接，我自己没试过。

刷系统什么的不在今天的讨论范围内，直接开搞。首先，要安装NginX，负责接受视频流和推送。打开命令行执行下列命令

sudo apt-get -y install nginx 
sudo apt-get -y remove nginx 
sudo apt-get clean

首先安装nginx包，再删掉，目的是为了获得nginx的启动文件，然后还要手动清除/etc/nginx文件夹里面的内容。因为后面我们要用make去安装新版本的nginx，所以不清除原来的内容的话，新的文件是不会覆盖旧的文件的。

sudo apt-get update 
sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev

安装必要的组建，如果有问题的话，可以再执行

sudo apt-get install -y curl build-essential libpcre3-dev libpcre++-dev zlib1g-dev libcurl4-openssl-dev lib

不过我在安装的时候，会发生找不到lib，以及后续找不到openssl的问题，所以还是建议只执行上面的安装。

cd /usr/scr 
sudo git clone git://github.com/arut/nginx-rtmp-module.git 
sudo wget http://nginx.org/download/nginx-1.8.0.tar.gz 
sudo tar xzf nginx-1.8.0.tar.gz 
cd nginx-1.8.0

我们把自己下载的nginx 1.8和rtmp的module都放在了/usr/src文件夹下，你也可以放在别的地方，但是可能会影响到其他命令的执行，到时候要记得自己手动修改目录

./configure –prefix=/var/www –sbin-path=/usr/sbin/nginx –conf-path=/etc/nginx/nginx.conf –pid-path=/var/run/nginx.pid –error-log-path=/var/log/nginx/error.log –http-log-path=/var/log/nginx/access.log –with-http_ssl_module –without-http_proxy_module –add-module=/usr/src/nginx-rtmp-module

注意，可能需要修改add-module的目录，如果你吧rtmp module放在其他的文件夹下了的话，然后执行

sudo mkdir -p /var/www

创建一个文件夹，用于放置一些网页文件

sudo make

这个时间会比较久，大概几分钟

sudo make install

安装，之后可以尝试执行

nginx -v 
sudo service nginx start 
sudo service nginx stop

查看nginx版本，启动服务，关闭服务。如果没有报错，就应该是已经安装上了。然后修改nginx的配置文件，位置应该是/etc/nginx/nginx.conf

user root; 
#Root is only OK if the server is not public. Otherwise you need to increase security on your own. 
# user www-data; 
#use up to 4 processes if you expect allot of traffic. But this causes issues with rtmp /stat page and possibly pushing/pulling 
#worker_processes 4; 
worker_processes 1;

events { 
worker_connections 1024; 
}

http { 
include /etc/nginx/mime.types; 
default_type application/octet-stream; 
sendfile on; 
keepalive_timeout 65; 
#if you want gzip enabled 
#gzip on; 
#gzip_disable “msie6”;

server { 
listen 80; 
server_name localhost;

# sample handlers 
#location /on_play { 
# if ($arg_pageUrl ~* localhost) { 
# return 201; 
# } 
# return 202; 
#} 
#location /on_publish { 
# return 201; 
#} 
#location /vod { 
# alias /var/myvideos; 
#} 
# rtmp stat 
location /stat { 
rtmp_stat all; 
rtmp_stat_stylesheet stat.xsl; 
}

location /stat.xsl { 
# you can move stat.xsl to a different location 
root /usr/src/nginx-rtmp-module; 
}

# rtmp control 
location /control { 
rtmp_control all; 
} 
error_page 500 502 503 504 /50x.html; 
location = /50x.html { 
root html; 
} 
} 
} 
rtmp { 
server { 
listen 1935; 
chunk_size 131072; 
max_message 256M; 
ping 30s; 
notify_method get; 
application app{ 
live on;

# You can push this stream to an external rtmp server while accessible locally. 
# If you experience artefacts and delays on external server lower the bitrate. 
# There seems to be a bug. When watching local stream and pushing to remote, the remote 
# stream become really weird with random blocks and strange shadows.(consider 1 one for now) 
# push rtmp://ip-address-external-rtmp/app/stream;

# sample play/publish handlers 
#on_play http://localhost:80/on_play; 
#on_publish http://localhost:80/on_publish; 
# sample recorder 
#recorder rec1 { 
# record all; 
# record_interval 30s; 
# record_path /tmp; 
# record_unique on; 
#} 
# sample HLS 
#hls on; 
#hls_path /tmp/hls; 
#hls_sync 100ms;

} 
# Video on demand 
#application vod { 
# play /var/Videos; 
#} 
# Video on demand over HTTP 
#application vod_http { 
# play http://localhost:80/vod/; 
#} 
} 
}

没有缩进，好丑……不过无所谓，这样配置就好了。其他的不多说，有两点要注意一下，第一是在rtmp里，有一行application app，后面的app一定不要改成其他名字，因为这是Twitch的视频流的推送标识（可能也不是这个词，总之不要改），其次，这个配置只是获取视频流并转发，后面我们可以用OBS来抓去并重新推送，当然，你也可以直接推送到斗鱼上去，只要application app里面live on;下面添加

push rtmp://send.douyu.tv/live/[STREAMKEY];

上面是斗鱼直播地址，[STREAMKEY]改成你的直播码。

然后记得

sudo service nginx start

启动服务，nginx就配置好了。这时候，如果你用浏览器直接访问你的服务器，比如http://192.168.0.16/stat 应该就可以看到一些内容，未来我们需要使用这个页面的内容来获取视频流相关的信息。至于如何获取服务器ip地址？ifconfig一下就知道了。

备注，按理说，机器重启之后，应该手动重启服务，或者写脚本自动启动，但是我在使用的时候，发现不手动重启也OK，所以就先不写教程了，后面的dnsmasq也有同样的问题。

然后我们开启系统的转发功能，修改/etc/sysctl.conf文件，设置net.ipv4.ip_forward=1，并执行

sysctl -p /etc/sysctl.conf

让更新生效。然后我们安装dnsmasq，执行

sudo apt-get install dnsmasq

就OK了。然后修改配置文件/etc/dnsmasq.conf，在相应位置添加，

address=/live.twitch.tv/192.168.0.16

live.twitch.tv是twitch的推送地址，192.168.0.16是我的服务器地址，不过只做这一步已经没有用了，而且我本人来讲，也不知道这一步是否是必须的，不过我做了，没有出现问题，所以就没有试不做会怎样。

sudo service dnsmasq start

启动服务就好了。

不过由于twitch现在修改了服务，所以只配置上面的dns解析是不够的，所以我们要执行

nslookup live.twitch.tv

可以查看live.twitch.tv都解析到哪里了。如果使用不了nslookup，输入

sudo apt-get install dnsutils

就可以安装nslookup命令了。然后，配置路由表的，用于把ip地址直接转发到服务器，因为我本人不是很懂iptables的配置，所以在网上找了三条命令。

sudo iptables -t nat -A PREROUTING -d 199.9.248.0/21 -p tcp –dport 1935 -j DNAT –to-destination 192.168.0.16:1935 
sudo iptables -t nat -A PREROUTING -d 199.9.0.0/16 -p tcp –dport 1935 -j DNAT –to-destination 192.168.0.16:1935 
sudo iptables -t nat -A POSTROUTING -j MASQUERADE

前两条的作用是把ip地址转到我的服务器，最后一条是把结果转回去。我试了各种命令都不对，就在我快要放弃的时候，试了一下只输入

sudo iptables -t nat -A PREROUTING -d 199.9.248.0/21 -p tcp –dport 1935 -j DNAT –to-destination 192.168.0.16:1935

竟然成功了。如果你有兴趣，或者你比较懂的话，可以试试修改这个命令，我就懒得改了。另外，iptables的配置，重启服务器后，肯定要重新配置的，这个没有疑问。

至此，服务器设置完成。

然后我们只要手动配置PS4获取ip地址，子网掩码，网关和dns服务器都设置成咱们自己的服务器也就是192.168.0.16就可以了。

然后，我们就可以使用Twitch，推送我们的视频信号了，具体操作不讲了。等看到屏幕右上角的On Air就表示已经在推送了。这时候，我们回到http://192.168.0.16/stat ，刷新一下就可以看到我们的nginx接受到了视频流的推送。

然后我们就可以用OBS接收这个视频流，地址格式是 rtmp://192.168.0.16/app/livexxxxxxxxxx，这个livexxxxxxxx就是你在http://192.168.0.16/stat看到那串码。然后你就可以用OBS，处理你的视频并推送到斗鱼直播上了。

就这么点事情，我前前后后搞了6个小时，不过终于搞定了，也是不错，如果你有什么问题，欢迎留言。转载请标明出处。

最后，我要感谢以下网页的作者，没有你们，我也不可能完成这件事，多亏了以你们的文章，我才能解决这些棘手的问题。Thanks！

http://bbs.a9vg.com/thread-4160686-1-1.html

http://bbs.a9vg.com/thread-4199530-1-1.html

http://pkula.blogspot.com/2013/06/live-video-stream-from-raspberry-pi.html

http://yiqingsim.net/post/103165692292/setting-up-dnsmasq-as-a-dnsdhcp-server-on-a

https://phelps.io/local-ps4-streaming/

目前这篇文章，没有排版，没有插图，有时间再做吧，已经凌晨3点了，睡觉了……

http://www.douyutv.com/

http://www.douyutv.com/