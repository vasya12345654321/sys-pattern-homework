# Домашнее задание к занятию "`Disaster recovery и Keepalived`" - `Лукьянов Михаил

# Домашнее задание 1 — HSRP

## Цель
Настроить отказоустойчивость сети с использованием HSRP.

---
## Настройка

<img width="1850" height="822" alt="Снимок экрана 2026-04-25 135104" src="https://github.com/user-attachments/assets/1b566d96-b155-49df-b855-4593dcd79381" />



---

## Проверка состояния HSRP

Router0:
<img width="1770" height="1805" alt="заданние1111" src="https://github.com/user-attachments/assets/49365457-c8d7-4a2f-9a7c-d55abab3a104" />


Router1:
<img width="1762" height="1775" alt="заданние11" src="https://github.com/user-attachments/assets/39f4197c-c939-43a3-ace2-c7bd4a5c3028" />

---

## Проверка отказоустойчивости

<img width="1775" height="1787" alt="заданние1" src="https://github.com/user-attachments/assets/885b8b1d-6024-4235-b30e-d35743079825" />

<img width="1950" height="1865" alt="заданние111" src="https://github.com/user-attachments/assets/219c0eb7-a191-4faa-a113-f290d65e5b29" />

---

## Вывод

HSRP настроен успешно.  
При отказе активного маршрутизатора происходит переключение на резервный с минимальной потерей пакетов.



# Домашнее задание 2 — Keepalived

## Цель

Настроить отказоустойчивость сервиса с использованием keepalived и виртуального IP.

---

## Описание стенда

* Ubuntu1 — 192.168.56.101 (MASTER)
* Ubuntu2 — 192.168.56.104 (BACKUP)
* Виртуальный IP — 192.168.56.200

---

## Установка

```bash
sudo apt update
sudo apt install nginx -y
sudo apt install keepalived -y
```

---

## Конфигурация keepalived

### Ubuntu1 (MASTER)

```bash
vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 3
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface enp0s9
    virtual_router_id 51
    priority 100
    advert_int 1
    preempt

    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
        192.168.56.200
    }

    track_script {
        chk_nginx
    }
}
```
<img width="1747" height="1215" alt="заданние222 конфиг" src="https://github.com/user-attachments/assets/6c064b66-af26-4e02-a848-aaa989f9e32d" />

---

### Ubuntu2 (BACKUP)

```bash
vrrp_script chk_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 3
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s9
    virtual_router_id 51
    priority 90
    advert_int 1
    preempt

    authentication {
        auth_type PASS
        auth_pass 1111
    }

    virtual_ipaddress {
        192.168.56.200
    }

    track_script {
        chk_nginx
    }
}
```
<img width="1902" height="1367" alt="Снимок экрана 2026-04-25 152803" src="https://github.com/user-attachments/assets/6202691c-2ef2-4ecb-a389-5883caf638ae" />

---

## Скрипт проверки nginx

```bash
#!/bin/bash

nc -z localhost 80
if [ $? -ne 0 ]; then
    exit 1
fi

if [ ! -f /var/www/html/index.html ]; then
    exit 1
fi

exit 0
```
<img width="2115" height="1290" alt="заданние222 конфиг233" src="https://github.com/user-attachments/assets/13c7d1a3-c33c-4ca8-bd2e-927ac9d84267" />


---

## Проверка работы

### 1. Нормальное состояние (VIP на Ubuntu1)

<img width="2735" height="660" alt="Снимок экрана 2026-04-25 152007" src="https://github.com/user-attachments/assets/01304d06-db87-4501-a008-6761fad4a58f" />

---

### 2. Отказ сервиса (nginx остановлен)

```bash
sudo systemctl stop nginx

![fail](img/4.png)



VIP переходит на Ubuntu2:


![fail](img/5.png)




---

### 3. Восстановление

```bash
sudo systemctl start nginx
```

VIP возвращается на Ubuntu1:
<img width="2210" height="1657" alt="заданние2" src="https://github.com/user-attachments/assets/522980eb-31bb-49d2-bcba-b33154f34abb" />



---

## Вывод

Настроена отказоустойчивая схема с использованием keepalived.
При остановке nginx виртуальный IP автоматически переключается на резервный сервер.
После восстановления сервиса IP возвращается обратно.


