---

title: Hetzner Online dedicated server 安装emby-server
summary: 使用hetzner的dedicated server搭建emby的过程
authors: [bush]
tags: [网络应用]
categories: [emby]
date: "2020-10-26T00:00:00Z"
slides:
theme: black
highlight_style: dracula
reading_time: false  # Show estimated reading time?
share: true  # Show social sharing links?
profile: false  # Show author profile?
commentable: true  # Allow visitors to comment? Supported by the Page, Post, and Docs content types.
editable: false  # Allow visitors to edit the page? Supported by the Page, Post, and Docs content types.
header:
  caption: ""
  image: "static/media/11.png"
---

---
进来由于emby服务器播放谷歌gdrive比较火，正好赶上科学家，z大各位大神，先行各种踏坑，我也记录一下个人使用hetzner的dedicated server搭建emby的过程。

### 1, 搭建环境的的选择

我搭建的目的是在墙内播放谷歌gdrive的片源，cpu能硬解4k，机器选择了hetzner的1275v5，拍卖获得，核显是p530，基本上能硬解播放1080p和4k，hevc格式，cpu占用单个用户播放，cpu占用不会超过100%，这样，减轻了服务端解码的压力。只要客户端建立合适的fd线路，基本流畅播放。

操作系统我选择的debian 10.3，hetzner提供的，64g内存，480g*2ssd硬盘，性能有点过剩。但是没有太合适的了。

### 2，安装emby

 访问 https://emby.media/linux-server.html
 ```bash
 wget https://github.com/MediaBrowser/Emby.Releases/releases/download/4.5.2.0/emby-server-deb_4.5.2.0_amd64.deb
 dpkg -i emby-server-deb_4.5.2.0_amd64.deb
 ```
 Open a web browser to http://ip:8096
设置管理员账户

### 3，安装rclone
```bash
curl https://rclone.org/install.sh | sudo bash
```
安装最新版本rclone，配置挂载团队盘，在此有两种选择：
a，将谷歌团队盘挂载本地目录上，用emby自行削刮
b，采用科学家大佬提供的削刮好的数据包（你需要有科学家团队盘的权限），下载解压使用现成的削刮元数据，可以节省大量时间，削刮好的元数据大约60-100g左右。
需要注意的是，采用削刮好的数据，你的团队盘的命名和挂载目录的命名，应该和科学家的命名一样，以减少错误。
科学家的挂载 [教程](https://blog.vwert.com/CloudStorage/Emby-GoogleDrive.html)

举例：

```bash
mkdir -p /mnt/EmbyMedia/MOVIE #为EmbyMedia_VIP1_Movie盘创建挂载点
rclone mount EmbyMedia_VIP_Movie_play: /mnt/EmbyMedia/MOVIE --umask 0000 --default-permissions --allow-non-empty --allow-other --buffer-size 32M --dir-cache-time 12h --vfs-read-chunk-size 64M --vfs-read-chunk-size-limit 1G & #挂载EmbyMedia_VIP1_Movie盘
```

但是，rclone mount 参数可以适当改变

--allow-other --uid 1000 --gid 1000 --umask 022 --default-permissions --allow-non-empty --allow-other --dir-cache-time 12h --poll-interval 5m --buffer-size 128M --vfs-read-chunk-size 32M --vfs-read-chunk-size-limit 1G

--vfs-read-chunk-size这个减少一点削刮会快一点

挂载完毕后

```bash
df -h
```

查看磁盘容量，看是否挂载成功

### 4，削刮元数据

如果是使用科学家大佬的团队盘，继续按照科学家教程走一遍即可，如果想自己削刮，需要注意的是：

不论电影还是电视连续剧的削刮，都需要使用themoviedb数据库，因为大家在一起维护数据库，修改错误。

hash值匹配，选择其他的选项，选择越多，数据越全，但是削刮速度也变慢，每个人根据情况选择。因为，电影电视总共10多万部。削刮是非常缓慢的过程。

科学家大佬的削刮好的数据包，还发现另外一个问题，就是削刮选项里有往源文件目录写入字幕，nfo等文件，但我们的权限常常是viewer，而非manager，所以，还应该把写入的选项去掉为好，读取为主！

### 5，转码  transcoding

转码是emby服务器，premiere提供的选项，如果是的话，你需要让核显运行才能显示解码器。如果，你的转码选项里没有出现

***Preferred Hardware Decoders***

***Preferred Hardware Encoders***

说明你的核显没有正常工作