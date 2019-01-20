<a href="README.md">Оглавление</a>

# PHP - problems & solutions. PHP - проблемы и решения 

## PHP+Apache, баг - невидимка. Команда echo и не всегдла уловимые тормоза при выводе. Буфер выходного потока

Эта проблема была [описана][1] еще в 2002 году и, судя по отсутствию отклика, до сих пор подстерегает нас.
Суть ее в том, что в при работе приложения на web-сервере команда `echo` может очень сильно тормозить. Но тормоза эти заметны не всегда, а при выдаче `echo` большого объема информации (что естественно) 
и только, если у Вас плохая связь с сервером (что уже может поставить в тупик). Подчеркну, что речь идет не о тормозах сети, а именно о времени генерации страницы, "затыкающейся" на команде `echo`.

Суть проблемы детально описана в статье на habr ["Функция echo в PHP может выполняться более 1 секунды" (автор gnomeby)][2]
Из предложенных решений как в первом случае, когда у нас выявился этот баг, так и привентивно в дальнейшем было использовано `php_flag output_buffering On` 
Следует отметить, что, насколько я понимаю, наличие `nginx` перед `Apache` также должно вылечивать эту ситуацию т.к. ответ `Apache` поучает теперь не от пользователя, а от `nginx`, находящегося рядом.

Важно понимать, что проблема может не наблюдаться разработчиками, имеющих хорошую связь с сервером, но в полный рост "радовать" удаленных пользователей. Поэтому, если мы имеем дело с `cms` в котрой страница отдается при помощи `echo`, меры нужно принимать заранее =)

[1]: https://bugs.php.net/bug.php?id=18029
[2]: https://habr.com/ru/post/45016/

## Memcache в PHP5 и PHP7
(Подробней о работе с memcache [у меня тут](PHPDBMemcache.md)
При переходе с PHP5 на PHP7 сталкиваемся с проблемой, [описанной][3] еще в 2016 году: перестала работать установка memcache: `sudo pecl install memcache`. Собственно, поддержка оказывается просто убранной и нам остается либо: 
* работать с неофициальной библиотекой [pecl-memcache][4]  (а точнее: https://github.com/websupport-sk/pecl-memcache/archive/php7.zip) и тогда код остается прежним
* либо идти через memcache-d В этом случае по-разному идет работа с данными:
    ```PHP
    # memcache:
    $memcache = new Memcache;
    $memcache->addServer($host);
    # memcacheD
    $memcacheD = new Memcached;
    $memcacheD->addServers($servers);
    ```
    Собственно, для совместимости старого и нового достаточно:
    *  разницы в объявлении
    ```PHP
    if(DO_MEMCACHED) { // PHP7
        $memcache = new Memcached();
        $memcache->addServers(array(array(MEMCACHE_HOST, MEMCACHE_PORT)));
    } else { // PHP5
        $memcache = new Memcache();
        $memcache->addServer(MEMCACHE_HOST, MEMCACHE_PORT);

    }
    ```
    * отправке данных
    ```PHP
      if(DO_MEMCACHED) {
            $memcache->set($key, $row, $expiration);
      } else {
            $memcache->set($key, $row, MEMCACHE_COMPRESSED, $expiration);
      }
    ```
    * и удалении ключей (для memcacheD это проще - `$memcache->deleteMulti(array($pattern));`)
    
* см. также http://php.net/manual/ru/book.memcached.php
    
<!--
Maryanna Nesina, [07.05.18 16:16]
я под php7 для CentOS на виртуалке с memcache, помнится, плясала с бубном - оно просто так не вставало

Maryanna Nesina, [07.05.18 16:16]
но у меня еще могли наклыдываться проблемы с ansible

Maryanna Nesina, [07.05.18 16:16]
и с тем, что http://rpms.remirepo.net  не работал и я в результате брала тут: https://github.com/websupport-sk/pecl-memcache/archive/php7.zip -->
[3]: https://bugs.php.net/bug.php?id=72887
[4]: https://github.com/websupport-sk/pecl-memcache

