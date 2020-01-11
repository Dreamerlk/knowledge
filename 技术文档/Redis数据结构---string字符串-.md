**string类型介绍**
与大多编程语言中的字符串类型一样，Redis的字符串也是字符序列，它是Redis中最为基础的数据存储类型，具有以下特点：
字符串类型是Redis中二进制安全的，这就意味着它们都有一个已知的长度，可以接受任何格式的数据（如信息交换中常用的Json格式字符串，甚至图像数据）。
Redis中字符串类型最多可以容纳的数据长度可达512M。

**string类型相关命令**
Redis字符串命令主要用于管理字符串值，主要包括以下命令：
**1、 set命令**
set命令用于设置指定键的值，具体格式为：
set key value [ex 秒数] [px 毫秒数] [nx/xx]　　

各个选项的含义如下：
ex：设置指定的到期时间，单位为秒
px：设置指定的到期时间，单位为毫秒，如果ex和px同时写，则以后面的有效期为准
nx：如果对应key不存在则创建
xx：如果对应key存在则修改其值

示例1：
 ```
127.0.0.1:6379> set mykey "this is redis"
OK
```

在示例1中，用set命令来设置key、value，操作成功后终端打印出“OK”。
**2、get命令**
get命令用来获取指定键的值，如果键不存在，则返回nil，如果返回值不是字符串，则返回错误。具体格式为：
get key

示例2：
```
127.0.0.1:6379> get mykey"this is redis"
127.0.0.1:6379> get yourkey(nil)
```

**3、mset命令**
mset命令用于一次性设置多个键和值，和set命令一样操作成功后返回字符串“OK”。具体格式为：
mset key1 value1 key2 value2 ...

示例3：
```
127.0.0.1:6379> mset key1 "this is key1" key2 "this is key2"
OK
127.0.0.1:6379> get key1
"this is key1"
127.0.0.1:6379> get key2
"this is key2"
```

**4、mget命令**
mget命令用于返回所有给定键的值。对于某个不存在值的键或者不存在的键，返回nil，否则返回指定键的值列表。具体格式为：
mget key1 key2 ...

示例4：
```
127.0.0.1:6379> set key1 "hello"
OK
127.0.0.1:6379> set key2 "world"
OK
127.0.0.1:6379> mget key1 key2 key3
1) "hello"
2) "world"
3) (nil)
```


**5、setrange命令**
setrange命令将字符串中偏移量为offset后的子串覆盖为指定的值，该命令返回修改后的字符串的长度。具体格式为：
setrange key offset value

如果偏移量offset > 原字符串长度，不足部分用0x00补全。
示例5：
```
127.0.0.1:6379> set key1 "Hello World"
OK
127.0.0.1:6379> setrange key1 6 "Redis"
(integer) 11
127.0.0.1:6379> get key1
"Hello Redis"
127.0.0.1:6379> setrange key1 15 "Hei"
(integer) 18
127.0.0.1:6379> get key1
"Hello Redis\x00\x00\x00\x00Hei"
```


**6、setex命令**
setex命令用来设置指定键的值，并指定该键值对应的存在时间（单位：秒）。具体格式如下：
setex key seconds value

示例6：
```
127.0.0.1:6379> setex key1 5 "hello"  // 5s的过期时间
OK
127.0.0.1:6379> get key1    // 马上访问
"hello"
127.0.0.1:6379> get key1    // 5s后访问
(nil)
```

7、setnx命令
setnx命令也可以用来设置指定键的字符串值，但该命令在设置前需要检查指定键是否已经存在。如果存在，则该命令的作用和set命令一样，操作完成后返回1，否则不重新设置已经存在的键的字符串值，直接返回0。具体格式如下：
setnx key value

示例7：
```
127.0.0.1:6379> setnx key1 "Hello"
(integer) 1
127.0.0.1:6379> setnx key1 "World"
(integer) 0
127.0.0.1:6379> get key1
"Hello"
```

