---
title: 排错指令合集 - 先别重启, 让我看看
description: 面向个人服务器, 博客, 反代, Docker 与网络问题的常用排障命令速查
date: 2026-06-30 09:40:00+0800
# image: cover.jpg
categories:
    - IT-Techs
tags:
    - Troubleshooting
    - Linux
    - Network
    - Docker
    - Nginx
---

## 机器概况

```bash
hostname
whoami
pwd
date
uptime
uname -a
cat /etc/os-release
nproc
```

```bash
free -h
df -h
df -i
top
```

---

## 服务状态

```bash
systemctl status nginx
systemctl status docker
systemctl list-units --failed
systemctl list-unit-files | grep enabled
```

```bash
journalctl -u nginx -n 100 --no-pager
journalctl -u nginx -f
journalctl -n 200 --no-pager
journalctl -b -n 200 --no-pager
```

```bash
systemctl restart 服务名
systemctl reload 服务名
systemctl enable --now 服务名
```

---

## 进程

```bash
ps aux
ps aux | grep nginx
pgrep -a nginx
pidof nginx
```

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

```bash
kill -TERM <pid>
kill -KILL <pid>
```

---

## 端口

```bash
ss -lntp
ss -lnup
ss -antp
```

```bash
sudo lsof -i :80
sudo lsof -iTCP:3000 -sTCP:LISTEN
sudo fuser -v 80/tcp
```

```bash
netstat -lntp
netstat -lnup
```

```text
127.0.0.1:3000   只监听本机
0.0.0.0:3000     监听所有 IPv4
[::]:3000        监听 IPv6
```

---

## 网络

```bash
ip addr
ip route
ip neigh
resolvectl status
cat /etc/resolv.conf
```

```bash
ping 1.1.1.1
ping example.com
traceroute example.com
mtr example.com
```

```bash
nc -vz example.com 80
nc -vz example.com 443
nc -vz 1.2.3.4 25565
telnet example.com 443
```

---

## 抓包

```bash
sudo tcpdump -ni any port 443
sudo tcpdump -ni any host 1.2.3.4
sudo tcpdump -ni eth0 port 80
```

```bash
sudo tcpdump -ni any 'tcp port 443 and tcp[tcpflags] & tcp-syn != 0'
sudo tcpdump -ni any 'icmp'
sudo tcpdump -ni any 'udp port 53'
```

```bash
sudo tcpdump -ni any -w capture.pcap port 443
```

---

## DNS

```bash
dig example.com
dig example.com A
dig example.com AAAA
dig example.com CNAME
dig example.com MX
dig example.com TXT
dig example.com NS
dig example.com SOA
```

```bash
dig @1.1.1.1 example.com
dig @8.8.8.8 example.com
dig @223.5.5.5 example.com
dig @ns1.example-dns.com example.com
```

```bash
dig +trace example.com
dig +short example.com
dig +nocmd example.com any +multiline +noall +answer
```

```cmd
nslookup example.com
ipconfig /displaydns
ipconfig /flushdns
```

---

## HTTP / HTTPS

```bash
curl -I https://example.com
curl -IL https://example.com
curl -v https://example.com
curl -vk https://example.com
```

```bash
curl -v -H 'Host: example.com' http://1.2.3.4
curl -v --resolve example.com:443:1.2.3.4 https://example.com
curl -v --connect-to example.com:443:1.2.3.4:443 https://example.com
```

```bash
curl -X POST https://example.com/api
curl -H 'Content-Type: application/json' -d '{"hello":"world"}' https://example.com/api
```

```bash
openssl s_client -connect example.com:443 -servername example.com
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -issuer -subject
```

---

## Nginx

```bash
sudo nginx -t
sudo nginx -T
sudo systemctl status nginx
sudo systemctl reload nginx
```

```bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

```bash
sudo nginx -T | grep -n "server_name"
sudo nginx -T | grep -n "proxy_pass"
sudo nginx -T | grep -n "client_max_body_size"
sudo nginx -T | grep -n "listen"
```

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

---

## 防火墙

```bash
sudo ufw status verbose
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

```bash
sudo firewall-cmd --list-all
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --reload
```

```bash
sudo iptables -S
sudo iptables -L -n -v
sudo iptables -t nat -S
sudo nft list ruleset
```

---

## 磁盘

```bash
df -h
df -i
lsblk
mount
```

```bash
du -sh *
du -h --max-depth=1
find /var -type f -size +500M -print
```

```bash
sudo lsof | grep deleted
sudo truncate -s 0 /var/log/big.log
```

---

## 内存 / CPU / IO

```bash
free -h
top
htop
uptime
```

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

```bash
vmstat 1
iostat -xz 1
```

```bash
dmesg | grep -i oom
journalctl -k | grep -i oom
```

---

## Docker

```bash
docker ps
docker ps -a
docker images
docker volume ls
docker network ls
```

```bash
docker logs -f container_name
docker logs --tail 100 container_name
docker inspect container_name
docker port container_name
docker stats
docker system df
```

```bash
docker exec -it container_name sh
docker exec -it container_name bash
docker restart container_name
```

---

## Docker Compose

```bash
docker compose ps
docker compose logs -f
docker compose logs --tail 100 service_name
```

```bash
docker compose up -d
docker compose pull
docker compose restart service_name
docker compose down
```

```bash
docker compose config
docker compose exec service_name sh
docker compose exec service_name bash
```

---

## Git

```bash
git status --short
git status --short --ignored
git diff
git diff --stat
git diff --cached
```

```bash
git log --oneline -10
git log --graph --oneline --decorate --all
git show <commit>
git blame path/to/file
```

```bash
git reflog
git stash push -m "debug temp"
git stash list
git stash show -p stash@{0}
git stash pop stash@{0}
```

```bash
git restore path/to/file
git restore --staged path/to/file
git clean -nd
git clean -fd
```

---

## Windows 网络

```cmd
ipconfig /all
ipconfig /displaydns
ipconfig /flushdns
```

```cmd
ping example.com
tracert example.com
route print
```

```powershell
Test-NetConnection example.com -Port 443
Test-NetConnection 1.2.3.4 -Port 25565
```

```cmd
netstat -ano | findstr :443
tasklist | findstr 1234
```

---

## Windows 端口转发

```cmd
netsh interface portproxy show all
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8080 connectaddress=192.168.1.10 connectport=80
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=8080
```

```cmd
netsh advfirewall firewall add rule name="PortProxy 8080" dir=in action=allow protocol=TCP localport=8080
netsh advfirewall firewall delete rule name="PortProxy 8080"
```

---

## 常用组合

服务打不开:

```bash
systemctl status 服务名
journalctl -u 服务名 -n 100 --no-pager
ss -lntp
curl -v http://127.0.0.1:端口
```

网站打不开:

```bash
dig 域名
curl -IL https://域名
sudo nginx -t
tail -f /var/log/nginx/error.log
```

端口不通:

```bash
ss -lntp
sudo ufw status
sudo tcpdump -ni any port 端口
nc -vz 主机 端口
```

服务器变慢:

```bash
uptime
top
free -h
df -h
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

Docker 异常:

```bash
docker compose ps
docker compose logs --tail 100 服务名
docker port 容器名
docker inspect 容器名
```

Git 状态奇怪:

```bash
git status --short --ignored
git diff --stat
git log --oneline -10
git reflog
```
