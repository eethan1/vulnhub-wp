1.發現主機
```bash
netdiscover -r 10.2.0.1/24
```
得到 10.2.0.15

2.掃描
```bash
nmap -A -T5 10.2.0.15
```
得到 22, 80, 443 port

3.查看 http(s)://10.2.0.15 

出現 apache 預設頁面
用 dirb 掃出 /uploads, /test， test/index.html 顯示有張圖 keys2.jpg ，試著掃描其他圖然並卵

從 nmap 結果可看到 ssl-cert 裡域名 weakness.jth
在 /etc/hosts 把 weakness.jth map 到 10.2.0.15就看到正常頁面

_出現 10.2.0.15 憑證有問題可能代表域名解析出錯_

4.掃目錄
```bash
dirb http://weakness.jth/
```
得到 /private/files，有兩個檔案 mykey.pub、 notes.txt，載到本地

5.使用 mykey.pub

mykey.pub 應該是 ssh 登入公鑰， notes.txt 提示 使用 openssl 0.9.8c-1 產生

搜尋 openssl 0.9.8c-1 漏洞，除了 google 外可以用
```bash
searchsploit openssl 0.9.8
```
找到 [CVE-2008-0166](https://www.cvedetails.com/cve/CVE-2008-0166/)，因為某次更新刪了幾行程式碼意外導致初始化 seed 也被刪除，每次都用同一組，因此可被預測

查了個 [repo](https://github.com/g0tmi1k/debian-ssh)，找到對應的 private key

6.ssh 登入

仔細看 public key 最後面 root@targetcluster，應該就是從那個 repo 複製下來的，用 root 登入失敗，然後很通靈的找到用戶名在 weakness.jth/index.html 小兔子旁 n30 ...

7.提權

group 得知 n30 可用 sudo，因此得到n30密碼就行。n30 家目錄下有個 code 是 python2 byte code 檔案，用 uncompyle6 逆向回去得到 n30:dMASDNB!!#B!$!$33，最後在 root 家目錄拿到 flag

題外話，uncompyle6 兼容 python2 和 3，所以相乘後得6的樣子...(python2 有 uncompyle2)