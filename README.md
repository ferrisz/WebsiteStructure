# 如何让你的网站快如飞
##服务器   VPS   空间的介绍
我们在做一个网站的时候，一般会用到这三个东西，但是在买的时候不知道这三个东西有什么区别
###服务器
如果你买一台服务器去托管到机房里，你租用他的空间、带宽、公用IP，这是一台真真正正的服务器，通常称为服务器，也是最贵的
###VPS
如阿里云，腾讯云这种云服务器，以及之前的IDC卖的称为VPS的东西
一台服务器只租给一个用户用，如果内存本来16G但是只用了一两个G，这样不仅资源浪费，而且商家的利润也不会很大。商家便想出用vsphere，xen或者KVM技术把一台服务器虚拟成多个服务器卖给多个用户，这样用户用的便宜，商家利润也会变高，这之中存在着很大的超量销售，一台服务八核，但是他可能卖掉的是三十二个单核服务器
这样做有几个理由
·大多数网站均为小网站，流量并不大，现在流量大的网站能有几个
·大多数Web应用为展示型东西运算量并不大，CPU很大程度上是浪费的
传统VPS和云服务器的区别
·传统VPS配置是固定的，不支持配置升级，云服务器是高度可配性，支持配置升级
·传统VPS只在独立的服务器上进行虚拟，云服务器是在服务器集群上进行虚拟，他的硬盘并不与他的CPU内存在同一台服务器
###空间（虚拟主机）
适用于asp,php,asp.net这类型Web应用，就是在一个服务器上给的一个空间有好的环境。。适合个人博客之类的
###从性能上说，从价格上说
服务器 VPS 空间
服务器 VPS 空间
##网站结构的演变过程
·一台WEB 有一个ip
·dns讲域名解析到ip
·db从web服务器分离
·增加web服务器
·做负载均衡
·为防止负载均衡服务器宕机，再增加负载均衡服务器
·正常使用一台负载均衡服务器进行分发，当其宕机时自动切换到另一台负载均衡服务器
·如果取得数据都是静态资源的话为什么每次都要到web服务器上取数据，所以在负载均衡和Web之间又加了一层缓存Squid,当收到请求时先到缓存上找有就直接返回，没有再会请求web，请求web取回时会在自己上备份一份，当再有相同请求时就会直接返回
·如果数据库宕机则网站也都打不开，所以增加数据库备份，mysql的ab复制或者主从复制
·类似web的缓存，当请求多时在web和数据库层也可以加缓存，便用到了memcached，降低主数据库压力

分层网站架构
·负载均衡层 LVS Nginx的upstream反向代理
·缓存层 Squid varnish
·Web层
·数据库缓存层
·数据库层
·存储层 （NFS,NAS,SAN）

##常用的网站优化方法
###架构的调整
###硬件的优化
###应用程序本身的优化 lnmpa
```
server {
        server_name   www.iustu.com ;

        listen       80;

        if ($http_transfer_encoding ~* chunked) {
                return 444;
        }

        gzip on;

        if (-d $request_filename) {

                rewrite ^/(.*)([^/])$ $scheme://$host/$1$2/ permanent;
        }

        error_log               /home/hosts_log/1_error.log notice;
        access_log              /home/hosts_log/1_access.log lu_access_log_1 buffer=32k;

         error_page 403 /__ErrorFiles__/403.html;
         error_page 404 /__ErrorFiles__/404.html;

         error_page 500 /__ErrorFiles__/500.html;
         error_page 501 /__ErrorFiles__/501.html;
         error_page 502 /__ErrorFiles__/502.html;

         root   /home/ftp/1520/www_iustu_com-20160508-gxW/www.iustu.com/;

         ssi off;
         ssi_silent_errors off;
         ssi_types text/shtml;

        location / {

                 index  index.html index.htm index.shtml index.php;
                 autoindex      off;
        }

        location /nginx_status {
                stub_status on;
                access_log off;
        }

        #------------
         location ~* ^(.+)\.(php[3-9]?|phtm[l]?)(\/.+)?$ {
                 set $real_script_name $1.$2;
                 set $path_info $3;

                 if (!-f $document_root$real_script_name) {
                         return 404;
                 }

                  proxy_pass http://1.lum:8889;
                  include proxy_pass.conf;
                  proxy_set_header Host $host;

         }
}
```
###程序的优化
##检测网站的打开速度
chrome network  因为有缓存所以打开快  可以删记录再测试
taobao异步加载
360网站测速
##域名，DNS解析原理
NS
FQDN 完全合格主机域名 主机头+域名
监控服务
TTL 递归

##CDN原理 内容分发网络
访问很慢 地域限制 访问很慢  南电北网
源站 
CDN节点 一个地域多台节点 选负载低的
就近去选择你所需要的网页内容

国内做CDN的很多
老牌 蓝汛 网宿 按带宽收费 比较贵
后起之秀 又拍云 阿里云 青牛 七牛云 结合对象存储

实现原理  智能DNS+若干节点

双向加速
移动互联网时代的网络复杂，且存在南北互通问题。七牛遍布全球的1000多个加速节点，自动选择离用户最近的节点，使得数据的上传下载速度均得到最优化。相较传统IDC，访问速度提升数倍。

断点续传
对上传和下载的双向断点续传支持，即使在极差的移动网络中照样可以读写数据，从而保障用户体验。

降低回源率
使用七牛的镜像存储对源站资源（文件/图片等）进行存储后，即可使用七牛内置的CDN加速分发访问，还可以有一个很低的回源率。

静态加速

针对如各类型图片、mp3、mp4、css、js 文件等静态资源，提供缓存控制、文件预热、代码压缩、镜像迁移、负载均衡等功能，快速加载资源，显著提升网站 60% 以上的响应速度。

动态加速

智能判断 php、jsp、asp.net、python 文件等动态资源，自动实现动静分离，通过全网智能选路、缓存控制等功能，将源站内容分发至全国所有的节点，提高访问的响应速度与网站的可用性，实现用户全站加速更快捷。

性能监控

提供专属管理后台，实时呈现带宽流量、请求次数、全站健康度、下载速度、缓存命中率等指标数据，用户随时监控网络性能，轻松掌控加速大局。

下载加速

针对影音、应用、游戏客户端等大文件类用户, 提供覆盖电信、移动、联通、教育网和各地方运营商网络的加速节点。智能适配 2G/3G/4G/WIFI 等网络环境，加速下载各类文件，节约带宽成本，提升下载速度和用户体验。 





