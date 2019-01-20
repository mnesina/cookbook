<a href="README.md">Оглавление</a>

# Для класса работы с базами данных - работа с memcache на примере функции получения записи таблицы (упрощенный вид)


```PHP

    ## 1) В общих установках определяем глобальную переменную и параметры подключения. Например, так:
    /** Memcache **/
    define("MEMCACHE_HOST", "localhost");	/* Хост */
    define("MEMCACHE_PORT", 11211);	/* Порт */
    define("DO_MEMCACHE", true);	/* Включен ли мемкеш */
    define("DO_MEMCACHED", true);           /* Включен ли мемкеш-D */
    
    ## 2) Подключаем memcache (по-разному для PHP5 и PHP7)
    if(DO_MEMCACHE) {
        if(DO_MEMCACHED) { // PHP7
            $memcache = new Memcached();
            $memcache->addServers(array(array(MEMCACHE_HOST, MEMCACHE_PORT)));

        } else { // PHP5
            $memcache = new Memcache();
            $memcache->addServer(MEMCACHE_HOST, MEMCACHE_PORT);

        }
    }


    ## 3) Пример использования в классе работы с базой данных
    /**
     * Запрос с возвращением одной записи
     *
     * @param sring $query
     * @param bool $do_cache - if true - using Memcache if it is swiched on in the project
     * @param int $expiration - cache expiration in seconds (for Memcache)
     * @param string $cache_pref - cache preffix - for Memcache key
     *
     * @return array/bool
     */
    public function queryLine($query, $do_cache = false, $expiration = 0, $cache_pref = '')
    {

        if ($do_cache && DO_MEMCACHE) {
            global $memcache;

            $key = $cache_pref . md5($query);

            $result_tmp = $memcache->get($key); // пробуем получить ответ из мемкеша
            if (empty($result_tmp)) { // если этого не получилось, обращаемся с запросом к базе данных
                $result = $this->mysqli->query($query);
            } else {
                return $result_tmp; // если получилось, возвращаем результат
            }
        } else {
            $result = $this->mysqli->query($query);
        }


        if ($result) {
            $row = $result->fetch_assoc();
            $result->close();

            if ($do_cache && DO_MEMCACHE) { // помещаем результат в мемкеш
                if(DO_MEMCACHED) {
                    $memcache->set($key, $row, $expiration);
                } else {
                    $memcache->set($key, $row, MEMCACHE_COMPRESSED, $expiration);
                }
            }
            return $row;
        } else {
            // Показываем ошибку только тем, кому можно, для остальных, например - redirect("/");
            $clientIp = getIp();
            if (DB_STAT_IP==$clientIp) {
                throw new Exception($query . ' ' . $this->mysqli->error);
            } else {
                redirect("/"); // или, например throw new Exception_SYS('приходите завтра');
            }

            return false;
        }
    }
    
    ## 4) Функция удаления ключей (не в db, т.к. к ней приходится обращаться и из других частей кода, использующих memcache)
    
    ```PHP
    /**
    * Удаляет ключи memcached по маске
    *
    * @param string $pattern
    * @return bool - true/false
    */
    function delMemcacheKeys($pattern)
    {

        global $memcache;

        if(DO_MEMCACHE===false) {
            return false;
        }

        if(DO_MEMCACHED) { // PHP7
            $memcache->deleteMulti(array($pattern));
            return true;
        } else { // PHP5
            $allSlabs = $memcache->getExtendedStats('slabs');
            $items = $memcache->getExtendedStats('items');

            if (!is_array($allSlabs))
                return false;

            foreach ($allSlabs as $server => $slabs) { //--allSlabs

                if (!is_array($slabs))
                    continue;

                foreach ($slabs AS $slabId => $slabMeta) { //--$slabs

                    $slabIdInt = (int)$slabId;
                
                    if(empty($slabIdInt))
                    continue;
                    $cdump = $memcache->getExtendedStats('cachedump', $slabIdInt);

                    if (!is_array($cdump))
                        continue;

                    foreach ($cdump AS $keys => $arrVal) { //--$cdump
                        if (!is_array($arrVal))
                            continue;

                        foreach ($arrVal AS $k => $v)  { //--$arrVal
                            if (strstr($k, $pattern)) { // тут для разных версий 5-й ветки PHP отрабатывало по-разному
                                //$memcache->delete($k,0);
                                $memcache->set($k, 0);
                                //$memcache->set($k,NULL);
                            } // -- if
                        } //--$arrVal
                    } //--$cdump
                } //--$slabs
            } //--allSlabs

            return true;
        }
    }
    ```
 
```
# Работа с memcache и sphinx на примере одной из функций (упрощенный вариант)

