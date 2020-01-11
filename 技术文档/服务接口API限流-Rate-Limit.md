一、场景描述                                                                                                
     很多做服务接口的人或多或少的遇到这样的场景，由于业务应用系统的负载能力有限，为了防止非预期的请求对系统压力过大而拖垮业务应用系统。
    也就是面对大流量时，如何进行流量控制？
    服务接口的流量控制策略：分流、降级、限流等。本文讨论下限流策略，虽然降低了服务接口的访问频率和并发量，却换取服务接口和业务应用系统的高可用。
     实际场景中常用的限流策略：
Nginx前端限流

         按照一定的规则如帐号、IP、系统调用逻辑等在Nginx层面做限流
业务应用系统限流

        1、客户端限流
        2、服务端限流
数据库限流

        红线区，力保数据库
二、常用的限流算法                                                                                       
     常用的限流算法由：楼桶算法和令牌桶算法。本文不具体的详细说明两种算法的原理，原理会在接下来的文章中做说明。
     1、漏桶算法
         漏桶(Leaky Bucket)算法思路很简单,水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出(访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率.示意图如下:
　　　![](http://upload-images.jianshu.io/upload_images/6954572-fb9e39784b70f9f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
         可见这里有两个变量,一个是桶的大小,支持流量突发增多时可以存多少的水(burst),另一个是水桶漏洞的大小(rate)。
         因为漏桶的漏出速率是固定的参数,所以,即使网络中不存在资源冲突(没有发生拥塞),漏桶算法也不能使流突发(burst)到端口速率.因此,漏桶算法对于存在突发特性的流量来说缺乏效率.
     2、令牌桶算法
         令牌桶算法(Token Bucket)和 Leaky Bucket 效果一样但方向相反的算法,更加容易理解.随着时间流逝,系统会按恒定1/QPS时间间隔(如果QPS=100,则间隔是10ms)往桶里加入Token(想象和漏洞漏水相反,有个水龙头在不断的加水),如果桶已经满了就不再加了.新请求来临时,会各自拿走一个Token,如果没有Token可拿了就阻塞或者拒绝服务.
![](http://upload-images.jianshu.io/upload_images/6954572-e270efc4f6e59366.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
　　令牌桶的另外一个好处是可以方便的改变速度. 一旦需要提高速率,则按需提高放入桶中的令牌的速率. 一般会定时(比如100毫秒)往桶中增加一定数量的令牌, 有些变种算法则实时的计算应该增加的令牌的数量.
三、基于Redis功能的实现                                                                                
       简陋的设计思路：假设一个用户（用IP判断）每分钟访问某一个服务接口的次数不能超过10次，那么我们可以在Redis中创建一个键，并此时我们就设置键的过期时间为60秒，每一个用户对此服务接口的访问就把键值加1，在60秒内当键值增加到10的时候，就禁止访问服务接口。在某种场景中添加访问时间间隔还是很有必要的。
      1）使用Redis的incr命令，将计数器作为Lua脚本         
```
 local current
 current = redis.call("incr",KEYS[1])
 if tonumber(current) == 1 then
 redis.call("expire",KEYS[1],1)
 end
```
 Lua脚本在Redis中运行，保证了incr和expire两个操作的原子性。
       2）使用Reids的列表结构代替incr命令
```
FUNCTION LIMIT_API_CALL(ip)
current = LLEN(ip)
IF current > 10 THEN
    ERROR "too many requests per second"
ELSE
    IF EXISTS(ip) == FALSE
        MULTI
            RPUSH(ip,ip)
            EXPIRE(ip,1)
        EXEC
    ELSE
        RPUSHX(ip,ip)
    END
    PERFORM_API_CALL()
END
```

 Rate Limit使用Redis的列表作为容器，LLEN用于对访问次数的检查，一个事物中包含了RPUSH和EXPIRE两个命令，用于在第一次执行计数是创建列表并设置过期时间，
    RPUSHX在后续的计数操作中进行增加操作。

四、基于令牌桶算法的实现                                                                                
       令牌桶算法可以很好的支撑突然额流量的变化即满令牌桶数的峰值。
import java.io.BufferedWriter;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStreamWriter;
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;
 
import com.google.common.base.Preconditions;
import com.netease.datastream.util.framework.LifeCycle;
 
 20 public class TokenBucket implements LifeCycle {
 
// 默认桶大小个数 即最大瞬间流量是64M
 private static final int DEFAULT_BUCKET_SIZE = 1024 * 1024 * 64;
 
// 一个桶的单位是1字节
 private int everyTokenSize = 1;
 
// 瞬间最大流量
 private int maxFlowRate;
 
// 平均流量
 private int avgFlowRate;
 
// 队列来缓存桶数量：最大的流量峰值就是 = everyTokenSize*DEFAULT_BUCKET_SIZE 64M = 1 * 1024 * 1024 * 64
 private ArrayBlockingQueue<Byte> tokenQueue = new ArrayBlockingQueue<Byte>(DEFAULT_BUCKET_SIZE);
 
private ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
 
private volatile boolean isStart = false;
 
private ReentrantLock lock = new ReentrantLock(true);
 
private static final byte A_CHAR = 'a';
 
public TokenBucket() {
 }
 
public TokenBucket(int maxFlowRate, int avgFlowRate) {
 this.maxFlowRate = maxFlowRate;
 this.avgFlowRate = avgFlowRate;
 }
 
public TokenBucket(int everyTokenSize, int maxFlowRate, int avgFlowRate) {
 this.everyTokenSize = everyTokenSize;
 this.maxFlowRate = maxFlowRate;
 this.avgFlowRate = avgFlowRate;
 }
 
public void addTokens(Integer tokenNum) {
 
// 若是桶已经满了，就不再家如新的令牌
 for (int i = 0; i < tokenNum; i++) {
 tokenQueue.offer(Byte.valueOf(A_CHAR));
 }
 }
 
public TokenBucket build() {
 
start();
 return this;
 }
 
/**
 * 获取足够的令牌个数
 *
 * @return
 */
 public boolean getTokens(byte[] dataSize) {
 
Preconditions.checkNotNull(dataSize);
 Preconditions.checkArgument(isStart, "please invoke start method first !");
 
int needTokenNum = dataSize.length / everyTokenSize + 1;// 传输内容大小对应的桶个数
 
final ReentrantLock lock = this.lock;
 lock.lock();
 try {
 boolean result = needTokenNum <= tokenQueue.size(); // 是否存在足够的桶数量
 if (!result) {
 return false;
 }
 
int tokenCount = 0;
 for (int i = 0; i < needTokenNum; i++) {
 Byte poll = tokenQueue.poll();
 if (poll != null) {
 tokenCount++;
 }
 }
 
return tokenCount == needTokenNum;
 } finally {
 lock.unlock();
 }
 }
 
@Override
 public void start() {
 
// 初始化桶队列大小
 if (maxFlowRate != 0) {
 tokenQueue = new ArrayBlockingQueue<Byte>(maxFlowRate);
 }
 
// 初始化令牌生产者
 TokenProducer tokenProducer = new TokenProducer(avgFlowRate, this);
 scheduledExecutorService.scheduleAtFixedRate(tokenProducer, 0, 1, TimeUnit.SECONDS);
 isStart = true;
 
}
 
@Override
 public void stop() {
 isStart = false;
 scheduledExecutorService.shutdown();
 }
 
@Override
 public boolean isStarted() {
 return isStart;
 }
 
class TokenProducer implements Runnable {
 
private int avgFlowRate;
 private TokenBucket tokenBucket;
 
public TokenProducer(int avgFlowRate, TokenBucket tokenBucket) {
 this.avgFlowRate = avgFlowRate;
 this.tokenBucket = tokenBucket;
 }
 
@Override
 public void run() {
 tokenBucket.addTokens(avgFlowRate);
 }
 }
 
public static TokenBucket newBuilder() {
 return new TokenBucket();
 }
 
public TokenBucket everyTokenSize(int everyTokenSize) {
 this.everyTokenSize = everyTokenSize;
 return this;
 }
 
public TokenBucket maxFlowRate(int maxFlowRate) {
 this.maxFlowRate = maxFlowRate;
 return this;
 }
 
public TokenBucket avgFlowRate(int avgFlowRate) {
 this.avgFlowRate = avgFlowRate;
 return this;
 }
 
private String stringCopy(String data, int copyNum) {
 
StringBuilder sbuilder = new StringBuilder(data.length() * copyNum);
 
for (int i = 0; i < copyNum; i++) {
 sbuilder.append(data);
 }
 
return sbuilder.toString();
 
}
 
public static void main(String[] args) throws IOException, InterruptedException {
 
tokenTest();
 }
 
private static void arrayTest() {
 ArrayBlockingQueue<Integer> tokenQueue = new ArrayBlockingQueue<Integer>(10);
 tokenQueue.offer(1);
 tokenQueue.offer(1);
 tokenQueue.offer(1);
 System.out.println(tokenQueue.size());
 System.out.println(tokenQueue.remainingCapacity());
 }
 
private static void tokenTest() throws InterruptedException, IOException {
 TokenBucket tokenBucket = TokenBucket.newBuilder().avgFlowRate(512).maxFlowRate(1024).build();
 
BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(new FileOutputStream("/tmp/ds_test")));
 String data = "xxxx";// 四个字节
 for (int i = 1; i <= 1000; i++) {
 
Random random = new Random();
 int i1 = random.nextInt(100);
 boolean tokens = tokenBucket.getTokens(tokenBucket.stringCopy(data, i1).getBytes());
 TimeUnit.MILLISECONDS.sleep(100);
 if (tokens) {
 bufferedWriter.write("token pass --- index:" + i1);
 System.out.println("token pass --- index:" + i1);
 } else {
 bufferedWriter.write("token rejuect --- index" + i1);
 System.out.println("token rejuect --- index" + i1);
 }
 
bufferedWriter.newLine();
 bufferedWriter.flush();
 }
 
bufferedWriter.close();
 }
 
}
