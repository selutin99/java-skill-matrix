# Как работать с systemd, iptables, tcpdump

## 1. systemd

### 1.1. Что это

* **systemd** — init-система в Linux (замена SysVinit, Upstart).
* Управляет: сервисами (демонами), юнитами, процессами, логами (journal).

### 1.2. Основные команды

```bash
systemctl status nginx            # статус сервиса
systemctl start nginx             # запустить
systemctl stop nginx              # остановить
systemctl restart nginx           # перезапустить
systemctl reload nginx            # перечитать конфиг (SIGHUP)
systemctl enable nginx            # включить автозапуск при старте
systemctl disable nginx           # выключить автозапуск
systemctl is-enabled nginx        # проверить автозапуск
systemctl list-units --type=service --state=running
```

### 1.3. Unit-файл (пример)

`/etc/systemd/system/myapp.service`

```ini
[Unit]
Description=My Spring Boot App
After=network.target

[Service]
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/java -jar app.jar
Restart=always
Environment=SPRING_PROFILES_ACTIVE=prod
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload   # перечитать unit-файлы
systemctl enable myapp    # включить автозапуск
systemctl start myapp     # запустить
```

### 1.4. Логи

```bash
journalctl -u myapp -f       # последние логи сервиса
journalctl -xe               # ошибки
journalctl --since "1 hour ago"
```

---

## 2. iptables

### 2.1. Что это

* **iptables** — утилита для настройки правил фильтрации пакетов в Linux (iptables = интерфейс к netfilter).
* Используется для: firewall, NAT, маршрутизации.

### 2.2. Основные таблицы

* **filter** — фильтрация пакетов (по умолчанию).
* **nat** — трансляция адресов (SNAT, DNAT, MASQUERADE).
* **mangle** — изменение заголовков пакетов.

### 2.3. Основные цепочки (chains)

* **INPUT** — входящие пакеты в хост.
* **OUTPUT** — исходящие пакеты с хоста.
* **FORWARD** — проходящие транзитом.
* **PREROUTING/POSTROUTING** — для NAT.

### 2.4. Базовые примеры

```bash
# Посмотреть правила
iptables -L -n -v

# Разрешить входящий SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Разрешить HTTP/HTTPS
iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# Запретить всё остальное (правило по умолчанию DROP)
iptables -P INPUT DROP

# NAT: проброс порта 8080 -> 80
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
```

### 2.5. Сохранение правил

* На Ubuntu/Debian: `iptables-save > /etc/iptables/rules.v4`
* На CentOS/RHEL: `service iptables save`
* Современный аналог → **nftables**.

---

## 3. tcpdump

### 3.1. Что это

* **tcpdump** — инструмент для перехвата и анализа сетевого трафика.
* Работает с интерфейсами (eth0, lo).
* Часто используют вместе с Wireshark (pcap-файлы).

### 3.2. Основные команды

```bash
tcpdump -i eth0                  # слушать интерфейс eth0
tcpdump -i any                   # слушать все интерфейсы
tcpdump -c 100                   # только 100 пакетов
tcpdump -n                       # не резолвить DNS
tcpdump -nn                      # не резолвить DNS + порты
tcpdump -v                       # подробный вывод
tcpdump -w dump.pcap             # сохранить в файл для Wireshark
```

### 3.3. Фильтры

* По протоколу:

  ```bash
  tcpdump tcp
  tcpdump udp
  tcpdump icmp
  ```
* По IP:

  ```bash
  tcpdump host 192.168.1.10
  tcpdump src 10.0.0.1
  tcpdump dst 8.8.8.8
  ```
* По порту:

  ```bash
  tcpdump port 80
  tcpdump src port 443
  tcpdump dst port 22
  ```
* Комбинации:

  ```bash
  tcpdump tcp port 80 and host 192.168.1.5
  ```

### 3.4. Примеры диагностики

```bash
tcpdump -i eth0 port 5432          # смотреть PostgreSQL-трафик
tcpdump -i any tcp port 8080 -A    # HTTP-трафик (печать ASCII)
tcpdump -i eth0 -nn -s0 -w trace.pcap   # полный дамп в файл
```

---

## 4. Выжимка для собеседования

* **systemd**: init-система Linux, управляет сервисами и юнитами.

    * `systemctl start/stop/status/enable`.
    * Unit-файлы (`[Unit]`, `[Service]`, `[Install]`).
    * Логи: `journalctl -u <service>`.

* **iptables**: настройка правил сетевого фильтра и NAT.

    * Таблицы: filter, nat, mangle.
    * Цепочки: INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING.
    * Пример: разрешить 22,80,443 → остальное DROP.

* **tcpdump**: диагностика сетевого трафика.

    * Слушает пакеты на интерфейсе, фильтрация по IP/порту/протоколу.
    * `-w` для записи в pcap (Wireshark).
    * Применяется для отладки сетевых проблем, перехвата HTTP/DB-запросов.