```PHP
    /**
     * Подсчет общего числа попугаев (некие сущности, пусть будут Smth)
     *
     * @param array $filterArr - массив фильтров
     * @param array $setSellArr - дополнительный массив для фильтров (с использованием OR)
     *
     * @return int - общее число объявлений в выборке
     */

    public static function countSmthSearchSphinx($filterArr, $setSellArr, $ind = 'ind1,ind1_delta', $key_q_string = '', $nomemcache=false)
    {

        global $sphinx, $memcache;

        $q = ''; // сам текстовый запрос

        if (DO_MEMCACHE && !$nomemcache) {
            if (empty($key_q_string)) {
                $filterArrAtring = implode(",", array_keys($filterArr));
                $filterArrAtring .= implode(",", $filterArr);
                $setSellArrString = implode(",", $setSellArr);
                $key_q_string = "{$filterArrAtring}:{$setSellArrString}:{$ind}";
            }

            $cache_pref = "smth:countSmthSearchSphinx_";
            $keyMain = $cache_pref . md5($key_q_string);

            $result = $memcache->get($keyMain);


            if (!empty($result) && !$debug)
                return $result;

        }


        // Вызываем в начале т.к. иначе, если не было ничего найдено при предыдущем вызове, новый вызов отрабатывает некорректно
        $sphinx->ResetFilters();

        //$sphinx->SetMatchMode( SPH_MATCH_ANY  ); // ищем хотя бы 1 слово из поисковой фразы		

        $ids = array();

        if (!empty($setSellArr)) {
            $setSelString = implode(' AND ', $setSellArr);
            $setSelString = " ( $setSelString ) AS my_sel ";

            $sphinx->SetSelect("*, $setSelString ");
            $sphinx->SetFilter("my_sel", array(1));
        }

        // Ищем только активные
        if(empty($admin)) {
            if($ind == 'ind1,ind1_delta')
                $sphinx->SetFilter('active', array(1));
        }

        // Готовим фильтры
        if (!empty($filterArr)) {
            foreach ($filterArr as $key => $value) {
                if ($key == 'q') {
                    $q = $value;
                } else {
                    if (is_array($value)) {
                        $sphinx->SetFilter($key, (int)$value[0], (int)$value[1]);
                    } else {
                        $sphinx->SetFilter($key, array((int)$value));
                    }
                }

            }
        }


        // Если не указать метод сортировки в явном виде,
        // отыгрывает предыдущий и, при использовании разных индексов, может быть ошибка
        $sphinx->SetSortMode(SPH_SORT_EXTENDED,' @relevance desc');
        $result = $sphinx->Query($q, $ind /*'ind1,ind1_delta'*/);

        $res_count = 0;
        $total_found = 0;

        if ($result === false) {
            if ($debug) {
                echo "Query failed: " . $sphinx->GetLastError() . ".\n";
            }

            return 0;
        } else {

            if (isset($result['total_found']))
                $total_found = $result['total_found'];
            if (isset($result['total']))
                $res_count = $result['total'];

            unset($result);

        }

        if (DO_MEMCACHE && !$nomemcache) {

            $expiration = 600;

            if(DO_MEMCACHED) {
                $memcache->set($keyMain, $total_found, $expiration);
            } else {
                $memcache->set($keyMain, $total_found, MEMCACHE_COMPRESSED, $expiration);
            }
        }

        return $total_found; //$res_count;

    }

```
