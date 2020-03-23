---
title: charles
date: 2020-03-24 00:04:07
tags: 
    - Test
---
## 第一步
Charles官网下载deb包，用sudo dpkg -i命令安装，或者按照说明将其添加到软件源，用sudo apt install就能安装，这一点不赘述；

## 第二步
下载Charles的SSL根证书：Help -> SSL Proxying -> Install Charles Root Certificate，安装后，应该能在home目录的.charles/ca/中找到charles-proxy-ssl-proxying-certificate.cer和charles-proxy-ssl-proxying-certificate.pem两个文件，对这种证书文件格式感兴趣的化，自行问搜索引擎，因为我们这里只是配置，就不补充这个知识了；

## 第三步
转换格式：终端中执行openssl x509 -outform der -in charles-proxy-ssl-proxying-certificate.pem -out charles-proxy-ssl-proxying-certificate.crt；

## 第四步
在/usr/share/ca-certificates文件夹下新建一个目录charles，再将转换格式后得到的证书charles-proxy-ssl-proxying-certificate.crt复制到/usr/share/ca-certificates/charles 中；

## 第五步 
在/etc/ca-certificates.conf这个配置文件的最后追加charles/charles-proxy-ssl-proxying-certificate.crt，用sudo update-ca-certificates更新证书，完成后发现/etc/ssl/certs目录中应该多了一个charles-proxy-ssl-proxying-certificate.pem文件，表示成功；

## 第六步
配置google浏览器，这时候你需要一个叫SwitchyOmega的代理管理插件，把http代理协议端口写成charles默认的8888，ip就填本地ip就好，另外在google浏览器的设置->高级->证书管理->授权中心中添加第四步中复制过的charles-proxy-ssl-proxying-certificate.crt证书，可以重复添加，会提示该证书已添加过，由此证明已经添加成功；

## 第七步 
在Charles中Proxy -> SSL Proxy Settings -> SSL Proxy中设置一个Host为*,Port为443的Location，主要是用来代理所有的HTTPS请求；

## 第八步
以上完成的话，基本大功告成，在google浏览器中用SwitchyOmega切换至设置的charlse 8888端口的代理，访问一个网站试一试，应该就可以看到Charles能成功抓取HTTP的各种信息了；