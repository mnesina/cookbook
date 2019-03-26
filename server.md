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
    30 3 * * * /usr/bin/rsync -Wav zzz.zzz.zzz.zzz::site/  /home/user/dump/zzz.zzz.zzz.zzz/site/www/ ## сюда можно вставить исключения для каких-то каталогов и директиву на удаление того, что исчезает на сервере

    50 3 * * * /usr/bin/rsync -Wav zzz.zzz.zzz.zzz::db/  /home/user/dump/zzz.zzz.zzz.zzz/site/db/
    # и тоже не забываем чистить `/home/user/dump/zzz.zzz.zzz.zzz/site/db/`, видимо, сохраняя еще и раз в неделю в отдельный каталог
    # сохраненное дерево каталогов желательно (особенно, если там удаляется то, что удалено на сервере) gzip-ить по крону раз в сутки и копировать куда-нибудь тоже удаляя лишние копии

    ```
  * Первый rsync (по крайней мере дерева каталогов) делаем вручную, чтобы потом закачивать уже только разницу
  
  
  

## Чистим каталоги с устаревающей информацией


## Проверка скорости загрузки web-страниц

`https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Fwww.myserver.ru%2F&tab=desktop`

## nginx и gzip

Для сжатия файлов `js` в `nginx.conf` к `gzip_types` надо кроме стандартного `application/x-javascript` добавить: `application/javascript`  и  `text/javascript`


## nginx и статика

Имеем отдельный сервер со статикой, которая именно статика (картинки). Говорим `nginx.conf`, чтобы в HTTP - загловке на статику сервер отдавал `expires` побольше, например:

```
expires +1d;
```
чтобы браузер пользователя загружал уже полученные ранее ресурсы с локального диска, а не из сети.

## sshd
и, хотя это дело админа, все-таки (почему-то очень часто об этом забывают, а то и прямо спрашивают на форумах как войти рутом =) ):
прописываем в `/etc/ssh/sshd_config`

```bash
PermitRootLogin no
```
и даем команду:

```bash
systemctl restart sshd.service
```
## Запуск единственного экземпляра скрипта и проблема перезагрузуи (flock)
