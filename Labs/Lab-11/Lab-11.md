### Лабораторная работа №11. Настройка Nginx в качестве обратного прокси-сервера

#### Цель работы:
Освоить настройку Nginx в качестве обратного прокси-сервера для обеспечения доступа к веб-ресурсам, расположенным на внутренних серверах, с использованием различных методов аутентификации и контроля доступа.

---

### 1. Подготовка виртуального стенда

1.1. Подготовьте виртуальный стенд, состоящий из 3-х машин:
- **VM1**: Сервер на базе Debian Linux, который будет выступать в роли обратного прокси-сервера (Nginx).
- **VM2**: Рабочая станция с ОС Linux без графического интерфейса.
- **VM3**: Рабочая станция с ОС Windows 7-10.

1.2. Настройте сетевые интерфейсы:
- **VM1**: Настройте два сетевых интерфейса:
  - Первый интерфейс: NAT (для доступа в интернет).
  - Второй интерфейс: Виртуальный адаптер хоста (для внутренней сети).
- **VM2**: Настройте один сетевой интерфейс с типом "виртуальный адаптер хоста".
- **VM3**: Настройте один сетевой интерфейс с типом "виртуальный адаптер хоста".

1.3. Назначьте IP-адреса:
- **VM1**: 
  - NAT: 192.168.56.2
  - Внутренний интерфейс: 192.168.56.1
- **VM2**: 192.168.56.10
- **VM3**: 192.168.56.100

1.4. Убедитесь, что все машины могут взаимодействовать друг с другом по сети (проверьте с помощью команды `ping`).

---

### 2. Установка и настройка Nginx на VM1

2.1. Установите Nginx на VM1:
```bash
sudo apt update
sudo apt install nginx -y
```

2.2. Запустите и включите Nginx:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

2.3. Проверьте статус Nginx:
```bash
sudo systemctl status nginx
```

2.4. Откройте конфигурационный файл Nginx для редактирования:
```bash
sudo nano /etc/nginx/nginx.conf
```

2.5. Настройте Nginx в качестве обратного прокси:
Добавьте следующий блок в конфигурацию:
```nginx
server {
    listen 80;
    server_name proxy.example.com;

    location / {
        proxy_pass http://192.168.56.10;  # Указываем адрес VM2
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

2.6. Проверьте конфигурацию Nginx на ошибки:
```bash
sudo nginx -t
```

2.7. Перезапустите Nginx для применения изменений:
```bash
sudo systemctl restart nginx
```

---

### 3. Настройка доступа к веб-серверу на VM2

3.1. На VM2 установите веб-сервер (например, Apache):
```bash
sudo apt update
sudo apt install apache2 -y
```

3.2. Создайте тестовую веб-страницу:
```bash
echo "<h1>Welcome to VM2</h1>" | sudo tee /var/www/html/index.html
```

3.3. Убедитесь, что веб-сервер работает:
```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

3.4. Проверьте доступность веб-сервера через браузер на VM3, используя IP-адрес VM1 (192.168.56.2).

---

### 4. Настройка аутентификации и контроля доступа

4.1. Установите MariaDB на VM1:
```bash
sudo apt install mariadb-server -y
```

4.2. Создайте базу данных и таблицу для аутентификации:
```bash
sudo mysql -u root -p
```
```sql
CREATE DATABASE nginx_auth;
USE nginx_auth;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(255) NOT NULL
);

INSERT INTO users (username, password) VALUES ('user1', PASSWORD('password1'));
INSERT INTO users (username, password) VALUES ('user2', PASSWORD('password2'));
```

4.3. Установите модуль для аутентификации через MySQL:
```bash
sudo apt install libnginx-mod-http-auth-pam -y
```

4.4. Настройте Nginx для использования аутентификации через MySQL:
Добавьте в конфигурацию Nginx:
```nginx
server {
    listen 80;
    server_name proxy.example.com;

    location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;

        proxy_pass http://192.168.56.10;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

4.5. Создайте файл `.htpasswd` для хранения учетных данных:
```bash
sudo sh -c "echo -n 'user1:' >> /etc/nginx/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

4.6. Перезапустите Nginx:
```bash
sudo systemctl restart nginx
```

---

### 5. Проверка работоспособности

5.1. На VM3 откройте браузер и попробуйте получить доступ к веб-серверу через IP-адрес VM1 (192.168.56.2).

5.2. Убедитесь, что запрашивается аутентификация.

5.3. Введите учетные данные, созданные в MariaDB, и проверьте доступ к веб-ресурсу.

---

### 6. Дополнительные задания (по желанию, на доп баллы)

6.1. Настройте SSL/TLS для Nginx, чтобы обеспечить безопасное соединение.

6.2. Добавьте ограничение доступа по IP-адресам для определенных ресурсов.

6.3. Настройте кэширование на Nginx для повышения производительности.
