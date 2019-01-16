<a href="https://github.com/mnesina/cookbook/blob/master/README.md">Оглавление</a>
<h1>Vagrant - проблемы</h1>
<h2>1) rsync</h2>
<h2>2) vagrant global-status и ~/.vagrant.d/data/machine-index/index</h2>

в какой-то момент команда  `vagrant global-status`  превратилась в тыкву и перестала приносить ID виртуалок. 
Сами эти ID ушли туда же и перестали распознаваться. т.е. команда вида 

```bash
vagrant rsync 637b72c
```
не работала и кричала, что такой машины с таким ID не знает и знать не желает. 
При этом виртуалки подымались, работали и с ними все было хорошо.

Всемогущий google ехидно предлагал уничтожить и пересоздать все эти виртуальные среды, чего мне, естественно не хотелось.

Помогло ковыряние в файлах и наличие виртуалок на разных машинах.

Оказалось, что файл `~/.vagrant.d/data/machine-index/index` обнулился =>
Вытаскиваем аналогичный файл с другой машины:

```bash
cat ~/.vagrant.d/data/machine-index/index
```
```
{"version":1,"machines":{"ee3d590faac24b0e9f883726d9a28803":{"local_data_path":"/home/mar/www/null_new_test_ansible/.vagrant","name":"default","provider":"virtualbox","state":"running","vagrantfile_name":null,"vagrantfile_path":"/home/mar/www/null_new_test_ansible","updated_at":null,"extra_data":{"box":{"name":"centos/7","provider":"virtualbox","version":"1609.01"}}},"42e76840739d4b8fbe99392a06d01a47":{"local_data_path":"/home/mar/www/null_new_test/.vagrant","name":"default","provider":"virtualbox","state":"running","vagrantfile_name":null,"vagrantfile_path":"/home/mar/www/null_new_test","updated_at":null,"extra_data":{"box":{"name":"centos/7","provider":"virtualbox","version":"1609.01"}}},"960343bb1f3b483aa92f5e95dcb2844b":{"local_data_path":"/home/mar/www/null_new_test_php/.vagrant","name":"default",
"provider":"virtualbox","state":"running","vagrantfile_name":null,"vagrantfile_path":"/home/mar/www/null_new_test_php","updated_at":null,"extra_data":{"box":{"name":"centos/7","provider":"virtualbox","version":"1609.01"}}}}}
```
корректируем для нашей машины ID виртуалок (берем их из соответствующий_каталог/.vagrant/machines/default/virtualbox/index_uuid и каталоги. В моем случае все остальное было правильным. Убираем лишние машины, или добавляем необходимые.  Сохраняем это в `~/.vagrant.d/data/machine-index/index`

запускаем vagrant global-status и получаем валидный ответ. Вуаля

Собственно, если этот файлик сразу сохранить куда-нибудь в загашник, 
то в случае чего можно просто работать с ним, или накатить его сверху (ID машин при этом все-таки нужно будет проверить)
