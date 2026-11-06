# Домашнє завдання до модуля 2: Транспортний рівень та служби мережі

<sup><sub>Нотатка: це робилося через віртуальну машину від VMWare.</sub></sup>

## Завдання 1. Аналіз відкритих портів

```bash
ddminkova@ddminkova-virtual-machine:~$ ss -tuln
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port Process 
udp    UNCONN  0       0           127.0.0.54:53           0.0.0.0:*            
udp    UNCONN  0       0        127.0.0.53%lo:53           0.0.0.0:*            
udp    UNCONN  0       0              0.0.0.0:5353         0.0.0.0:*            
udp    UNCONN  0       0              0.0.0.0:38412        0.0.0.0:*            
udp    UNCONN  0       0                 [::]:49676           [::]:*            
udp    UNCONN  0       0                 [::]:5353            [::]:*            
tcp    LISTEN  0       4096         127.0.0.1:39075        0.0.0.0:*            
tcp    LISTEN  0       4096        127.0.0.54:53           0.0.0.0:*            
tcp    LISTEN  0       5              0.0.0.0:8080         0.0.0.0:*            
tcp    LISTEN  0       4096     127.0.0.53%lo:53           0.0.0.0:*            
tcp    LISTEN  0       4096         127.0.0.1:631          0.0.0.0:*            
tcp    LISTEN  0       4096           0.0.0.0:22           0.0.0.0:*            
tcp    LISTEN  0       4096             [::1]:631             [::]:*            
tcp    LISTEN  0       4096              [::]:22              [::]:*            
```

### Аналіз

| Протокол | Порт | Стан | Сервіс |
|---|---|---|---|
| TCP | 22 | LISTEN | SSH, доступний з мережі (`0.0.0.0`) |
| TCP | 631 | LISTEN | CUPS (друк), лише локально (`127.0.0.1`) |
| TCP | 8080 | LISTEN | Тестовий веб-сервер `python3 -m http.server 8080` |
| UDP | 53 | UNCONN | Локальний DNS-резолвер systemd-resolved (`127.0.0.53`) |
| UDP | 5353 | UNCONN | mDNS (Avahi) |

- TCP-сокети мають стан `LISTEN`, а UDP — `UNCONN`, бо UDP не встановлює з'єднань.
- Адреса `127.0.0.x` означає, що сервіс доступний лише з цієї машини; `0.0.0.0` / `[::]` — з будь-якого інтерфейсу.
- Прапорці: `-t` TCP, `-u` UDP, `-l` лише сокети, що слухають, `-n` числові порти без імен.

---

## Завдання 2. Аналіз процесів, що слухають порти

```bash
ddminkova@ddminkova-virtual-machine:~$ sudo ss -tulpn
Netid   State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port  Process                                                                         
udp     UNCONN   0        0             127.0.0.54:53             0.0.0.0:*      users:(("systemd-resolve",pid=638,fd=16))                                      
udp     UNCONN   0        0          127.0.0.53%lo:53             0.0.0.0:*      users:(("systemd-resolve",pid=638,fd=14))                                      
udp     UNCONN   0        0                0.0.0.0:5353           0.0.0.0:*      users:(("avahi-daemon",pid=900,fd=12))                                         
udp     UNCONN   0        0                0.0.0.0:38412          0.0.0.0:*      users:(("avahi-daemon",pid=900,fd=14))                                         
udp     UNCONN   0        0                   [::]:49676             [::]:*      users:(("avahi-daemon",pid=900,fd=15))                                         
udp     UNCONN   0        0                   [::]:5353              [::]:*      users:(("avahi-daemon",pid=900,fd=13))                                         
tcp     LISTEN   0        4096           127.0.0.1:39075          0.0.0.0:*      users:(("containerd",pid=14069,fd=15))                                         
tcp     LISTEN   0        4096          127.0.0.54:53             0.0.0.0:*      users:(("systemd-resolve",pid=638,fd=17))                                      
tcp     LISTEN   0        5                0.0.0.0:8080           0.0.0.0:*      users:(("python3",pid=26327,fd=3))                                             
tcp     LISTEN   0        4096       127.0.0.53%lo:53             0.0.0.0:*      users:(("systemd-resolve",pid=638,fd=15))                                      
tcp     LISTEN   0        4096           127.0.0.1:631            0.0.0.0:*      users:(("cupsd",pid=25723,fd=7))                                               
tcp     LISTEN   0        4096             0.0.0.0:22             0.0.0.0:*      users:(("sshd",pid=1187,fd=3),("systemd",pid=1,fd=295))                        
tcp     LISTEN   0        4096               [::1]:631               [::]:*      users:(("cupsd",pid=25723,fd=6))                                               
tcp     LISTEN   0        4096                [::]:22                [::]:*      users:(("sshd",pid=1187,fd=4),("systemd",pid=1,fd=297))                        
ddminkova@ddminkova-virtual-machine:~$ ps -p 26327 -o pid,cmd
    PID CMD
  26327 python3 -m http.server 8080
```

### Аналіз

| Процес | PID | Порт |
|---|---|---|
| `python3` | 26327 | 8080/TCP |
| `systemd-resolve` | 638 | 53/TCP, 53/UDP |
| `sshd` | 1187 | 22/TCP |

