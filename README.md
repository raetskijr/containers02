# Лабораторная работа №2

**Тема:** Debian Server в QEMU, LAMP, phpMyAdmin и WordPress  
**Студент:** Raetchi Danila  
**Группа:** I2402   

## 1. Задание

Требуется подготовить виртуальную машину Debian Server x64 в QEMU, установить серверное окружение LAMP, развернуть phpMyAdmin и WordPress, настроить Apache VirtualHost и проверить открытие сайтов через порт `1080` в Windows.

## 2. Файлы в проекте

- `.gitignore` - исключает ISO, ZIP-архивы и виртуальные диски.
- `dvd/readme.md` - пояснение для папки с установочным ISO.
- `apache/01-phpmyadmin.conf` - конфигурация сайта phpMyAdmin.
- `apache/02-wordpress.conf` - конфигурация сайта WordPress.
- `notes/commands.md` - краткая памятка с командами.
- `readme.md` - отчет.

Файлы `debian.iso`, `debian.qcow2` и скачанные ZIP-архивы не добавляются в репозиторий.

## 3. Подготовка

ISO-образ Debian Server был сохранен как:

```text
dvd/debian.iso
```

Виртуальный диск создается так:

```text
qemu-img create -f qcow2 debian.qcow2 8G
```

Пример вывода:

```text
Formatting 'debian.qcow2', fmt=qcow2 size=8589934592 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

Установка Debian:

```text
qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G
```

При установке использованы параметры:

- имя компьютера: `debian`;
- хостовое имя: `debian.localhost`;
- пользователь: `user`;
- пароль: `password`.

## 4. Запуск установленной системы

После установки виртуальная машина запускается командой:

```text
qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22
```

Порт `1080` на Windows перенаправляется на порт `80` в Debian. Порт `1022` перенаправляется на SSH-порт `22`.

## 5. Установка LAMP

В Debian выполняется:

```text
su
apt update -y
apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip
```

Назначение пакетов:

- `apache2` - HTTP-сервер;
- `php` - язык выполнения серверных скриптов;
- `libapache2-mod-php` - запуск PHP через Apache;
- `php-mysql` - подключение PHP к MariaDB;
- `mariadb-server` - сервер базы данных;
- `mariadb-client` - консольный клиент;
- `unzip` - распаковка ZIP-файлов.

## 6. phpMyAdmin и WordPress

Загрузка:

```text
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip
wget https://wordpress.org/latest.zip
```

Проверка файлов:

```text
ls -l
```

Пример:

```text
-rw-r--r-- 1 root root 15024774 Apr 19 12:05 phpMyAdmin-5.2.2-all-languages.zip
-rw-r--r-- 1 root root 26771790 Apr 19 12:06 latest.zip
```

Распаковка:

```text
mkdir /var/www
unzip phpMyAdmin-5.2.2-all-languages.zip
mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin
unzip latest.zip
mv wordpress /var/www/wordpress
```

## 7. База данных WordPress

В MariaDB:

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
```

Так WordPress получает отдельную базу и отдельного пользователя.

## 8. Apache VirtualHost

Для phpMyAdmin используется `01-phpmyadmin.conf`, для WordPress - `02-wordpress.conf`. Копии этих файлов находятся в папке `apache`.

Активация сайтов:

```text
/usr/sbin/a2ensite 01-phpmyadmin
/usr/sbin/a2ensite 02-wordpress
```

В `/etc/hosts` добавлены строки:

```text
127.0.0.1 phpmyadmin.localhost
127.0.0.1 wordpress.localhost
```

Перезагрузка Apache:

```text
systemctl reload apache2
```

## 9. Проверка

Команда:

```text
uname -a
```

Пример вывода:

```text
Linux debian 6.1.0-18-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.76-1 (2024-02-01) x86_64 GNU/Linux
```

Адреса для проверки:

```text
http://wordpress.localhost:1080
http://phpmyadmin.localhost:1080
```

Оба адреса должны открываться через Apache в виртуальной машине.

## 10. Ответы на вопросы

**Как скачать файл через `wget`?**  
Нужно указать URL:

```text
wget https://example.com/archive.zip
```

**Зачем каждому сайту своя база и пользователь?**  
Это повышает изоляцию. Один сайт не должен иметь доступ к данным другого сайта.

**Как поменять доступ к phpMyAdmin на порт `1234`?**  
Достаточно изменить проброс QEMU на `hostfwd=tcp::1234-:80` и затем открывать `http://phpmyadmin.localhost:1234`. Если требуется менять порт MariaDB, он задается в конфигурации сервера MariaDB.

**Какие преимущества дает виртуализация?**  
Она позволяет безопасно тестировать сервер, не меняя основную систему. Виртуальную машину можно удалить, пересоздать или перенести.

**Зачем на сервере правильная временная зона?**  
Она нужна для корректных логов, планировщика задач, сертификатов, сессий и диагностики ошибок.

**Сколько места занимает установленная ОС?**  
Файл `debian.qcow2` имеет лимит 8 ГБ, но реально после установки Debian и LAMP занимает примерно 3 ГБ, так как `qcow2` растет по мере заполнения.

**Как рекомендуется разбивать диск сервера?**  
Обычно отдельно выделяют `/`, `/var`, `/home`, `swap`, иногда `/tmp` и `/var/log`. Это снижает риск, что переполнение одного раздела остановит всю систему.

## 11. Вывод

В работе была подготовлена виртуальная Debian-среда с Apache, PHP, MariaDB, phpMyAdmin и WordPress. Задание помогает понять, как связаны виртуализация, веб-сервер, база данных, DNS-имена в hosts-файле и проброс портов.

## Источники

- QEMU Documentation.
- Debian Documentation.
- Apache HTTP Server Documentation.
- MariaDB Documentation.
- WordPress и phpMyAdmin Documentation.
