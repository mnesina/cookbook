
Inline `code` has `back-ticks around` it.

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```

```PHP
$s = 1+2;
echo($s);
```
TEST - 1
```bash
git clone https://github.com/mnesina/cookbook.git


git add test.md 
git commit -m 'test commit'
git push origin master

git pull origin master
```

<a href="README.md">Оглавление</a>

# Сервера: дампы, место на диске и т.д. - проблемы и решения 

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
 
 

## Чистим каталоги с устаревающей информацией