- Команду запущено через `sudo`, бо без нього стовпчик з процесами для системних сервісів порожній.
- `ps -p 26327` підтверджує, що PID належить веб-серверу з Завдання 1.
- На порту 22 також вказано `systemd` (PID 1): systemd тримає сокет і передає з'єднання `sshd` (socket activation).

---

## Завдання 3. Перевірка TCP-з'єднання

```bash
ddminkova@ddminkova-virtual-machine:~$ nc -vz google.com 80
Connection to google.com (142.251.98.138) 80 port [tcp/http] succeeded!
```

### Аналіз

- Тип з'єднання: TCP
- Порт: 80 (HTTP)
- Результат: з'єднання встановлено. `succeeded` означає, що пройшло TCP-рукостискання (SYN → SYN-ACK → ACK).
- `-z` — лише перевірити порт без передачі даних, `-v` — детальний вивід.

---

## Завдання 4. Порівняння TCP vs UDP

```bash
ddminkova@ddminkova-virtual-machine:~$ nc -vz 8.8.8.8 53
Connection to 8.8.8.8 53 port [tcp/domain] succeeded!
ddminkova@ddminkova-virtual-machine:~$ nc -u -vz 8.8.8.8 53
Connection to 8.8.8.8 53 port [udp/domain] succeeded!
```

### Аналіз

| | TCP | UDP |
|---|---|---|
| Встановлення з'єднання | Так, рукостискання | Ні (connection-less) |
| Що означає `succeeded` | Сервер підтвердив з'єднання | Лише те, що датаграму відправлено і не прийшла ICMP-помилка «port unreachable» |
| Гарантія доставки | Так | Ні |

- Результат однаковий, але значення різне: для TCP це реальне підтвердження, для UDP — ні. Відкритий, відфільтрований порт або втрачений пакет для nc виглядають однаково.
- DNS працює на обох протоколах: зазвичай UDP, а TCP — для великих відповідей і передачі зон.

---

## Завдання 5. DNS резолвінг

```bash
ddminkova@ddminkova-virtual-machine:~$ nslookup itch.io.
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	itch.io
Address: 104.26.8.198
Name:	itch.io
Address: 172.67.69.99
Name:	itch.io
Address: 104.26.9.198
Name:	itch.io
Address: 2606:4700:20::ac43:4563
Name:	itch.io
Address: 2606:4700:20::681a:9c6
Name:	itch.io
Address: 2606:4700:20::681a:8c6

ddminkova@ddminkova-virtual-machine:~$ dig itch.io

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> itch.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28985
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 4, ADDITIONAL: 8

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;itch.io.			IN	A

;; ANSWER SECTION:
itch.io.		5	IN	A	104.26.8.198
itch.io.		5	IN	A	172.67.69.99
itch.io.		5	IN	A	104.26.9.198

;; AUTHORITY SECTION:
io.			5	IN	NS	a2.nic.io.
io.			5	IN	NS	a0.nic.io.
io.			5	IN	NS	c0.nic.io.
io.			5	IN	NS	b0.nic.io.

;; ADDITIONAL SECTION:
a0.nic.io.		5	IN	A	65.22.160.17
a2.nic.io.		5	IN	A	65.22.163.17
a2.nic.io.		5	IN	AAAA	2a01:8840:a1::17
b0.nic.io.		5	IN	A	65.22.161.17
b0.nic.io.		5	IN	AAAA	2a01:8840:9f::17
c0.nic.io.		5	IN	A	65.22.162.17
c0.nic.io.		5	IN	AAAA	2a01:8840:a0::17

;; Query time: 21 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Fri Nov 06 18:33:34 EET 2026
;; MSG SIZE  rcvd: 304

ddminkova@ddminkova-virtual-machine:~$ resolvectl status
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (ens33)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.216.2
       DNS Servers: 192.168.216.2
        DNS Domain: localdomain

Link 3 (docker0)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

ddminkova@ddminkova-virtual-machine:~$ dig @8.8.8.8 itch.io

; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> @8.8.8.8 itch.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43569
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;itch.io.			IN	A

;; ANSWER SECTION:
itch.io.		300	IN	A	104.26.9.198
itch.io.		300	IN	A	104.26.8.198
itch.io.		300	IN	A	172.67.69.99

;; Query time: 54 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
;; WHEN: Fri Nov 06 18:33:58 EET 2026
;; MSG SIZE  rcvd: 84
```

### Аналіз

| Параметр | Значення |
|---|---|
| Домен | `itch.io` |
| IPv4 | `104.26.8.198`, `104.26.9.198`, `172.67.69.99` |
| IPv6 | `2606:4700:20::ac43:4563`, `2606:4700:20::681a:9c6`, `2606:4700:20::681a:8c6` |
| DNS-сервер | `127.0.0.53` (локальний systemd-resolved) → `192.168.216.2` (upstream) |

- Відповів локальний резолвер `127.0.0.53` (рядок `Server` у nslookup, `SERVER` у dig). Він лише проміжний: `resolvectl status` показує, що запити йдуть далі на `192.168.216.2` — той самий шлюз VMware NAT, що й у ДЗ 1.
- `Non-authoritative answer` — відповідь з кешу резолвера, а не від авторитетного сервера домену.
- Домен має кілька IP-адрес — розподіл навантаження.
- Запит напряму до `8.8.8.8` повертає ті самі адреси. Різниця в TTL (5 проти 300 секунд) — VMware NAT DNS скорочує час кешування.