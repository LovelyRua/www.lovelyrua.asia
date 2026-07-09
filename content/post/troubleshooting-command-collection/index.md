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
-------

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

## 代理设置

临时设置当前 Shell 的代理：

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890
```

```bash
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
export ALL_PROXY=socks5://127.0.0.1:7890
```

取消当前 Shell 的代理：

```bash
unset http_proxy
unset https_proxy
unset all_proxy
unset HTTP_PROXY
unset HTTPS_PROXY
unset ALL_PROXY
```

测试代理是否生效：

```bash
curl -I https://example.com
curl -x http://127.0.0.1:7890 -I https://example.com
```

Git 设置代理：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

Git 取消代理：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

Docker 服务设置代理，适合拉镜像慢或拉不下来时排查：

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/proxy.conf
```

```ini
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1,::1"
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl show --property=Environment docker
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
curl -iv https://example.com --connect-to example.com:443:12.34.56.78:443
```

```text
--connect-to 会保持 URL 和 SNI 还是 example.com,
但实际 TCP 连接会打到 12.34.56.78:443。
适合排查 DNS、CDN、源站差异。
```

```bash
curl -X POST https://example.com/api
curl -H 'Content-Type: application/json' -d '{"hello":"world"}' https://example.com/api
```

```bash
openssl s_client -connect example.com:443 -servername example.com
openssl s_client -connect 1.2.3.4:443 -servername example.com -brief
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -issuer -subject
```

```text
-connect 1.2.3.4:443       实际连接的 IP 和端口
-servername example.com    TLS SNI 使用的域名
-brief                     输出简洁结果
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

```bash
nginx -T | grep -A 5 -i "proxy_pass"
```

```text
-A 5 表示匹配到 proxy_pass 后，再额外显示后面 5 行。
适合快速确认 proxy_set_header、proxy_http_version 等反代配置。
```

```bash
nginx -T > /tmp/full_nginx.conf && less /tmp/full_nginx.conf
```

```text
nginx -T 会输出主配置和所有 include 进来的配置。
导出到 /tmp/full_nginx.conf 后更方便搜索和分页查看。
```

```bash
awk '$9 ~ /^5[0-9][0-9]$/' /var/log/nginx/access.log | tail -n 50
grep -E '(connect\(\) failed|no live upstreams)' /var/log/nginx/error.log | tail -n 20
```

```text
access.log 默认第 9 列通常是 HTTP 状态码。
/^5[0-9][0-9]$/ 匹配 500 到 599。
connect() failed 表示 Nginx 连不上后端 upstream。
no live upstreams 表示 upstream 里没有可用后端。
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
find / -xdev -type f | cut -d "/" -f 2 | sort | uniq -c | sort -n
```

```text
/                         从根目录开始
-xdev                     不跨文件系统, 避免扫到挂载盘、proc、sys 等
-type f                   只统计普通文件
cut -d "/" -f 2           取一级目录名
sort | uniq -c | sort -n  统计数量并排序
```

```bash
du -sh *
du -h --max-depth=1
find /var -type f -size +500M -print
```

```bash
sudo lsof | grep deleted
sudo lsof +L1
sudo truncate -s 0 /var/log/big.log
```

```text
lsof +L1 用来查看已经删除但仍被进程占用的文件。
常见场景是日志文件被删了, 但进程还开着文件句柄, 磁盘空间不会释放。
```

查看大于 10 MB 的已删除占用文件：

```bash
sudo lsof | grep -E '(deleted|COMMAND)' | awk '{if($7 > 10485760) print $0}'
```

```text
$7 通常是文件大小字段, 单位是字节。
10485760 = 10 MB。
deleted 表示文件已经从目录中删除, 但仍被进程持有。
```

更清晰地列出大于 10 MB 的 deleted 文件：

```bash
sudo lsof +L1 | awk 'NR==1 || $7 > 10485760'
```

如果只是想分页查看：

```bash
sudo lsof +L1 | less
```

```text
sudo lsof +L1 | lmd 里的 lmd 不是常见系统命令,
可能是本机 alias 或笔误。
```

清空进程仍持有的文件描述符：

```bash
true > /proc/<PID>/fd/<FD>
```

```text
<PID> 是占用文件的进程 ID。
<FD> 是 lsof 输出里的文件描述符编号。
这个操作会把进程打开的那个文件句柄清空。
```

让进程关闭指定 FD：

```bash
sudo gdb -p <PID> -ex 'p close(4)' -ex 'quit'
```

```text
close(4) 表示关闭 FD 4, 需要换成实际 FD 编号。
这个方法会 attach 到进程, 可能导致进程短暂停顿。
生产环境谨慎使用。
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

