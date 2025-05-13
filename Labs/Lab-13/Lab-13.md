# Лабораторная работа №13 - Асимметричная криптография в Linux с использованием GnuPG (GPG)  

#### **Цель работы**  
Освоить основы асимметричной криптографии на Linux с помощью **GnuPG (GPG)** — стандартного инструмента для шифрования, подписи и проверки данных.  

---

## **1. Подготовка стенда**  
**Используется 1 виртуальная машина** с ОС Linux (Ubuntu/Debian).  

### **1.1. Установка GnuPG**  
```bash
sudo apt update
sudo apt install gnupg -y
```  

### **1.2. Проверка установки**  
```bash
gpg --version
```  
*Вывод должен содержать версию GnuPG (например, `gpg (GnuPG) 2.2.19`).*  

---

## **2. Генерация ключевой пары**  

### **2.1. Создание пары RSA**  
```bash
gpg --full-generate-key
```  
1. Выберите тип ключа: **RSA (1)**.  
2. Размер ключа: **4096 бит**.  
3. Срок действия: **0 (ключ не истекает)**.  
4. Введите данные пользователя:  
   - Имя: `Alice`  
   - Email: `alice@example.com`  
   - Комментарий: `Lab Key`  
5. Подтвердите ввод.  
6. Задайте **пароль** для защиты закрытого ключа.  

### **2.2. Проверка ключей**  
```bash
gpg --list-keys
```  
Вывод:  
```text
pub   rsa4096 2025-04-01 [SC]
      ABCD1234ABCD1234ABCD1234ABCD1234ABCD1234
uid           [ultimate] Alice <alice@example.com>
sub   rsa4096 2025-04-01 [E]
```  

---

## **3. Экспорт и импорт ключей**  

### **3.1. Экспорт открытого ключа**  
```bash
gpg --export -a "Alice" > alice_public.key
```  
*Файл `alice_public.key` можно передавать другим пользователям.*  

### **3.2. Экспорт закрытого ключа (осторожно!)**  
```bash
gpg --export-secret-key -a "Alice" > alice_private.key
```  
**Важно:** Закрытый ключ должен храниться в безопасном месте!  

### **3.3. Импорт открытого ключа**  
Если другой пользователь хочет добавить ваш ключ:  
```bash
gpg --import alice_public.key
```  

---

## **4. Подпись и проверка документа**  

### **4.1. Создание тестового файла**  
```bash
echo "This is a secret message for Bob." > document.txt
```  

### **4.2. Подпись файла**  
```bash
gpg --sign --output document.sig document.txt
```  
- Файл `document.sig` содержит подпись + исходный текст.  

#### **Вариант: Отдельная подпись (detached signature)**  
```bash
gpg --detach-sign --output document.sig document.txt
```  
*Создается только файл подписи (`document.sig`), исходный файл остается неизменным.*  

### **4.3. Проверка подписи**  
```bash
gpg --verify document.sig document.txt
```  
**Пример вывода при успешной проверке:**  
```text
gpg: Signature made Sat 01 Apr 2025 12:00:00 PM UTC
gpg:                using RSA key ABCD1234ABCD1234ABCD1234ABCD1234ABCD1234
gpg: Good signature from "Alice <alice@example.com>" [ultimate]
```  

**Если подпись неверна:**  
```text
gpg: BAD signature from "Alice <alice@example.com>"
```  

---

## **5. Шифрование и расшифровка файла**  

### **5.1. Шифрование файла для получателя**  
```bash
gpg --encrypt --recipient "Alice" --output document.encrypted document.txt
```  
- Файл `document.encrypted` можно расшифровать только закрытым ключом Alice.  

### **5.2. Расшифровка файла**  
```bash
gpg --decrypt --output document.decrypted document.encrypted
```  
*GPG запросит пароль от закрытого ключа.*  

---

## **6. Удаление ключей (опционально)**  

### **6.1. Удаление закрытого ключа**  
```bash
gpg --delete-secret-key "Alice"
```  

### **6.2. Удаление открытого ключа**  
```bash
gpg --delete-key "Alice"
```  

---

### **Дополнительные задания**  
1. Настройте **автоматическую проверку подписей** в Git:  
   ```bash
   git config --global commit.gpgsign true
   ```  
2. Создайте **отзыв сертификата (revocation certificate)** на случай компрометации ключа:  
   ```bash
   gpg --gen-revoke "Alice" > alice_revoke.asc
   ```  