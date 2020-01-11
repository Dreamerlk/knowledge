~~~
查询json类型中key为指定数字的数据
select * from gcV4.finance_invoice where locate('"8280":',payproid)
~~~
![图片.png](https://upload-images.jianshu.io/upload_images/6954572-99d534d01ce35e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

~~~
select * from gcV4.finance_invoice where invoiceinfo->'$.tel'  like '%电%'
或者
select * from gcV4.finance_invoice where JSON_EXTRACT(invoiceinfo, "$.tel") like '%电%'
~~~
![图片.png](https://upload-images.jianshu.io/upload_images/6954572-c7c20d93b7be3af4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
~~~
MYSQL JSON操作函数
JSON_ARRAY 生成json数组
JSON_OBJECT 生成json对象
JSON_QUOTE 加"号
JSON_CONTAINS 指定数据是否存在
JSON_CONTAINS_PATH 指定路径是否存在
JSON_EXTRACT 查找所有指定数据
JSON_KEYS 查找所有指定键值
JSON_SEARCH 查找所有指定值的位置
JSON_ARRAY_APPEND  指定位置追加数组元素
JSON_ARRAY_INSERT 指定位置插入数组元素
JSON_INSERT 指定位置插入
JSON_REPLACE 指定位置替换
JSON_SET 指定位置设置
JSON_MERGE 合并
JSON_REMOVE 指定位置移除
JSON_UNQUOTE 去"号
JSON_DEPTH 深度
JSON_LENGTH 长度
JSON_TYPE 类型
JSON_VALID 是否有效json格式
~~~
~~~
-- 增加键
UPDATE t_json SET info = json_set(info,'$.ip','192.168.1.1') WHERE id = 2;

-- 变更值
UPDATE t_json SET info = json_set(info,'$.ip','192.168.1.2') WHERE id = 2;

-- 删除键
UPDATE t_json SET info = json_remove(info,'$.ip') WHERE id = 2;

-- JSON_EXTRACT(json_doc, path[, path] ...)
-- 从json文档里抽取数据。如果有参数有NULL或path不存在，则返回NULL。如果抽取出多个path，则返回的数据封闭在一个json array里。
set @j2 = '[10, 20, [30, 40]]';
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]'); -- 20
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]', '$[0]'); -- [20, 10]
SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[2][*]'); -- [30, 40]

-- JSON_ARRAY_APPEND(json_doc, path, val[, path, val] ...)
-- 在指定path的json array尾部追加val。如果指定path是一个json object，则将其封装成一个json array再追加。如果有参数为NULL，则返回NULL。
SET @j4 = '["a", ["b", "c"], "d"]';
-- SELECT JSON_ARRAY_APPEND(@j4, '$[1][0]', 3); -- ["a", [["b", 3], "c"], "d"]
SET @j5 = '{"a": 1, "b": [2, 3], "c": 4}';
SELECT JSON_ARRAY_APPEND(@j5, '$.b', 'x'); -- {"a": 1, "b": [2, 3, "x"], "c": 4} 
SELECT JSON_ARRAY_APPEND(@j5, '$.c', 'y'); -- {"a": 1, "b": [2, 3], "c": [4, "y"]}
SELECT JSON_ARRAY_APPEND(@j5, '$', 'z'); -- [{"a": 1, "b": [2, 3], "c": 4}, "z"]

-- JSON_ARRAY_INSERT(json_doc, path, val[, path, val] ...)
-- 在path指定的json array元素插入val，原位置及以右的元素顺次右移。如果path指定的数据非json array元素，则略过此val；如果指定的元素下标超过json array的长度，则插入尾部。
SET @j6 = '["a", {"b": [1, 2]}, [3, 4]]';
SELECT JSON_ARRAY_INSERT(@j6, '$[1]', 'x'); -- ["a", "x", {"b": [1, 2]}, [3, 4]]
SELECT JSON_ARRAY_INSERT(@j6, '$[100]', 'x'); -- ["a", {"b": [1, 2]}, [3, 4], "x"]
SELECT JSON_ARRAY_INSERT(@j6, '$[1].b[0]', 'x'); -- ["a", {"b": ["x", 1, 2]}, [3, 4]]
SELECT JSON_ARRAY_INSERT(@j6, '$[0]', 'x', '$[3][1]', 'y'); -- ["x", "a", {"b": [1, 2]}, [3, "y", 4]]

-- JSON_INSERT(json_doc, path, val[, path, val] ...)
-- 在指定path下插入数据，如果path已存在，则忽略此val（不存在才插入）。
SET @j7 = '{ "a": 1, "b": [2, 3]}';
SELECT JSON_INSERT(@j7, '$.a', 10, '$.c', '[true, false]'); -- {"a": 1, "b": [2, 3], "c": "[true, false]"}

-- JSON_REPLACE(json_doc, path, val[, path, val] ...)
-- 替换指定路径的数据，如果某个路径不存在则略过（存在才替换）。如果有参数为NULL，则返回NULL。
SELECT JSON_REPLACE(@j7, '$.a', 10, '$.c', '[true, false]'); -- {"a": 10, "b": [2, 3]}

-- JSON_SET(json_doc, path, val[, path, val] ...)
-- 设置指定路径的数据（不管是否存在）。如果有参数为NULL，则返回NULL。
SELECT JSON_SET(@j7, '$.a', 10, '$.c', '[true, false]'); -- {"a": 10, "b": [2, 3], "c": "[true, false]"}

-- JSON_REMOVE(json_doc, path[, path] ...)
-- 移除指定路径的数据，如果某个路径不存在则略过此路径。如果有参数为NULL，则返回NULL。
SET @j8 = '["a", ["b", "c"], "d"]';
SELECT JSON_REMOVE(@j8, '$[1]'); -- ["a", "d"]

-- JSON_MERGE(json_doc, json_doc[, json_doc] ...)
-- merge多个json文档。规则如下：
-- 如果都是json array，则结果自动merge为一个json array；
-- 如果都是json object，则结果自动merge为一个json object；
-- 如果有多种类型，则将非json array的元素封装成json array再按照规则一进行mege。
SELECT JSON_MERGE('[1, 2]', '[true, false]'); -- [1, 2, true, false]
SELECT JSON_MERGE('{"name": "x"}', '{"id": 47}'); -- {"id": 47, "name": "x"}
SELECT JSON_MERGE('1', 'true'); -- [1, true]
SELECT JSON_MERGE('[1, 2]', '{"id": 47}'); -- [1, 2, {"id": 47}]

-- JSON_REMOVE(json_doc, path[, path] ...)
-- 移除指定路径的数据，如果某个路径不存在则略过此路径。如果有参数为NULL，则返回NULL。
SET @j8 = '["a", ["b", "c"], "d"]';
SELECT JSON_REMOVE(@j8, '$[1]'); -- ["a", "d"]


~~~