**8、append命令**
故名思议，append命令将字符串追加到指定键的原值上，返回值为新字符串的长度。具体格式为：
append key value 

示例8：
```
127.0.0.1:6379> set key1 "Hello "
OK
127.0.0.1:6379> append key1 "World"
(integer) 11
127.0.0.1:6379> get key1
"Hello World"
```

**9、getrange命令**
getrange命令获取字符串指定范围的子串，具体格式为：
getrange key start stop

getrange命令返回字符串中下标范围为[start，stop]范围的值。类似python，该命令下标支持负偏移量，右边第一个下标为-1。假设字符串的长度为length，getrange根据以下原则决定返回值：
当start > length，则返回空字符串
当stop >= length，则截取至字符串尾
如果start > stop，则返回空字符串
如果0 <= start <= stop < length，返回指定范围的子串

示例9：
```
127.0.0.1:6379> set key1 "This is getrange testing"
OK
127.0.0.1:6379> getrange key1 5 6
"is"
127.0.0.1:6379> getrange key1 100 200
""
127.0.0.1:6379> getrange key1 5 100
"is getrange testing"
```

**10、incr命令**
incr命令用于自增一个指定键对应的整数值并返回新的值。如果该键不存在，则创建该键，对应的value被置为0然后执行自增操作，如果该键对应的值不能转换为整数，则返回错误。
具体格式为：
incr key

示例8：
```
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> set key1 10
OK
127.0.0.1:6379> incr key1
(integer) 11
127.0.0.1:6379> incr key2
(integer) 1
127.0.0.1:6379> set key2 "non"
OK
127.0.0.1:6379> incr key2
(error) ERR value is not an integer or out of range
127.0.0.1:6379>
```

**10、incrby命令**
该命令与incr命令相似，不同的是：incrby命令可以自定义自增值，这也是命令中“by”的含义。具体格式为：
incrby key k

示例9：
```
127.0.0.1:6379> set key1 10
OK
127.0.0.1:6379> incrby key1 5
(integer) 15
127.0.0.1:6379> get key1
"15"
```

**11、incrbyfloat命令**
从字面上我们就可以看出incrbyfloat是和incrby相似的命令，不同的是：incrbyfloat对指定键的值自增一个浮点数。该命令返回修改后的新值。具体格式为：
incrbyfloat by f

示例10：
```
127.0.0.1:6379> set key1 100
OK
127.0.0.1:6379> incrbyfloat key1 0.5
"100.5"
```

**12、decr和decrby命令**
decr命令和incr命令作用相反，具体格式如下：
decr key

decrby命令和incrby命令作用相反，具体格式如下：
decrby key decrement

**13、strlen命令**
strlen命令返回指定键的字符串值的长度。具体格式为：
strlen key

示例13：
```
127.0.0.1:6379> set key1 "Hello Redis"
OK
127.0.0.1:6379> strlen key1
(integer) 11
```

**14、setbit命令**
setbit命令用来设置指定键的字符串在offset偏移量上对应二进制位上的值，并返回该为上的旧值。由于该命令操作的是二进制位，所以设置的新值只能为0或1。如果指定key不存在，则创建一个新值并在指定的offset上设置二进制值。如果offset大于字符串的长度，不足部分用0填充后在指定offset上设置二进制值。具体格式如下：
setbit key offset value

示例14：
```
127.0.0.1:6379> set key1 "Hello Redis"
OK
127.0.0.1:6379> strlen key1
(integer) 11
```

15、getbit命令
与setbit命令相对应，getbit命令用于返回指定偏移量offset上二进制位的值。如果offset大于value的长度，或者指定key不存在，则返回0。具体格式如下：
getbit key offset

示例15：
```
127.0.0.1:6379> setbit key1 7 1
(integer) 1
127.0.0.1:6379> getbit key1 7
(integer) 1
127.0.0.1:6379> getbit key1 100
(integer) 0
```
