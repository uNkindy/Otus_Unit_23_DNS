### Домашнее задание №23 (DNS)
#### 1. Создан [Vagrantfile](https://github.com/uNkindy/Otus_Unit_23_DNS/blob/main/Vagrantfile) c 4 ВМ: ns01, ns02, client. Дополнительно в стенд введена ВМ client2;
#### 2. Добавил 2 имени в зону dns.lab:
```console
web1 IN A 192.168.50.15
web2 IN A 192.168.50.16
```
Проверим:
```console
[root@client vagrant]# dig @192.168.50.10 web1.dns.lab

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.13 <<>> @192.168.50.10 web1.dns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2418
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;web1.dns.lab.                  IN      A

;; ANSWER SECTION:
web1.dns.lab.           3600    IN      A       192.168.50.15

;; AUTHORITY SECTION:
dns.lab.                3600    IN      NS      ns01.dns.lab.
dns.lab.                3600    IN      NS      ns02.dns.lab.

;; ADDITIONAL SECTION:
ns01.dns.lab.           3600    IN      A       192.168.50.10
ns02.dns.lab.           3600    IN      A       192.168.50.11

;; Query time: 1 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Tue Jan 31 07:49:18 UTC 2023
;; MSG SIZE  rcvd: 127
```
#### 3. Создание новой зоны и добавление в нее записей:
```console
www IN A 192.168.50.15
www IN A 192.168.50.16
```
#### 4. Настройка Split-DNS:
Проверим работу стенда, будем пингать разные адреса:
```console
## ВМ client:

[root@client vagrant]# ping www.newdns.lab
PING www.newdns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from client (192.168.50.15): icmp_seq=1 ttl=64 time=0.006 ms
64 bytes from client (192.168.50.15): icmp_seq=2 ttl=64 time=0.060 ms
64 bytes from client (192.168.50.15): icmp_seq=3 ttl=64 time=0.060 ms
^C
--- www.newdns.lab ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2006ms
rtt min/avg/max/mdev = 0.006/0.042/0.060/0.025 ms

__

[root@client vagrant]# ping web1.dns.lab
PING web1.dns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from client (192.168.50.15): icmp_seq=1 ttl=64 time=0.007 ms
64 bytes from client (192.168.50.15): icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from client (192.168.50.15): icmp_seq=3 ttl=64 time=0.061 ms
^C
--- web1.dns.lab ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.007/0.046/0.070/0.027 ms

__

[root@client vagrant]# ping web2.dns.lab
ping: web2.dns.lab: Name or service not known
```
В результате видим, что client видит обе зоны (dns.lab и newdns.lab), однако информацию о хосте web2.dns.lab он получить не может.

```console
## ВМ client2:

[root@client2 vagrant]# ping www.newdns.lab
ping: www.newdns.lab: Name or service not known

__

[root@client2 vagrant]# ping web1.dns.lab
PING web1.dns.lab (192.168.50.15) 56(84) bytes of data.
64 bytes from 192.168.50.15 (192.168.50.15): icmp_seq=1 ttl=64 time=0.334 ms
64 bytes from 192.168.50.15 (192.168.50.15): icmp_seq=2 ttl=64 time=0.349 ms
64 bytes from 192.168.50.15 (192.168.50.15): icmp_seq=3 ttl=64 time=0.933 ms
^C
--- web1.dns.lab ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.334/0.538/0.933/0.280 ms

__

[root@client2 vagrant]# ping web2.dns.lab
PING web2.dns.lab (192.168.50.16) 56(84) bytes of data.
64 bytes from client2 (192.168.50.16): icmp_seq=1 ttl=64 time=0.006 ms
64 bytes from client2 (192.168.50.16): icmp_seq=2 ttl=64 time=0.075 ms
64 bytes from client2 (192.168.50.16): icmp_seq=3 ttl=64 time=0.061 ms
^C
--- web2.dns.lab ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.006/0.047/0.075/0.030 ms
```
Тут видим, что client2 видит всю зону dns.lab и не видит зону
newdns.lab
