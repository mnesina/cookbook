<a href="README.md">Оглавление</a>

# Для класса работы с базами данных - добавление записи к таблице


```PHP
    /**
     * Добавление записи к таблице
     *
     * @param string $table
     * @param array $data
     * @param null/string $allowable_tags
     * @param  string $ignore
     * @param string $on_duplicate_key
     * @return mixed/bool
     */

    public function createRow($table, $data, $allowable_tags = null, $ignore = '', $on_duplicate_key = '',$do_transaction_isolation=null, $do_transaction_isolation_old='REPEATABLE READ')
    {
        try {
            foreach ($data as $k => $v) {

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

            $cols = implode("`, `", (array_keys($data)));
            $vals = implode(', ', (array_values($data)));

            $cols = "`" . $cols . "`";

            $query = "INSERT {$ignore} INTO " . $table . " (" . $cols . ") VALUES (" . $vals . ") {$on_duplicate_key} ";


            $this->query($query);

            return $this->insert_id; 

        } catch (Exception_SYS $e) {
            echo "DB Exception: $e";
            // тут `exit;` или `return false;`
        }

    }
```
