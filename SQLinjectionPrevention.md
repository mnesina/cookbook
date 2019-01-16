
<a href="README.md">Оглавление</a>
# SQL injection prevention tips
## 1) in nginx.conf :

```bash
if ($args ~* '(select|union|update|insert|table|ascii|hex|unhex|drop)' ) { return 403; }
```

## 2) in php first file i.e. index.php :

```PHP
$queryStringArr = explode('&', ($_SERVER['QUERY_STRING']));
foreach ($queryStringArr as $q) {
    $qArr = explode("=", $q);
    if(isset($qArr[1])) {
        if (preg_match('/select|update|delete|insert|table|union|join|hex|unhex|drop/i',$qArr[1]))
            redirect("/"); # or do smth else
    }

}
```

## 3) in php database using lib i.e. db.php :

```PHP
// Показываем ошибку только тем, кому можно (он же может видеть, например, статистику по запросам)
$clientIp = getIp();
if (DB_STAT_IP=='all' || DB_STAT_IP==$clientIp || empty($clientIp))
    throw new Exception_SYS($query . ' ' . $this->mysqli->error);
else {
    redirect("/"); // или, например throw new Exception_SYS('приходите завтра');
}

// функция getIp():
/**
 * Получение IP адреса пользователя.
 *
 * @return string
 */
function getIp()
{
    if ($ip = getenv("HTTP_CLIENT_IP")) {
        return $ip;
    }
    if ($ip = getenv("HTTP_X_FORWARDED_FOR")) {
        if ($ip == '' || $ip == "unknown") {
            $ip = getenv("REMOTE_ADDR");
        }

        return $ip;
    }
    if ($ip = getenv("REMOTE_ADDR")) {
        return $ip;
    }

    return false;
}

// DB_STAT_IP берем из базы, или определяем в каком-нибудь settings.php

```
