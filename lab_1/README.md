<div align="center">

# Лабораторная работы №1
</div>

<div align="justify">

Открыть терминал и перейти в папку с **Virtual Box** (например, **C:\Program Files\Oracle\VirtualBox**)

```bash
vboxmanage dhcpserver add --netname intnet --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.2 --upperip 10.10.10.212 --enable
```

Данная команда создает DHCP сервер для локальной сети **intnet**

---

Скачать **Ubuntu Server** с [официального сайта](https://ubuntu.com/download/server "скачивание Ubuntu Server")

Скачать **Virtual Box** с [официального сайта](https://www.virtualbox.org/wiki/Downloads "скачивание Virtual Box")

---

Создать 2 виртуальные машины (все вроде тривиально, единственное я увеличил размер выделяемой памяти для каждой виртуальной машины)

У одной виртуальной машины (я назвал её **Router**) во вкладке **Настроить &rarr; Сеть** настроить 2 сетевых адаптера (один - сетевой мост, второй - внутренняя сеть)

У второй (я назвал её **Server**) аналогичным образом настроить 1 сетевой адаптер (внутренняя сеть)

Запустить обе виртуальные машины и подключить ранее скачанный образ **Ubuntu Server**

---

В терминалах **Router** и **Server** прописать команду, чтобы работать от лица пользователя с правами **root** (скорее всего потребуется ранее установленный, при установке образов, пароль)

```bash
sudo su
```
</div>

---

<div align="center">

# Router
</div>

<div align="justify">

**enp0s8** - интерфейс внутренней сети

**enp0s3** - интерфейс сетевого моста

```bash
apt-get update && apt-get upgrade -y

apt-get install iptables-persistent

echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.d/gateway.conf

sysctl -p /etc/sysctl.d/gateway.conf

iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT

iptables -A FORWARD -i enp0s8 -o enp0s3 -m state --state RELATED,ESTABLISHED -j ACCEPT

iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

netfilter-persistent save
```

Установка **nginx**

```bash
apt update
apt install nginx
```

Переписать с помощью команды конфигурацию **nginx**

```bash
vim /etc/nginx/nginx.conf 
# нажать i, чтобы редактировать файл
```

Файл **/etc/nginx/nginx.conf**

```
events {
}

http {
    server {
        listen 80;
        server_name 10.10.10.6; # ip этой виртуальной машины (Router)

        location / {
            proxy_pass http://10.10.10.5:80; # ip виртуальной машины (Server)
            proxy_set_header Host $host;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

Сохранить изменения

```bash
<ESC>
:wq
```

Перезапустить **nginx**

```bash
nginx -s reload
systemctl start nginx
```

</div>

---

<div align="center">

# Server
</div>

<div align="justify">

```bash
vim /etc/netplan/00-installer-config.yaml 
# нажать i, чтобы редактировать файл
```

Файл **/etc/netplan/00-installer-config.yaml**

```
network:
    ethernets:
        enp0s3:
            dhcp4: true
            routes:
                - to: default
                  via: 10.10.10.6
            nameservers:
                addresses: [8.8.8.8, 8.8.4.4]
    version: 2
```

Сохранить изменения

```bash
<ESC>
:wq
```

```bash
netplan try

netplan apply

systemctl restart systemd-networkd

ping gooogle.com -c 5 # проверили подключение к Интернет
```

Установка **apache2**

```bash
apt update
apt install apache2
systemctl start apache2
```

</div>

---

<div align="justify">

Вдохновение брал с [Гришиного github](https://github.com/migregal/bmstu-iu7-devops/blob/master/lab_01/int_net_gateway.md#vms-creation "Лабораторная работа №1").
</div>