查看当前最容易被 OOM Killer 杀掉的进程：

```bash
printf "PID\tOOM Score\tCommand\n" && printf "%s\n" /proc/[0-9]* | awk -F/ '{print $3}' | while read pid; do if [ -f /proc/$pid/oom_score ]; then echo -e "$pid\t$(cat /proc/$pid/oom_score)\t$(cat /proc/$pid/cmdline | tr '\0' ' ')"; fi; done | sort -k2 -nr | head -n 10
```

```text
/proc/<PID>/oom_score      当前进程被 OOM Killer 选中的倾向
分数越高, 越容易在内存不足时被杀
cmdline                    进程启动命令
sort -k2 -nr               按 OOM Score 从高到低排序
head -n 10                 只看前 10 个
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

查看所有正在运行容器的容器名、容器 IP 和网络 ID：

```bash
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}} ({{.NetworkID}}){{end}}' $(docker ps -q)
```

```text
.Name                         容器名
.NetworkSettings.Networks     容器加入的 Docker 网络
.IPAddress                    容器在该网络内的 IP
.NetworkID                    Docker 网络 ID
```

进入容器网络命名空间抓包：

```bash
PID=$(docker inspect -f '{{.State.Pid}}' <容器名>)
sudo nsenter -t $PID -n tcpdump -ni any port 80
```

```text
docker inspect 取出容器对应的宿主机进程 PID。
nsenter -t $PID -n 进入该进程的 network namespace。
tcpdump -ni any port 80 在容器网络视角下抓 80 端口流量。
```

如果 tcpdump 版本支持按容器 ID 抓包：

```bash
sudo tcpdump --container-id <容器名或ID> -ni any
```

```text
--container-id 不是所有系统的 tcpdump 都支持。
如果报 unknown option, 改用 nsenter 方法。
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

```text
docker compose config 会合并 compose.yaml、override、
环境变量替换后的最终配置。
适合排查 ports、volumes、environment、networks 是否写错。
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
docker compose config
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

磁盘满但看不出哪里占用:

```bash
df -h
du -h --max-depth=1 /
sudo lsof +L1
sudo lsof +L1 | awk 'NR==1 || $7 > 10485760'
```

Nginx 反代异常:

```bash
sudo nginx -t
nginx -T | grep -A 5 -i "proxy_pass"
grep -E '(connect\(\) failed|no live upstreams)' /var/log/nginx/error.log | tail -n 20
awk '$9 ~ /^5[0-9][0-9]$/' /var/log/nginx/access.log | tail -n 50
```

容器网络异常:

```bash
docker ps
docker port 容器名
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}} ({{.NetworkID}}){{end}}' $(docker ps -q)
PID=$(docker inspect -f '{{.State.Pid}}' <容器名>)
sudo nsenter -t $PID -n tcpdump -ni any port 80
```

疑似 OOM:

```bash
free -h
ps aux --sort=-%mem | head
dmesg | grep -i oom
journalctl -k | grep -i oom
printf "PID\tOOM Score\tCommand\n" && printf "%s\n" /proc/[0-9]* | awk -F/ '{print $3}' | while read pid; do if [ -f /proc/$pid/oom_score ]; then echo -e "$pid\t$(cat /proc/$pid/oom_score)\t$(cat /proc/$pid/cmdline | tr '\0' ' ')"; fi; done | sort -k2 -nr | head -n 10
```
