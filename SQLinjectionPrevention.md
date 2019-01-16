
<a href="README.md">Оглавление</a>
<h1>SQL injection prevention tips</h1>
<h2>1) in nginx.conf :</h2>

```bash
if ($args ~* '(select|union|update|insert|table|ascii|hex|unhex|drop)' ) { return 403; }
```

<h2>2) in php first file i.e. index.php</h2>

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

<h2>3) in php database using lib i.e. db.php</h2> 
