# iptables
- реализовать ```knocking port```, 
```centralRouter``` может попасть на ssh ```inetrRouter``` через knock скрипт пример в материалах
- добавить ```inetRouter2```, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост
- запустить nginx на ```centralServer```
- пробросить 80й порт на ```inetRouter2``` 8080
- дефолт в инет оставить через ```inetRouter```
- реализовать проход на 80й порт без маскарадинга

Выполним настройку iptables на inetRouter предварительно установим пакет ```iptables-services```, чтобы наши правила работали после перезагрузки
```ruby
[root@inetRouter vagrant]# yum install -y iptables-services
[root@inetRouter vagrant]# systemctl enable --now iptables
[root@inetRouter vagrant]# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
[root@inetRouter vagrant]# iptables -A INPUT -p tcp --dport 22 -j ACCEPT
[root@inetRouter vagrant]# iptables -I FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
[root@inetRouter vagrant]# iptables -P INPUT DROP
[root@inetRouter vagrant]# iptables -P FORWARD DROP
```
Для того, чтобы ```iptables-services``` при загрузке сервер считывал настройки с файла ```/etc/sysconfig/iptables``` используем утилиту ```iptables-save```
```ruby
[root@inetRouter vagrant]# iptables-save > /etc/sysconfig/iptables
```
# 1. реализовать ```knocking port```, ```centralRouter``` может попасть на ssh ```inetrRouter``` через knock скрипт
Добавим на ```inetrRouter``` пользователя testuser
```ruby
[root@inetRouter vagrant]# useradd testuser
[root@inetRouter vagrant]# echo "test" | passwd --stdin testuser
[root@inetRouter vagrant]# sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config && systemctl restart sshd.service
```
Для реализации ```knocking port``` нам необходимо  создать три цепочки в таблице filter: TRAFFIC, SSH-INPUT, SSH-INPUTTWO
```ruby
[root@inetRouter vagrant]# iptables -N TRAFFIC
[root@inetRouter vagrant]# iptables -N SSH-INPUT
[root@inetRouter vagrant]# iptables -N SSH-INPUTTWO
```
так же необходимо добавить следующие правила
```ruby
[root@inetRouter vagrant]# iptables -A INPUT -j TRAFFIC
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp --dport 22 -m recent --rcheck --seconds 30 --name SSH2 -j ACCEPT
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp -m recent --name SSH2 --remove -j DROP
```
Добавим правила проверки последовательности портов. 
```ruby
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp --dport 9992 -m recent --rcheck --name SSH1 -j SSH-INPUTTWO
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp -m recent --name SSH1 --remove -j DROP
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp --dport 7772 -m recent --rcheck --name SSH0 -j SSH-INPUT
[root@inetRouter vagrant]# iptables -A TRAFFIC -m conntrack --ctstate NEW -m tcp -p tcp -m recent --name SSH0 --remove -j DROP
```











