# Лабораторная работа №3 - Восстановление пароля в Windows 10

## Предварительные требования:
- ВМ с ОС Windows 10
- Загрузочный (установочный) образ Windows 10

## План выполнения работы:

### Восстановление пароля в ОС Windows 10, метод 1
- Загрузиться до «поиск и устранение неисправностей»
- Выбрать командную строку
- Выполнить regedit
- Выделить ветку ветку — HKEY_LOCAL_MACHINE
- Нажимаем «Файл» и выпадающем списке контекстного меню выбираем «Загрузить куст»
- Выбрать системный диск/windows/system32/config/файл SYSTEM
- Задаем имя, например, sysadmin
- Разворачиваем данный раздел, выбираем setup, выбираем параметр CmdLine, задаем для него значение – «cmd.exe»
- Выбираем параметр SetypType и ставим параметр 2  
- Файл – выход, извлекаем диск и перезагружаемся.
- Используем команды: net user, net user “admin” 0000, net user Administrator /active:yes

### Восстановление пароля в ОС Windows 10, метод 2
- Загрузиться до «поиск и устранение неисправностей»
- Выбрать командную строку
- Diskpart и list volume, exit
- move d:\windows\system32\sethc.exe d:\windows\system32\sethc2.exe — этой командой мы создаем дубликат утилиты для залипания клавиш.
- copy d:\windows\system32\cmd.exe d:\windows\system32\sethc.exe — этой мы подменяем утилиту для залипания клавиш командной строкой и меняем ей имя.
- Пять раз нажать Shift
- net user, net user “admin” 0000, net user Administrator /active:yes