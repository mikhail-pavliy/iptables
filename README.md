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
