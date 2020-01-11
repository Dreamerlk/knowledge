##一 、场景描述  
 在开发接口服务器的过程中，为了防止客户端对于接口的滥用，保护服务器的资源， 通常来说我们会对于服务器上的各种接口进行调用次数的限制。比如对于某个 用户，他在一个时间段（interval）内，比如 1 分钟，调用服务器接口的次数不能够 大于一个上限（limit），比如说 100 次。如果用户调用接口的次数超过上限的话，就直接拒绝用户的请求，返回错误信息。
       服务接口的流量控制策略：分流、降级、限流等。本文讨论下限流策略，虽然降低了服务接口的访问频率和并发量，却换取服务接口和业务应用系统的高可用。
##二、常用的限流算法
1、漏桶算法
        漏桶(Leaky Bucket)算法思路很简单,水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出(访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率.示意图如下:
　　　![](http://upload-images.jianshu.io/upload_images/6954572-41b129bdcfec8e80.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
        可见这里有两个变量,一个是桶的大小,支持流量突发增多时可以存多少的水(burst),另一个是水桶漏洞的大小(rate)。
         因为漏桶的漏出速率是固定的参数,所以,即使网络中不存在资源冲突(没有发生拥塞),漏桶算法也不能使流突发(burst)到端口速率.因此,漏桶算法对于存在突发特性的流量来说缺乏效率.
     2、令牌桶算法
                令牌桶算法(Token Bucket)和 Leaky Bucket 效果一样但方向相反的算法,更加容易理解.随着时间流逝,系统会按恒定1/QPS时间间隔(如果QPS=100,则间隔是10ms)往桶里加入Token(想象和漏洞漏水相反,有个水龙头在不断的加水),如果桶已经满了就不再加了.新请求来临时,会各自拿走一个Token,如果没有Token可拿了就阻塞或者拒绝服务.
![](http://upload-images.jianshu.io/upload_images/6954572-9e1f32efe25ca77f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
　　       令牌桶的另外一个好处是可以方便的改变速度. 一旦需要提高速率,则按需提高放入桶中的令牌的速率. 一般会定时(比如100毫秒)往桶中增加一定数量的令牌, 有些变种算法则实时的计算应该增加的令牌的数量.
##三、基于PHP+Redis实现的令牌桶算法
```
<?php
namespace Api\Lib;

/**
 * 限流控制
 */
class RateLimit
{
    private $minNum = 60; //单个用户每分访问数
    private $dayNum = 10000; //单个用户每天总的访问量

    public function minLimit($uid)
    {
        $minNumKey = $uid . '_minNum';
        $dayNumKey = $uid . '_dayNum';
        $resMin    = $this->getRedis($minNumKey, $this->minNum, 60);
        $resDay    = $this->getRedis($minNumKey, $this->minNum, 86400);
        if (!$resMin['status'] || !$resDay['status']) {
            exit($resMin['msg'] . $resDay['msg']);
        }
    }

    public function getRedis($key, $initNum, $expire)
    {
        $nowtime  = time();
        $result   = ['status' => true, 'msg' => ''];
        $redisObj = $this->di->get('redis');
        $redis->watch($key);
        $limitVal = $redis->get($key);
        if ($limitVal) {
            $limitVal = json_decode($limitVal, true);
            $newNum   = min($initNum, ($limitVal['num'] - 1) + (($initNum / $expire) * ($nowtime - $limitVal['time'])));
            if ($newNum > 0) {
                $redisVal = json_encode(['num' => $newNum, 'time' => time()]);
            } else {
                return ['status' => false, 'msg' => '当前时刻令牌消耗完！'];
            }
        } else {
            $redisVal = json_encode(['num' => $initNum, 'time' => time()]);
        }
        $redis->multi();
        $redis->set($key, $redisVal);
        $rob_result = $redis->exec();
        if (!$rob_result) {
            $result = ['status' => false, 'msg' => '访问频次过多！'];
        }
        return $result;
    }
}
```
代码要点：
1：首先定义规则
单个用户每分钟访问次数（$minNum），单个用户每天总的访问次数（$dayNum），接口总的访问次数等不同的规则。
2：计算速率
该代码示例以秒为最小的时间单位，速率=访问次数/时间（$initNum / $expire）
3：每次访问后补充的令牌个数计算方式
获取上次访问的时间即上次存入令牌的时间，计算当前时刻与上次访问的时间差乘以速率就是此次需要补充的令牌个数，注意补充令牌后总的令牌个数不能大于初始化的令牌个数，以补充数和初始化数的最小值为准。
4：程序流程
第一次访问时初始化令牌个数（$minNum），存入Redis同时将当前的时间戳存入以便计算下次需要补充的令牌个数。第二次访问时获取剩余的令牌个数，并添加本次应该补充的令牌个数，补充后如何令牌数>0则当前访问是有效的可以访问，否则令牌使用完毕不可访问。先补充令牌再判断令牌是否>0的原因是由于还有速率这个概念即如果上次剩余的令牌为0但是本次应该补充的令牌>1那么本次依然可以访问。
5：针对并发的处理
使用Redis的乐观锁机制
##四、Redis乐观锁介绍
redis对事务的支持比较简单。redis只能保证一个客户端发起的事务命令可以执行，中间不会插入其他事务。因为redis是单线程的，所以做到上面这点很容易。一般redis接受到客户端的命令后会立即执行，但是如果客户端发起multi命令，redis不会立即执行，而是让当前连接进入事务上下文，把命令放到队列中，接受到exec命令后，redis会顺序执行队列中的命令。并把执行结果打包到一起返回客户端，之后就结束了事务上下文。
一、简单的事务控制
  ![](http://upload-images.jianshu.io/upload_images/6954572-5e59163571585275?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   这个例子可以看到：两个set命令发出后并没有立即执行而是放到队列中，redis接受到exec命令才开始执行。
   如果有两个线程同时修改了一个变量的值，如何控制事务回滚？下面看乐观锁怎么控制的？
二、乐观锁控制事务
   1.什么是乐观锁？
       大多是基于数据版本的记录机制。什么是数据版本？就是为数据增加一个版本标识，即为数据库表添加一个version字段，当读取数据时，把数据库版本一同读出，当做了修改后，将数据库版本+1，同修改一起提交。如果提交数据的版本号 >数据库当前版本号，提交成功。如图：
   ![](http://upload-images.jianshu.io/upload_images/6954572-497186faad8f73f5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  2.乐观锁实例
     假设数据库中账户信息表中有一个version字段，当前值为1，账户余额为$500
    ![](http://upload-images.jianshu.io/upload_images/6954572-570dc4e8c762c116?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
     这样避免了操作员B用旧数据修改表中记录的的可能。
  3.在redis中怎么体现的？
         redis中用watch监视key，如果key在提交前被修改，则提交不成功。如下：
        ![](http://upload-images.jianshu.io/upload_images/6954572-4bebfbd90aa45e2b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
         当session1还没来得及对age进行修改，session2已经将age的值设为30，session1再执行的时候失败，因为session1对age加了乐观锁的缘故。
        watch命令会监视key，当exec时如果监视的key从调用watch后发生过变化，则整个事务会失败。也可以调用watch多次监视多个key。
三、redis事务存在的问题
        redis保证事务中的命令连续执行，但是如果其中一条命令执行失败，事务并不回滚。
       ![](http://upload-images.jianshu.io/upload_images/6954572-bb0be1eac55de9df?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
       ![](http://upload-images.jianshu.io/upload_images/6954572-941a31998ba07303?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    为age +1的命令成功，因为anme是string类型的，所以不能做加操作，命令有一个失败也不会回滚，age的值已经被修改了。
