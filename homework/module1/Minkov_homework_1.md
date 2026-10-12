# Домашнє завдання до модуля 1: Основи комп'ютерних мереж

<sup><sub>Нотатка: це робилося через віртуальну машину від VMWare.</sub></sup>

## Завдання 1. Дослідження мережі

```bash
ddminkov@ddminkov-virtual-machine:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:79:c6:79 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.216.128/24 brd 192.168.216.255 scope global dynamic noprefixroute ens33
       valid_lft 1691sec preferred_lft 1691sec
    inet6 fe80::2038:2a14:9bc:b96d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 56:7d:33:c0:de:b1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

### Аналіз

| Параметр | Значення | Пояснення |
|---|---|---|
| Інтерфейс | `ens33` | Основний робочий інтерфейс (стан `UP`). `lo` — службовий loopback, `docker0` — міст Docker, зараз неактивний (`NO-CARRIER`, `DOWN`), бо немає запущених контейнерів. |
| Тип підключення | Ethernet (віртуальний адаптер) | Віртуальна машина VMware, мережевий адаптер емульований; префікс MAC `00:0c:29` належить VMware. |
| MAC-адреса | `00:0c:29:79:c6:79` | Фізична адреса інтерфейсу. Канальний рівень (L2) моделі OSI. |
| IP-адреса | `192.168.216.128` | Приватна адреса IPv4, отримана динамічно (`dynamic`). Мережевий рівень (L3) моделі OSI. |
| Маска | `/24` = `255.255.255.0` | Визначає, яка частина адреси належить мережі, а яка — хосту. Мережевий рівень (L3) моделі OSI. |
| Gateway | `192.168.216.2` | Шлюз за замовчуванням (видно у виводі `ip r`, рядок `default via`). |
| Broadcast | `192.168.216.255` | Широкомовна адреса мережі (`brd` у виводі). |
| Hosts | 254 | Адреса мережі `192.168.216.0`, діапазон хостів `192.168.216.1`–`192.168.216.254`, broadcast `192.168.216.255`. Кількість хостів: 2⁸ − 2 = 254. |

---

## Завдання 2. Дослідження маршрутизації

```bash
ddminkov@ddminkov-virtual-machine:~$ ip r
default via 192.168.216.2 dev ens33 proto dhcp src 192.168.216.128 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
192.168.216.0/24 dev ens33 proto kernel scope link src 192.168.216.128 metric 100 
```

### Аналіз

- Протокол, через який отримано IP: DHCP. Маршрут за замовчуванням має `proto dhcp`, тобто IP-адресу та шлюз `192.168.216.2` отримано від DHCP-сервера (у VMware його надає віртуальний NAT-сервіс). Це узгоджується з `dynamic` та обмеженим часом оренди `valid_lft 1691sec` у виводі `ip a`.
- Два інших маршрути мають `proto kernel`: їх автоматично створило ядро Linux, коли інтерфейси отримали адреси. Маршрут `172.17.0.0/16` позначений `linkdown`, бо `docker0` неактивний.
- Правило `default via 192.168.216.2` означає: увесь трафік, для якого немає точнішого маршруту, відправляється на шлюз.

---

## Завдання 3. Публічна IP-адреса

```bash
ddminkov@ddminkov-virtual-machine:~$ curl ifconfig.io
46.133.xx.xx
```

(останні октети приховано)

### Порівняння

| | Адреса |
|---|---|
| Локальна | `192.168.216.128` |
| Публічна | `46.133.xx.xx` |