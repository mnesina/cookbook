<a href="README.md">Оглавление</a>

# Сервера: дампы, место на диске и т.д. - проблемы и решения 

Мой собственный сисадминский опыт уже давно устарел, поэтому тут - только немного полезного мне как разработчику

## Дампы для относительно небольших проектов (с использованием mysql)

* со стороны сервера: 
  * устанавливаем rsyncd 
  * `/etc/rsyncd.conf`

     ```
    pid file = /var/run/rsyncd.pid
    uid = root
    gid = root # or wheel if FreeBSD

    [site]
    path = /path_to_site
    read only = yes
    list = false
    hosts allow = first_host second_host

    [db]
    path = /path_to_backup/db/
    read only = yes
    list = false
    hosts allow = first_host second_host
    ```
    
  * `/etc/hosts` - на случай изменений IP удобней в конфигурационных фалах использовать не IP, а имена хостов 

    ```bash
    xxx.xxx.xxx.xxx first_host
    yyy.yyy.yyy.yyy second_host
    ```
  * если храним дампы за несколько дней на сервере, то пишем скрипт для дампа базы данных

    ```bash
    #!/bin/sh

    
    
    /usr/local/bin/mysqldump -R -uusername -ppassword dbname  > /path_to_backup/db/dbname_`date +%s`.sql
    
    # или, если база небольшая, и дамп сразу с gzip:
    /usr/local/bin/mysqldump -R -uusername -ppassword dbname | gzip > /path_to_backup/db/dbname_`date +%s`.sql.gz

    ```
  * пользовательский `crontab`
    
    ```bash
    45 1 * * * /usr/local/bin/mysqldump -R -uusername -ppassword dbname  > /path_to_backup/db/dbname.sql
    # или, если база небольшая, и дамп сразу с gzip:
    45 1 * * * /usr/local/bin/mysqldump -R -uusername -ppassword dbname | gzip > /path_to_backup/db/dbname.sql.gz
    
    # или, если храним базы данных за несколько дней, то:
    45 1 * * * /bin/sh /path_to_backup_script/backup_script.sh
    
    # и, если храним базы данных за несколько дней, то чистим (например, раз в 5 дней):
    5 2 * * * /usr/bin/find /path_to_backup/db/ -type f -name "*.sql.*"  -mtime +5 -exec rm -rf {} \;
    ```

* со стороны клиентских машин: 
  * устанавливаем `rsync`
  * пользовательский `crontab`
    ```bash
    30 3 * * * /usr/bin/rsync -Wav zzz.zzz.zzz.zzz::site/  /home/user/dump/zzz.zzz.zzz.zzz/site/www/

    50 3 * * * /usr/bin/rsync -Wav zzz.zzz.zzz.zzz::db/  /home/user/dump/zzz.zzz.zzz.zzz/site/db/
    # и тоже не забываем чистить базы, видимо, сохраняя еще и раз в неделю в отдельный каталог

    ```
    
  
  

## Чистим каталоги с устаревающей информацией


## Проверка скорости загрузки web-страниц

`https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Fwww.myserver.ru%2F&tab=desktop`

## nginx и gzip

Для сжатия файлов `js` в `nginx.conf` к `gzip_types` надо кроме стандартного `application/x-javascript` добавить: `application/javascript`  и  `text/javascript`


