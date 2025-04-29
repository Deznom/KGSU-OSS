# Лабораторная работа №10 - Проксирование трафика с использованием Squid  

#### **Цель работы:**  
Освоить навыки настройки и администрирования прокси-сервера **Squid** в корпоративной сети, включая:  
- Разграничение доступа по правилам  
- Реализацию аутентификации через СУБД MariaDB  
- Мониторинг интернет-активности  

---

## **1. Подготовка виртуального стенда**  

### **1.1. Состав стенда**  
| Машина       | ОС                | Роль                     | IP-адрес      |  
|--------------|-------------------|--------------------------|---------------|  
| **VM1**      | Debian Linux      | Прокси-сервер (Squid)    | 192.168.56.2  |  
| **VM2**      | Linux (без GUI)   | Клиент (ограниченный доступ) | 192.168.56.10 |  
| **VM3**      | Windows 7-10      | Клиент (полный доступ)   | 192.168.56.100|  

### **1.2. Настройка сети**  
- **VM1**:  
  - Интерфейс 1: **NAT** (доступ в интернет)  
  - Интерфейс 2: **Host-only** (`192.168.56.2`)  
- **VM2** и **VM3**:  
  - **Host-only** (`192.168.56.10` и `192.168.56.100`)  

**Проверка:**  
```bash
ping 192.168.56.10  # с VM1 → VM2
ping 192.168.56.100 # с VM1 → VM3
```

---

## **2. Установка и базовая настройка Squid**  

### **2.1. Установка Squid на VM1**  
```bash
sudo apt update
sudo apt install squid -y
sudo systemctl start squid
sudo systemctl enable squid
```  

### **2.2. Конфигурация доступа**  
Редактируем `/etc/squid/squid.conf`:  

#### **Для VM2 (только репозитории Debian)**  
```nginx
acl vm2 src 192.168.56.10
acl debian_repos dstdomain .debian.org
http_access allow vm2 debian_repos
http_access deny vm2
```  

#### **Для VM3 (запрет "черного списка")**  
1. Создаем файл `/etc/squid/blacklist.txt`:  
   ```text
   youtube.com
   facebook.com
   ```  
2. Добавляем в конфиг:  
   ```nginx
   acl vm3 src 192.168.56.100
   acl blacklist dstdomain "/etc/squid/blacklist.txt"
   http_access deny blacklist
   http_access allow vm3
   ```  

#### **Перезапуск Squid**  
```bash
sudo systemctl restart squid
```  

---

## **3. Настройка аутентификации через MariaDB**  

### **3.1. Установка MariaDB**  
```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation
```  

### **3.2. Создание БД и пользователей**  
```sql
CREATE DATABASE squid;
USE squid;
CREATE TABLE passwd (
    user varchar(32) NOT NULL default '',
    password varchar(35) NOT NULL default '',
    enabled tinyint(1) NOT NULL default '1',
    PRIMARY KEY (user)
);
INSERT INTO passwd VALUES ('testuser', 'test', 1);
GRANT ALL ON squid.* TO 'squiduser'@'localhost' IDENTIFIED BY '12345678';
FLUSH PRIVILEGES;
```  

### **3.3. Интеграция с Squid**  
Добавить в `/etc/squid/squid.conf`:  
```nginx
auth_param basic program /usr/lib/squid/basic_db_auth --user squiduser --password 12345678 --plaintext
auth_param basic realm "Squid Proxy"
acl db-auth proxy_auth REQUIRED
http_access allow db-auth
```  

**Порядок правил:**  
```nginx
http_access allow localhost
http_access allow db-auth
http_access deny all
```  

### **3.4. Проверка аутентификации**  
```bash
curl -x http://testuser:test@192.168.56.2:3128 http://example.com
```  

---

## **4. Мониторинг трафика**  

### **4.1. Логи доступа**  
```bash
tail -f /var/log/squid/access.log
```  

### **4.2. Блокировка нежелательных запросов**  
Пример лога при блокировке:  
```text
1654321000.000      0 192.168.56.100 TCP_DENIED/403 GET http://youtube.com/ - NONE/- text/html
```  

---

### **FAQ**  
**Q:** Как добавить больше пользователей?  
**A:** Вставьте новые записи в таблицу `passwd`:  
```sql
INSERT INTO passwd VALUES ('admin', 'securepass123', 1);
```  

**Q:** Почему не работает аутентификация?  
**A:** Проверьте:  
- Доступность MySQL (`sudo systemctl status mariadb`).  
- Пароль в конфиге Squid.  
- Логи (`/var/log/squid/access.log`, `/var/log/squid/cache.log`).  
