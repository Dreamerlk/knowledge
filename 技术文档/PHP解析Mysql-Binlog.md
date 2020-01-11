PHP解析Mysql Binlog，依赖于mysql-replication-listener库
详见：[https://github.com/bullsoft/php-binlog](https://github.com/bullsoft/php-binlog)

## Install MySQL Replication Listener

*   [https://github.com/bullsoft/mysql-replication-listener/archive/master.zip](https://github.com/bullsoft/mysql-replication-listener/archive/master.zip)

*   该源代码，有一处bug，在 tcp_driver.cpp 第 650 行处：

```
int Binlog_tcp_driver::set_position(const std::string &str, unsigned long position)
{
  /*
    Validate the new position before we attempt to set. Once we set the
    position we won't know if it succeded because the binlog dump is
    running in another thread asynchronously.
  */

  /*
    // 这个地方会导致，假设 set_position 不是最后一个 binlog file，并且 position 又大于最后一个 binlog size，则会返回失败，特此屏蔽掉该推断
    if(position >= m_binlog_offset) {
      return ERR_FAIL;
    }
  */
```

*   改动后的 mysql-replication-listener 源代码和 php-binlog 源代码打包下载地址：
    [http://download.csdn.net/download/xtjsxtj/9843275](http://download.csdn.net/download/xtjsxtj/9843275)

```
unzip mysql-replication-listener-master.zip
cd mysql-replication-listener-master
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql-replication
make & make install
```

## Install php-binlog

*   [https://github.com/bullsoft/php-binlog/archive/master.zip](https://github.com/bullsoft/php-binlog/archive/master.zip)

```
unzip php-binlog-master.zip
cd php-binlog-master/ext
/usr/local/php5.5.15/bin/phpize
./configure --with-php-config=/usr/local/php5.5.15/bin/php-config --with-mysql-binlog=/usr/local/mysql-replication
```

## Examples

注：Binlog为行格式

```
<?php

$link = binlog_connect("mysql://root:cpyf@127.0.0.1:3306");
//binlog_set_position($link, 4);  
//binlog_set_position($link, 4, 'mysql-bin.000006');                           

while($event=binlog_wait_for_next_event($link)) {
    // it will block here                                 
    switch($event['type_code']) {
        case BINLOG_DELETE_ROWS_EVENT:
            var_dump($event);
            // do what u want ...                           
            break;
        case BINLOG_WRITE_ROWS_EVENT:
            var_dump($event);
            // do what u want ...                           
            break;
        case BINLOG_UPDATE_ROWS_EVENT:
            var_dump($event);
            // do what u want ...                           
            break;
        default:
            // var_dump($event);                            
            break;
    }
}

```

### Update_rows

```
update `type` set type_id = 22 WHERE id in (58, 59);
```

```
array(5) {
  'type_code' =>
  int(24)
  'type_str' =>
  string(11) "Update_rows"
  'db_name' =>
  string(5) "cloud"
  'table_name' =>
  string(4) "type"
  'rows' =>
  array(4) {
    [0] =>
    array(5) {
      [0] =>
      string(2) "58"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(2) "22"
      [4] =>
      string(1) "0"
    }
    [1] =>
    array(5) {
      [0] =>
      string(2) "58"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(1) "4"
      [4] =>
      string(1) "0"
    }
    [2] =>
    array(5) {
      [0] =>
      string(2) "59"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(2) "22"
      [4] =>
      string(1) "0"
    }
    [3] =>
    array(5) {
      [0] =>
      string(2) "59"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(1) "4"
      [4] =>
      string(1) "0"
    }
  }
}
```

### Delete_rows

```
delete from `type` WHERE id in (58, 59);
```

```
array(5) {
  'type_code' =>
  int(25)
  'type_str' =>
  string(11) "Delete_rows"
  'db_name' =>
  string(5) "cloud"
  'table_name' =>
  string(4) "type"
  'rows' =>
  array(2) {
    [0] =>
    array(5) {
      [0] =>
      string(2) "58"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(2) "22"
      [4] =>
      string(1) "0"
    }
    [1] =>
    array(5) {
      [0] =>
      string(2) "59"
      [1] =>
      string(8) "adsfasdf"
      [2] =>
      string(4) "asdf"
      [3] =>
      string(2) "22"
      [4] =>
      string(1) "0"
    }
  }
}
```

### Write_rows

```
insert into type values (Null, "Hello, World", "Best world", 4, 0), (NULL, "你好，世界", "世界非常美好", 3, 5);
```

```
array(5) {
  'type_code' =>
  int(23)
  'type_str' =>
  string(10) "Write_rows"
  'db_name' =>
  string(5) "cloud"
  'table_name' =>
  string(4) "type"
  'rows' =>
  array(2) {
    [0] =>
    array(5) {
      [0] =>
      string(2) "95"
      [1] =>
      string(12) "Hello, World"
      [2] =>
      string(10) "Best world"
      [3] =>
      string(1) "4"
      [4] =>
      string(1) "0"
    }
    [1] =>
    array(5) {
      [0] =>
      string(2) "96"
      [1] =>
      string(15) "你好。世界"
      [2] =>
      string(15) "世界非常美好"
      [3] =>
      string(1) "3"
      [4] =>
      string(1) "5"
    }
  }
}
```
