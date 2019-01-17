<a href="README.md">Оглавление</a>

# Псевдоадаптив

Мы вводили мобильную версию не сразу, а потом все перевели на псевдоадаптивный вариант, т.е. и мобильная, и десктопная версия работают по одним и тем же URL. 
В результате в начале работы разбираемся с тем, куда направить пользователя

// Это кусок для перенаправления мобильных устройств на мобильную версию - начало
```PHP
if(DO_MOBILE) {
    $test_is_mobile = get_dfirst_index_php();

    if(DO_ADAPTIVE) {
        $cookie_time = 30 * 60 * 60 * 24;
        switch ($test_is_mobile) {
            case "m": // требуется мобильная версия
                $old_http_host = substr($_SERVER['HTTP_HOST'], 2);

                $redirect_url = $_SERVER['REQUEST_SCHEME']."://".$old_http_host.$_SERVER['REQUEST_URI']; 

                // В этом случае нам надо переадресовать на обычную страницу, но поставить куку в знак мобильности
                if(isset($_COOKIE['mob2pc'])) {
                    setcookie("mob2pc", "", time() - 3600, "/", SITE_DOMEIN);
                }

                setcookie("pc2mob", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                //redirect($redirect_url,true);
                header('HTTP/1.1 301 Moved Permanently');
                header("Location: $redirect_url");
                break;
            default: // url -  обычная версия
                if(isset($_COOKIE['pc2mob']) || isset($_COOKIE['mob2pc']) ||  isset($_COOKIE['mobIsdetected'])) {
                    // мы тут уже были
                    if(isset($_COOKIE['pc2mob'])) { // PC-->мобильная
                        setcookie("mob2pc", "", time() - 3600, "/", SITE_DOMEIN);
                        //setcookie("mobIsdetected", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                        define("IS_MOBILE", 1);
                    } elseif(isset($_COOKIE['mob2pc'])) { // мобильная-->PC
                        setcookie("pc2mob", "", time() - 3600, "/", SITE_DOMEIN);
                        //setcookie("mobIsdetected", "", time() - 3600, "/", SITE_DOMEIN);
                        define("IS_MOBILE", 0);
                    } elseif(isset($_COOKIE['mobIsdetected'])) { // мобильная
                        setcookie("mob2pc", "", time() - 3600, "/", SITE_DOMEIN);
                        setcookie("mobIsdetected", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                        define("IS_MOBILE", 1);
                    }
                } else {
                    // мы тут первый раз
                    require_once(PATH_DIFFER.'mobiledetect/Mobile_Detect.php'); // подключаем библиотеку http://mobiledetect.net/
                    $detect = new Mobile_Detect;
                    if( $detect->isMobile() && !$detect->isTablet()  ){ // мобильники без планшетов (ну, или чтио-то меняем)
                        setcookie("mobIsdetected", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                        setcookie("pc2mob", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                        define("IS_MOBILE", 1);
                    } else {
                        //$cookie_time = 30 * 60 * 60 * 24;
                        setcookie("mob2pc", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                        define("IS_MOBILE", 0);
                    }
                }
                // конец мукам выбора
                break;
        }
    } else {
        switch ($test_is_mobile) {

            case "m": // требуется мобильная версия
                $old_http_host = substr($_SERVER['HTTP_HOST'], 2);

                // 1) направление: десктоп --> мобильная
                if(isset($_SERVER['HTTP_REFERER']) && $_SERVER['HTTP_REFERER']==$_SERVER['REQUEST_SCHEME']."://".$old_http_host.$_SERVER['REQUEST_URI']) {
                    // разбираемся с куками
                    if(isset($_COOKIE['mob2pc'])) {
                        setcookie("mob2pc", "", time() - 3600, "/", SITE_DOMEIN);
                    }
                    $cookie_time = 30 * 60 * 60 * 24;
                    setcookie("pc2mob", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                }

                break;
            default: // требуется обычная версия
                $mobile_url = $_SERVER['REQUEST_SCHEME']."://m.".$_SERVER['HTTP_HOST'].$_SERVER['REQUEST_URI'];
                // 1) направление: мобильная - десктоп
                if(isset($_SERVER['HTTP_REFERER']) && $_SERVER['HTTP_REFERER']==$mobile_url) {
                    // разбираемся с куками
                    if(isset($_COOKIE['pc2mob'])) {
                        setcookie("pc2mob", "", time() - 3600, "/", SITE_DOMEIN);
                        setcookie("mobIsdetected", "", time() - 3600, "/", SITE_DOMEIN);
                    }

                    $cookie_time = 30 * 60 * 60 * 24;
                    setcookie("mob2pc", 1, time() + $cookie_time, "/", SITE_DOMEIN);

                } else { // просто пришли - значит, проверяем с какого устройства и есть ли куки
                    if(!isset($_COOKIE['mob2pc']) && !isset($_COOKIE['mobIsdetected'])) { // Только если не стоит кука выбора, не трогаем

                        require_once(PATH_DIFFER.'mobiledetect/Mobile_Detect.php'); // подключаем библиотеку http://mobiledetect.net/
                        $detect = new Mobile_Detect;
                        if( $detect->isMobile() && !$detect->isTablet() ) { // мобильники без планшетов (ну, или чтио-то меняем)
                            $cookie_time = 30 * 60 * 60 * 24;
                            setcookie("mobIsdetected", 1, time() + $cookie_time, "/", SITE_DOMEIN);
                            header("Location: $mobile_url");
                        }

                    } elseif(!isset($_COOKIE['mob2pc']) && isset($_COOKIE['mobIsdetected'])) {
                        // Нет куки выбора, но есть уже имеющаяся кука-признак того, что устройство было определено как мобильное
                        header("Location: $mobile_url");
                    }
                }

                break;

        }
    }
}
// --Это кусок для перенаправления мобильных устройств на мобильную версию - конец

function get_dfirst_index_php()
{
    $http_host = $_SERVER['HTTP_HOST'];
    $http_host_array = explode(".", $http_host);
    $res = $http_host_array[0];

    return $res;
}

// SITE_DOMEIN - определено в settings.php как, например, 
define("SITE_DOMEIN", "test.com");		/* Домен */ 
```
