**PHP扩展安装** 

1\. 环境要求：PHP_VERSION >= 5.3.9，composer工具

2\. 在E盘新建文件夹命名为elastic,，拷贝composer.phar到
     E:/elastic目录下面

3\. 打开命令行窗口，进入E:/elastic

4\. 在命令行运行：
      php composer.phar require elasticsearch/elasticsearch

5\. 此时E:/elastic目录下会出现一个vendor目录，安装成功

6\. 使用方法：
       require 'vendor/autoload.php'; 
      $client = new Elasticsearch\Client();

**创建索引**

1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’;  //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $data[‘body’][‘settings’][‘number_of_shards’] = 5;  //主分片数量  
6.  $data[‘body’][‘settings’][‘number_of_replicas’] = 0; //从分片数量  
7.  $elastic->indices()->create($index);  

**插入索引数据**

1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘id’] = 1   //不指定id，系统会自动生成唯一id  
6.  $index[‘body’] = array(  
7.  ‘mac’ => 'fcd5d900beca',  
8.  ‘customer_id’ => 3,  
9.  ‘product_id’ => 5,  
10.  ‘version’ => 2  
11.  );  
12.  $elastic->index($index);  

**查询**

1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘body’][‘query’][‘match’][‘mac’] = ‘fcd5d900beca’;  
6.  $index[‘size’] = 10;  
7.  $index[‘from’] = 200;  
8.  $elastic->search($index);  

  相当于sql语句：  
 select*from ems_run_log where mac=‘fcd5d900beca’  limit 200,10;

1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘body’][‘query’][‘bool’][‘must’] = array(  
6.  array(‘match’ => array(‘mac’ => ‘fcd5d900beca’)),  
7.  array(‘match’ => array(‘product_id’ => 20))  
8.  );  
9.  $index[‘size’] = 10;  
10.  $index[‘from’] = 200;  
11.  $elastic->search($index);  

  相当于sql语句：  
  select*from ems_run_log where mac=‘fcd5d900beca’   and product_id = 20 limit 200,10;  


1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘body’][‘query’][‘bool’][‘should’] = array(  
6.  array(‘match’ => array(‘mac’ => ‘fcd5d900beca’)),  
7.  array(‘match’ => array(‘product_id’ => 20))  
8.  );  
9.  $index[‘size’] = 10;  
10.  $index[‘from’] = 200;  
11.  $elastic->search($index);  

  当于sql语句：  
  select*from ems_run_log where mac=‘fcd5d900beca’   or product_id = 20 limit 200,10;  


1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘body’][‘query’][‘bool’][‘must_not’] = array(  
6.  array(‘match’ => array(‘mac’ => ‘fcd5d900beca’)),  
7.  array(‘match’ => array(‘product_id’ => 20))  
8.  );  
9.  $index[‘size’] = 10;  
10.  $index[‘from’] = 200;  
11.  $elastic->search($index);  

 相当于sql语句：  
 select*from ems_run_log where mac!=‘fcd5d900beca’  and product_id != 20 limit 200,10;  


1.  include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index[‘index’] = ‘log’; //索引名称  
4.  $index[‘type’] = ‘ems_run_log’; //类型名称  
5.  $index[‘body’][‘query’][‘range’] = array(  
6.  ‘id’ => array(‘gte’ => 20,’lt’ => 30);  
7.  );  
8.  $index[‘size’] = 10;  
9.  $index[‘from’] = 200;  
10.  $elastic->search($index);  

相当于sql语句：  
 select*from ems_run_log where id>=20  and id<30  limit 200,10;  

**删除文档**

1.  <pre name="code" class="php">include('./vendor/autoload.php');  
2.  $elastic = new Elasticsearch\Client();  
3.  $index['index'] = 'test';  //索引名称  
4.  $index['type'] = 'ems_test'; //类型名称  
5.  $index['id'] = 2;   
6.  $elastic->delete($index);
