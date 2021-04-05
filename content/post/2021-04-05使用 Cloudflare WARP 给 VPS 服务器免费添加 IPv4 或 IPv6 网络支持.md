---
title: 使用 Cloudflare WARP 给 VPS 服务器免费添加 IPv4 或 IPv6 网络支持
summary: netsurf
date: "2021-04-05T00:00:00Z"
subtitle: vps 加速
feature: true
categories: ["vps"，“cloudflare”]
publishDate: 2021-04-05
draft: false

# View.
#   1 = List
#   2 = Compact
#   3 = Card
view: 2

---
##前言

以 Debian 10 为例，介绍如何搭建 WireGuard 后，采用warp技术，通过连接到cloudflare的边缘节点，实现链路优化，连接到warp来获取额外到网络连通性支持。
##安装wireguard

‘’‘
apt update
apt install curl sudo lsb-release -y
’‘’
###添加backports源：
‘’‘
sudo sh -c "echo 'deb https://deb.debian.org/debian buster-backports main contrib non-free' > /etc/apt/sources.list.d/buster-backports.list"
’‘’

###安装软件包
'''
sudo apt update
sudo apt -t buster-backports install wireguard -y
'''
###使用 wgcf 生成 WireGuard 配置文件
wgcf 是 Cloud­flare WARP 的非官方 CLI 工具，它可以模拟 WARP 客户端注册账号，并生成通用的 Wire­Guard 配置文件。
安装wgcf
'''
curl -fsSL git.io/wgcf.sh | sudo bash
'''
注册 WARP 账户 (将生成 wgcf-account.toml 文件保存账户信息)
'''
wgcf register
'''
生成 Wire­Guard 配置文件 (wgcf-profile.conf)
'''
wgcf generate
'''
生成的两个文件记得备份好，尤其是 wgcf-profile.conf，万一未来工具失效、重装系统后可能还用得着。
###编辑 WireGuard 配置文件wgcf-profile.conf

为 IPv4 Only 服务器添加 IPv6 网络支持
将配置文件中的 engage.cloudflareclient.com 替换为 162.159.192.1，并删除 AllowedIPs = 0.0.0.0/0。
###启用 WireGuard 网络接口
将 Wire­Guard 配置文件复制到 /etc/wireguard/ 并命名为 wgcf.conf。
'''
sudo cp wgcf-profile.conf /etc/wireguard/wgcf.conf
'''
开启网络接口（命令中的 wgcf 对应的是配置文件 wgcf.conf 的文件名前缀）。
wg-quick up wgcf
执行执行ip a命令，此时能看到名为wgcf的网络接口
执行以下命令检查是否连通。同时也能看到正在使用的是 Cloud­flare 的网络
# IPv4 Only VPS
curl -6 ip.p3terx.com
# IPv6 Only VPS
curl -4 ip.p3terx.com
测试完成后关闭相关接口，因为这样配置只是临时性的。
sudo wg-quick down wgcf
正式启用 Wire­Guard 网络接口
# 启用守护进程
sudo systemctl start wg-quick@wgcf
# 设置开机启动
sudo systemctl enable wg-quick@wgcf
###Cloudflare WARP 网速测试
目前暂时还未找到很好的方法来测试指定网络接口的速度，而目前已知 speedtest.net 提供的测速是 IPv4 Only 的，那么就可以使用其提供的 CLI 工具在 IPv6 Only 的 VPS 上来测试通过 WARP 访问外部网络的极限网速。
安装 Ookla Speedtest CLI
curl -fsSL git.io/speedtest-cli.sh | sudo bash
执行speedtest命令测速。
