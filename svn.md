<a href="README.md">Оглавление</a>

# SVN - разные полезности
Иногда svn переглючивает. Поскольку бывает это редко, то пути решения забываются. Тут будут разные разности

## 1) Сбои при svn update
<p>При попытке сделать svn update появилась ошибка из-за того, что один из файлов с таким же названием, как тот, 
что должен был появиться в результате update-а,- уже существовал. 
Попытка удалить файл и сделать update ни к чему не привела, файл не появился. 
Поскольку речь шла о рабочем сервере пришлось загрузит файл по scp. После этого было так:</p>

```bash
$svn status 
D C some_file.txt
> local unversioned, incoming add upon update
Summary of conflicts: 
Tree conflicts: 1
```

<b>Решение проблемы:</b>

```bash
$ svn resolve —accept working some_file.txt
Resolved conflicted state of 'some_file.txt' 
$ svn revert some_file.txt
Reverted 'some_file.txt'
$ svn status
```
