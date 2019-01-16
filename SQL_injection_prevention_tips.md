
<h1>SQL injection prevention tips</h1>
1) <h2>in nginx.conf :</h2>
<pre>
if ($args ~* '(select|union|update|insert|table|ascii|hex|unhex|drop)' ) { return 403; }
</pre>
2) <h2>in php first file i.e. index.php</h2>
<pre>
$queryStringArr = explode('&', ($_SERVER['QUERY_STRING']));
foreach ($queryStringArr as $q) {
    $qArr = explode("=", $q);
    if(isset($qArr[1])) {
        if (preg_match('/select|update|delete|insert|table|union|join|hex|unhex|drop/i',$qArr[1]))
            redirect("/"); # or do smth else
    }

}
</pre>
3) <h2>in php database using lib i.e. db.lib</h2> 
