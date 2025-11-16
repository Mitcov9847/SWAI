# Лабораторная работа №3  
**Мониторинг ИТ-инфраструктуры с использованием Zabbix**  

---

## Цель работы
Целью данной работы является ознакомление студентов с системой Zabbix для мониторинга серверов, сервисов и сетевого оборудования. В ходе работы студенты изучают установку и настройку Zabbix, добавление хостов, настройку сбора метрик, создание триггеров и оповещений, а также создание персональной панели мониторинга (Dashboard).

---

## Задачи
1. Установить и настроить сервер Zabbix.  
2. Установить агент Zabbix на контролируемую машину.  
3. Добавить и настроить хост в системе.  
4. Просмотреть собираемые метрики и создать оповещения.  

---

## Необходимое оборудование и программное обеспечение
- Две виртуальные машины (например, Ubuntu Server 22.04).  
- Доступ в Интернет.  
- Zabbix Server + MySQL + Apache/Nginx.  
- Zabbix Agent.  

---

## Ход работы

### 1. Установка и настройка сервера Zabbix
Для начала работы требуется подготовить виртуальную машину с Ubuntu 22.04. После обновления системы выполняем установку Zabbix:  

```
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

> **Пояснение:**  
> - Zabbix Server — основной компонент для сбора и обработки данных.  
> - Frontend (веб-интерфейс) позволяет визуализировать метрики.  
> - Apache/Nginx используется для отображения веб-интерфейса.  
> - MySQL хранит все метрики и конфигурации.
<img width="812" height="483" alt="{C02113D1-426B-44E4-BB9D-34B857AD93DF}" src="https://github.com/user-attachments/assets/4343347a-c245-47d6-b76c-1c415d882e4f" />

<img width="829" height="102" alt="{D0D4F3BF-8D21-4E68-AADA-1345DE0D3A16}" src="https://github.com/user-attachments/assets/ca75f862-5f39-4bfa-adf0-0af07b7bfa24" />

<img width="708" height="215" alt="{E75051F1-601F-4C13-93B5-1A51CBDF60DD}" src="https://github.com/user-attachments/assets/cfa9e90d-b8f3-4640-9c14-574a336c6fca" />

---

### 2. Настройка базы данных и веб-интерфейса
Создаем базу данных для Zabbix и пользователя:  

```
sudo mysql -uroot -p
CREATE DATABASE zabbix character set utf8mb4 collate utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'zabbix_password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
<img width="831" height="586" alt="{F46168AD-DD4A-4219-87D0-CA52E98DF002}" src="https://github.com/user-attachments/assets/51dcf3db-36be-49ab-88db-3d2052aa25d0" />

Импортируем схему базы данных:

```bash
zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -uzabbix -pzabbix_password zabbix
```
<img width="810" height="37" alt="{0111167C-A1D1-49A5-B7A6-651FBE072CD9}" src="https://github.com/user-attachments/assets/7a1e5dbc-946e-459e-9641-b110c1acb102" />

Настраиваем конфигурацию Zabbix сервера (`/etc/zabbix/zabbix_server.conf`), указывая логин и пароль к базе данных.

---

### 3. Установка Zabbix Agent на другой машине
На контролируемой машине устанавливаем Zabbix Agent:  

```
sudo apt install zabbix-agent -y
sudo systemctl enable zabbix-agent
sudo systemctl start zabbix-agent
sudo systemctl status zabbix-agent
```
<img width="738" height="358" alt="{1CB94D9E-E54E-4FD1-A469-B09E9C1B8524}" src="https://github.com/user-attachments/assets/fa333af9-b964-4160-8a2d-f990e07d461d" />

> **Пояснение:**  
> Zabbix Agent собирает метрики с сервера или рабочего места и отправляет их на Zabbix Server.  
> Связь может быть активной (Agent отправляет данные серверу) или пассивной (сервер опрашивает агент).
> 
<img width="817" height="378" alt="{5D1A10B8-0EAE-4FCA-9021-15EA9179878D}" src="https://github.com/user-attachments/assets/37beba04-a125-45cc-8d8b-685fac847b25" />

