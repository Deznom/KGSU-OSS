# Команды Linux. Продвинутый уровень.

## Полезные команды для продвинутого использования

1. `cat  `
**Синтаксис:** `cat [имя_файла]`  
**Описание:** Используется для просмотра содержимого текстовых файлов.  
**Пример:** `cat test.txt` выведет содержимое файла test.txt.

2. `head`  
**Синтаксис:** `head [количество_строк] [имя_файла]`  
**Описание:** Выводит первые строки файла.  
**Пример:** `head -2 test.txt` выведет первые две строки файла `test.txt`.

3. `tail`  
**Синтаксис:** `tail [количество_строк] [имя_файла]`
**Описание:** Выводит последние строки файла.  
**Пример:** `tail -4 test.txt` выведет последние четыре 
строки файла `test.txt`.

4. `grep`  
**Синтаксис:** `grep [шаблон] [имя_файла]`
**Описание:** Используется для поиска строк в файлах или директориях.  
**Пример:** `grep Тест test_test.txt` найдет строку “Тест” в файле `test_test.txt`.

5. `sed`  
**Синтаксис:** `sed 'начало,конец!d' [имя_файла]`
**Описание:** Используется для редактирования строк в файле.
**Пример:** `sed '2,3!d' test.txt` удалит строки 2 и 3 из файла `test.txt`.

### Создание файлов:

1. `touch`  
**Синтаксис:** `touch [имя_файла]`  
**Описание:** Создает пустой файл.  
**Пример:** `touch test.txt` создаст пустой файл `test.txt`.

2. `echo`  
**Синтаксис:** `echo [текст] > [имя_файла]`  
**Описание:** Выводит текст в файл (stdout тоже считается файлом, можно не указывать `> [имя файла]`, тогда `echo` выведет текст в консоль).  
**Пример:** `echo "Привет, мир!" > test.txt` создаст файл `test.txt` с текстом “Привет, мир!”.

### Различие между операторами перенаправления вывода > и >>:
- Оператор `>`  
**Синтаксис:** `echo текст > имя_файла`  
**Описание:** Этот оператор перезаписывает содержимое файла. Если файл не существует, он будет создан.  
Пример: `echo "Привет, мир!" > test.txt` создаст файл `test.txt` с текстом “Привет, мир!”. Если файл уже существует, его содержимое будет заменено на “Привет, мир!”.

- Оператор `>>`  
**Синтаксис:** `echo текст >> имя_файла`  
**Описание:** Этот оператор добавляет текст в конец файла. Если файл не существует, он будет создан.  
Пример: `echo "Привет, мир!" >> test.txt` добавит строку “Привет, мир!” в конец файла `test.txt`. Если файл уже существует, новая строка будет добавлена в его конец.

**Обратите внимание**, что эти операторы можно использовать с **любой** командой, которая генерирует какой-либо вывод в консоль.

## Command Pipelining в Linux

1. `command1 && command2`  
Выполняет `command1`, затем, если `command1` завершилась успешно (с нулевым кодом выхода), выполняет `command2`.  
`mkdir new_dir && cd new_dir`  
Эта команда создает новый каталог `new_dir` и затем переходит в него.

2. `command1 || command2`  
Выполняет `command1`, затем, если `command1` завершилась неудачно (с ненулевым кодом выхода), выполняет `command2`.  
`mkdir new_dir || echo "Ошибка при создании каталога"`  
Эта команда пытается создать каталог `new_dir`, и если это не удается, выводит сообщение об ошибке.

3. `command1 | command2`  
Выполняет `command1`, передавая его стандартный вывод на стандартный ввод `command2`.  
`ls -l | grep "txt"`  
Эта команда выводит список файлов и директорий, а затем передает его на `grep`, который фильтрует строки, содержащие “txt”.  

4. `command1 |& command2`  
Аналогично `command1 | command2`, но передает как стандартный вывод, так и данные об ошибках в command2.  
`ls -l |& grep "txt"`  
Эта команда передает как стандартный вывод, так и данные об ошибках в `grep`, который фильтрует строки, содержащие “txt”.

5. `command1; command2`  
Выполняет `command1`, затем `command2`, независимо от кода выхода command1.  
`mkdir new_dir; cd new_dir`  
Эта команда создает новый каталог `new_dir` и затем переходит в него, независимо от успешности создания каталога.

6. `command1 & command2 &`  
Выполняет command1 и command2 как фоновые задачи.  
`mkdir new_dir & cd new_dir &`  
Эта команда создает новый каталог `new_dir` и затем переходит в него, запуская обе команды в фоновом режиме.