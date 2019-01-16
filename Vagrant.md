<a href="https://github.com/mnesina/cookbook/blob/master/README.md">Оглавление</a>
# Vagrant - проблемы
## 1) rsync
Пусть у нас имеется некое приложение, над кодом которого мы работаем у себя, а тестируем в виртуалке, создав там машину при помощи vgrant и ansible. Собственно, задача автоматизировать синхронизацию наших изменений и того, что получается на тестовой площадке. Казалось бы для этого существует `rsync-auto` , однако желающим скормить эту строчку google, поисковик заботливо предложит продолжение `vagrant rsync auto not working` 
Решение:
1. создаем в корне папки нашего проекта файл `vagrant_rsync.py` Вот такой:

```python
#!/usr/bin/python
import os
import time

while True:
    os.system("flock -n /tmp/vagrant_rsync_sh.lock -c '/bin/sh /your_path/vagrant_rsync.sh 2>&1'")
    time.sleep(5)
```
или шаблон для него.
2. `vagrant_rsync_dot_sh` с кодом:

```bash
vagrant rsync {{id}}
```
3. Когда все, включая среду разрабортки и т.д. подготовлено, запускаем крон-скрипты для автоматического поддержания состояния файлов кода:
  * 
```bash
cp vagrant_rsync_dot_sh vagrant_rsync.sh
```
  * на машине-хозяине в папке пректа запускаем команду: `vagrant global-status` и среди прочих данных получаем `id` машины
  * вставляем этот `id` в файл `vagrant_rsync.sh` вместо `{{id}}` (например: было - `vagrant rsync {{id}}` стало - `vagrant rsync 569263c` )
  * добавляем в пользовательский крон строчку:

```bash
* * * * * flock -n /tmp/vagrant_rsync_py-cron-job.lock -c '/usr/bin/python3.4 /your_path/vagrant_rsync.py 2>&1'
```

## 2) vagrant global-status и ~/.vagrant.d/data/machine-index/index

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
