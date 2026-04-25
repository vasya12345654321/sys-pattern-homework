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

---

## Проверка работы

### 1. Нормальное состояние (VIP на Ubuntu1)

<img width="1782" height="192" alt="заданние2" src="https://github.com/user-attachments/assets/2155bd60-d149-4550-9eeb-5f96abdd23fc" />

---

### 2. Отказ сервиса (nginx остановлен)

```bash
sudo systemctl stop nginx
```

VIP переходит на Ubuntu2:

<img width="1862" height="255" alt="заданние22 уходит" src="https://github.com/user-attachments/assets/707ed3bd-c5a4-4f68-b839-e61757d0e149" />


---

### 3. Восстановление

```bash
sudo systemctl start nginx
```

VIP возвращается на Ubuntu1:
<img width="1782" height="192" alt="заданние2" src="https://github.com/user-attachments/assets/5709b212-f3da-460a-bccf-6eb3b37acdbf" />


---

## Вывод

Настроена отказоустойчивая схема с использованием keepalived.
При остановке nginx виртуальный IP автоматически переключается на резервный сервер.
После восстановления сервиса IP возвращается обратно.


