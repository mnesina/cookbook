<a href="https://github.com/mnesina/cookbook/blob/master/README.md">Оглавление</a>
<h1>SVN - разные полезности</h1>
Иногда svn переглючивает. Поскольку бывает это редко, то пути решения забываются. Тут будут разные разности

<h2>1) Сбои при svn update</h2>
<p>При попытке сделать svn update появилась ошибка из-за того, что один из файлов с таким же названием, как тот, 
что должен был появиться в результате update-а,- уже существовал. 
Попытка удалить файл и сделать update ни к чему не привела, файл не появился. 
Поскольку речь шла о рабочем сервере пришлось загрузит файл по scp. После этого было так:</p>
<pre>
$svn status 
D C some_file.txt
> local unversioned, incoming add upon update
Summary of conflicts: 
Tree conflicts: 1
</pre>
<b>Решение проблемы:</b>
<pre>
$ svn resolve —accept working some_file.txt
Resolved conflicted state of 'some_file.txt' 
$ svn revert some_file.txt
Reverted 'some_file.txt'
$ svn status
</pre>
