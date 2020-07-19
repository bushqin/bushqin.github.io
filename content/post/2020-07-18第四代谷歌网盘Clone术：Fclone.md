---
title: 第四代谷歌网盘Clone术：Fclone
summary: drive转存技术从最初的rclone转存到目前到fclone进化了4代，本文简单介绍mark，备忘
authors: [转自BlueSkyXN]
date: "2020-07-18T00:00:00Z"
subtitle: 
feature: true
categories: ["gdrive"]
publishDate: 2020-07-18
draft: false
---


# 第四代谷歌网盘Clone术：Fclone

## 简介



Fclone是目前最新最强的Rclone衍生产品
与rclone和gclone兼容
同时与gclone相比,能有几十上百倍的转存速度
常规转存速度也达50+files/s
相当于装满一个团队盘(40W文件),单开的情况下,只需要135分钟左右
如果将参数拉高，使用性能较好的机器，速度可达150F/s，此速度下一个盘还不要一个小时
而且如果sa够多，还能无限拉升，不怕不够快，就怕你sa不够冲
可以说是几十上百倍的提升,而且稳定性大幅提升,不会受到众多小文件影响
而且还支持参数调整,可以充分利用高配主机的性能
所有前代产品均通用,无须额外修改,安装新程序后换个命令就能用



### Rclone（第一代）的安装

# 

官方教程：https://rclone.org/install/
Linux一键脚本

```bash
curl https://rclone.org/install.sh | sudo bash
```



### Autoclone (2nd generation)的安装并获取sa

项目地址：

```bash
https://github.com/xyou365/AutoRclone
```



### 安装方法：

---

#### Step 1. Copy code to your VPS or local machine

**For Linux system**: Install [screen](https://www.interserver.net/tips/kb/using-screen-to-attach-and-detach-console-sessions/), `git` and [latest rclone](https://rclone.org/downloads/#script-download-and-install). If in Debian/Ubuntu, directly use this command

```bash
sudo apt-get install screen git && curl https://rclone.org/install.sh | sudo bash
```

```bash
sudo git clone https://github.com/xyou365/AutoRclone && cd AutoRclone && sudo pip3 install -r requirements.txt
```

#### Step 2. Generate service accounts [What is service account](https://cloud.google.com/iam/docs/service-accounts) [How to use service account in rclone](https://rclone.org/drive/#service-account-support)

Enable the Drive API in [Python Quickstart](https://developers.google.com/drive/api/v3/quickstart/python) and save the file `credentials.json` into project directory



''Note: 1 service account can copy around 750gb a day, 1 project makes 100 service accounts so thats 75tb a day, for most users this should easily suffice. ''

```bash
python3 gen_sa_accounts.py --quick-setup 1
```

or

```bash
python3 gen_sa_accounts.py --quick-setup 1 --new-only
```

note: 这将覆盖现存的一个项目

#### Step 3. Add service accounts to Google Groups (Optional but recommended for hassle free long term use)



[Official limits to the members of Team Drive](https://support.google.com/a/answer/7338880?hl=en) (Limit for individuals and groups directly added as members: 600)

对于 **Gsuite Admin** 管理员

1. Turn on the Directory API following [official steps](https://developers.google.com/admin-sdk/directory/v1/quickstart/python) (save the generated json file to folder `credentials`).

2. Create group for your organization [in the Admin console](https://support.google.com/a/answer/33343?hl=en). After create a group, you will have an address for example`sa@yourdomain.com`.

3. run
   
   ```bash
   python3 add_to_google_group.py -g sa@yourdomain.com
   ```
   
   --- 
   
   #### 对于普通用户

4. Create [Google Group](https://groups.google.com/) then add the service accounts as members by hand. Limit is 10 at a time, 100 a day but if you read our warning and notes above, you would have 1 project and hence easily in your range.
   
   #### Step 4. Add service accounts or Google Groups into Team Drive
   
   #### Step 5. Start your task
   
   ```bash
   python3 rclone_sa_magic.py -s SourceID -d DestinationID -dp DestinationPathName -b 1 -e 600
   ```
   
   Add `--disable_list_r` if `rclone` [cannot read all contents of public shared folder](https://forum.rclone.org/t/rclone-cannot-see-all-files-folder-in-public-shared-folder/12351)
   
   rclone --config rclone.conf size --disable ListR src001:





### Gclone(第三代)的安装和使用

#### 安装方法

---

linux一键脚本：



```bash
bash <(wget -qO- https://git.io/gclone.sh)
```

```bash
gclone --version
```

```bansh
[gc]
type = drive
scope = drive
service_account_file = /root/AutoRclone/accounts/0079098d8d94d71d7a3042da3838a4d07346eb78.json
service_account_file_path = /root/AutoRclone/accounts/
root_folder_id = root
```

转存用法

---

```bash
gclone copy gc:{【源盘】ID} gc:{【目的盘】ID} --drive-server-side-across-configs
```

大小检查

---

```bash
gsize gc:{【目的盘】ID}
```



### fclone(第四代)的安装使用

项目地址：

---

https://github.com/mawaya/rclone

下载地址：

---

https://github.com/mawaya/rclone/releases

安装使用方法：

以0.3.1为例

```bash
wget https://github.com/mawaya/rclone/releases/tag/fclone-v0.3.1
unzip fclone-v0.3.1.zip /usr/bin/fclone
chmod +x /usr/bin/fclone
```

**使用方法**

低配vps

`fclone copy gc:{【源盘】ID} gc:{【目的盘】ID} —drive-server-side-across-configs --stats=1s --stats-one-line -vP --checkers=128 --transfers=128 --drive-pacer-min-sleep=1ms --check-first`

低配vps推荐2

`fclone copy gc:{【源盘】ID} gc:{【目的盘】ID} --drive-server-side-across-configs --stats=1s --stats-one-line -vP --checkers=128 --transfers=256 --drive-pacer-min-sleep=1ms --check-first`

通用

`fclone copy gc:{【源盘】ID} gc:{【目的盘】ID} --drive-server-side-across-configs --stats=1s --stats-one-line -vP --checkers=256 --transfers=256 --drive-pacer-min-sleep=1ms --check-first`

`fclone copy gc:{【源盘】ID} gc:{【目的盘】ID} --drive-server-side-across-configs --stats=1s --stats-one-line -vP --checkers=256 --transfers=256 --drive-pacer-min-sleep=1ms --drive-pacer-burst=5000 --check-first`

中高杜甫推荐

`fclone copy gc:{【源盘】ID} gc:{【目的盘】ID} --drive-server-side-across-configs --stats=1s --stats-one-line -vP --checkers=320 --transfers=320 --drive-pacer-min-sleep=1ms --drive-pacer-burst=5000 --check-first`



参数说明：

---

--drive-server-side-across-configs #服务端传递

 --stats=1s --stats-one-line -vP #日常参数 

--checkers=数字 #越大越快 

--transfers=数字 #越大越快 

--drive-pacer-min-sleep=数字ms #越小越快 

--drive-pacer-burst=数字 #越大越快 

--check-first #fmod必须参数


