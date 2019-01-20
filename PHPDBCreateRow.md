<a href="README.md">Оглавление</a>

# Для класса работы с базами данных - добавление записи к таблице


```PHP
    /**
     * Добавление записи к таблице
     *
     * @param string $table - таблица
     * @param array $data - массив данных
     * @param null/string $allowable_tags - разрешенные теги html 
     * @param  string $ignore - для директивы IGNORE (для таблиц с индексами UNIQUE) 
     * @param string $on_duplicate_key - если для таблиц с индексами UNIQUE следует отработать ON DUPLICATE KEY
     * @return int/bool
     *
     * NB! В случае использования ON DUPLICATE KEY следует позаботиться о подготовке того, что там вводится
     */

    public function createRow($table, $data, $allowable_tags = null, $ignore = '', $on_duplicate_key = '')
    {
        try {
            foreach ($data as $k => $v) { // работаем с входным массивом данных - с тем, что, собственно, мы хотим внести в запись таблицы $table

                if (is_array($v)) {
                    $v_tmp = implode(",", $v);
                    $v_tmp = trim($v_tmp);
                    unset($v);
                    $v = $v_tmp;
                }

                if ($allowable_tags != 'all' && (!empty($allowable_tags)))
                    $v = strip_tags($v, $allowable_tags);

                if (empty($allowable_tags))
                    $v = strip_tags($v);

                $data[$k] = "'" . $this->escape($v) . "'";
            }

            $cols = implode("`, `", (array_keys($data))); // получаем строку из массива полей таблицы (кстати, можно их еще дополнительно проверить)
            $vals = implode(', ', (array_values($data))); // получаем строку из массива значений для полей таблицы

            $cols = "`" . $cols . "`"; // для mysql

            // Получаем итоговую строку запроса
            $query = "INSERT {$ignore} INTO {$table} ( {$cols} ) VALUES ( {$vals} ) {$on_duplicate_key} ";


            $this->query($query); // выполняем запрос

            return $this->insert_id; // Возвращаем ID записи

        } catch (Exception_SYS $e) {
            echo "DB Exception: $e"; // или то, что мы с этим делаем
            // тут `exit;` или `return false;`
        }

    }
```
