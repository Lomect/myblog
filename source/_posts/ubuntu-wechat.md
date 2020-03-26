---
title: ubuntu_wechat
date: 2020-03-26 22:30:18
tags: 
    - Ubuntu
    - wine 
---

## 克隆代码

```shell script
 git clone https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu.git wine
```

## 安装
```shell script
./install.sh
```

## 下载
```shell script
wget http://mirrors.aliyun.com/deepin/pool/non-free/d/deepin.com.weixin.work/deepin.com.weixin.work_2.4.16.1347deepin0_i386.deb
```

## 安装软件

```shell script
sudo dpkg -i deepin.com.weixin.work_2.4.16.1347deepin0_i386.deb
```

## 常见问题
1. 中文乱码
```shell script
sudo vim /opt/deepinwine/tools/run.sh
```
修改`WINE_CMD="LC_ALL=zh_CN.UTF-8 deepin-wine"`