---
### Первоначальная настройка Zabbix через веб-интерфейс
В браузере стал доступен установочный мастер по адресу:
http://localhost/zabbix 
![photo_5247232477368094042_y](https://github.com/user-attachments/assets/920a1669-125d-44be-a737-e8821a6d5b58)

![photo_5247232477368094042_y (1)](https://github.com/user-attachments/assets/3553e22c-006a-44a4-9b8f-cbea7381a0a1)

### 4. Добавление хоста в веб-интерфейсе Zabbix
В интерфейсе Zabbix переходим в `Configuration → Hosts → Create host` и добавляем контролируемую машину.  

- Host name: `jenia-desktop`  
- Groups: `Linux servers`  
- Interfaces: IP 127.0.0.1, порт 10050  
- Templates: `Linux by Zabbix agent`  

---
<img width="1033" height="647" alt="{9D63943B-A747-4B6A-8EE0-21B1B87D15B6}" src="https://github.com/user-attachments/assets/f0ebb0ac-8c25-4b73-8093-d8374b0dc084" />

---

### 6. Настройка оповещения (триггер + действие)
Создаем триггер:  
- Name: `High CPU usage on jenia-desktop`  
- Expression: `{jenia-desktop:system.cpu.util.last()}>80`  
- Severity: `High`  

> **Пояснение:**  
> Triggers позволяют Zabbix уведомлять администратора при критических событиях.  
> Action log фиксирует все срабатывания триггеров.  
<img width="863" height="728" alt="{8EB2FD21-9D7E-4C04-90C1-266F2F651017}" src="https://github.com/user-attachments/assets/0867a65b-9f19-44df-9222-032534dc7af9" />

---

### 7. Создание персональной панели мониторинга (Dashboard)
В интерфейсе Zabbix создаем Dashboard:  
- Добавляем виджеты с графиками CPU, RAM, сети  
- Настраиваем обновление каждые 10 секунд  
- Можно визуально отслеживать состояние хоста
- 
<img width="936" height="404" alt="{DE6DF35C-E8F5-4AE4-84F7-501F47DBD5BE}" src="https://github.com/user-attachments/assets/164efdea-086b-4f1f-9a82-f89643fdf253" />

<img width="592" height="331" alt="{B9455510-1D84-4B2E-9AB5-EEBE3888A450}" src="https://github.com/user-attachments/assets/732f873f-30ff-444d-b6a6-38b94210a1c3" />

---

## Контрольные вопросы

1. **Какова роль основных компонентов Zabbix (Server, Agent, Proxy, Database, Frontend)?**  
- **Server:** собирает и обрабатывает данные.  
- **Agent:** собирает метрики на удаленных хостах.  
- **Proxy:** промежуточный сборщик для удаленных сетей.  
- **Database:** хранит все метрики и конфигурации.  
- **Frontend:** веб-интерфейс для визуализации данных.  

2. **Как осуществляется связь между Zabbix Server и Zabbix Agent?**  
Связь может быть:  
- **Активная:** агент отправляет данные серверу.  
- **Пассивная:** сервер опрашивает агент по расписанию.  
В обоих случаях используется порт 10050 для связи.  

3. **Что такое Items и Triggers в Zabbix?**  
- **Items:** метрики, которые Zabbix собирает с хостов (CPU, RAM, Disk, Network).  
- **Triggers:** условия, при которых генерируются оповещения о проблемах (например, CPU > 80%).  

---

## Заключение
В ходе лабораторной работы я установил и настроил сервер Zabbix и агент на другой машине, добавил хост, настроил сбор метрик, создал триггер и визуализировал данные в Dashboard. Работа показала, как Zabbix позволяет мониторить состояние ИТ-инфраструктуры в реальном времени и своевременно реагировать на критические события.

