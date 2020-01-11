在实际工作中，使用PHP写api接口是经常做的，PHP写好接口后，前台就可以通过链接获取接口提供的数据，而返回的数据一般分为两种情况，xml和json,在这个过程中，服务器并不知道，请求的来源是什么，有可能是别人非法调用我们的接口，获取数据，因此就要使用安全验证。
**验证原理**
**示意图**
![这里写图片描述](http://upload-images.jianshu.io/upload_images/6954572-21237d381c4320b4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**原理**
从图中可以看得很清楚，前台想要调用接口，需要使用几个参数生成签名。
时间戳：当前时间
随机数：随机生成的随机数
口令：前后台开发时，一个双方都知道的标识，相当于暗号
算法规则：商定好的运算规则，上面三个参数可以利用算法规则生成一个签名。

前台生成一个签名，当需要访问接口的时候，把时间戳，随机数，签名通过URL传递到后台。后台拿到时间戳，随机数后，通过一样的算法规则计算出签名，然后和传递过来的签名进行对比，一样的话，返回数据。
**算法规则**
在前后台交互中，算法规则是非常重要的，前后台都要通过算法规则计算出签名，至于规则怎么制定，看你怎么高兴怎么来。
我这个算法规则是
时间戳，随机数，口令按照首字母大小写顺序排序
然后拼接成字符串
进行sha1加密
再进行MD5加密
转换成大写。

**前台**
这里我并没有实际的前台，直接使用一个PHP文件代替前台，然后通过CURL模拟GET请求。我使用的是TP框架，URL格式是pathinfo格式。
**源代码**
```
<?php
/**
 * Created by PhpStorm.
 * User: Administrator
 * Date: 2017/3/16 0016
 * Time: 15:56
 */
namespace Client\Controller;
use Think\Controller;

class ClientController extends Controller{
    const TOKEN = 'API';
    //模拟前台请求服务器api接口
    public function getDataFromServer(){
        //时间戳
        $timeStamp = time();
        //随机数
        $randomStr = $this -> createNonceStr();
        //生成签名
        $signature = $this -> arithmetic($timeStamp,$randomStr);
        //url地址
        $url = "http://www.apitest.com/Server/Server/respond/t/{$timeStamp}/r/{$randomStr}/s/{$signature}";
        $result = $this -> httpGet($url);
        dump($result);
    }

    //curl模拟get请求。
    private function httpGet($url){
        $curl = curl_init();

        //需要请求的是哪个地址
        curl_setopt($curl,CURLOPT_URL,$url);
        //表示把请求的数据已文件流的方式输出到变量中
        curl_setopt($curl,CURLOPT_RETURNTRANSFER,1);

        $result = curl_exec($curl);
        curl_close($curl);
        return $result;
    }

    //随机生成字符串
    private function createNonceStr($length = 8) {
        $chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        $str = "";
        for ($i = 0; $i < $length; $i++) {
            $str .= substr($chars, mt_rand(0, strlen($chars) - 1), 1);
        }
        return "z".$str;
    }

    /**
     * @param $timeStamp 时间戳
     * @param $randomStr 随机字符串
     * @return string 返回签名
     */
    private function arithmetic($timeStamp,$randomStr){
        $arr['timeStamp'] = $timeStamp;
        $arr['randomStr'] = $randomStr;
        $arr['token'] = self::TOKEN;
        //按照首字母大小写顺序排序
        sort($arr,SORT_STRING);
        //拼接成字符串
        $str = implode($arr);
        //进行加密
        $signature = sha1($str);
        $signature = md5($signature);
        //转换成大写
        $signature = strtoupper($signature);
        return $signature;
    }
}
```

**服务器端**
接受前台数据进行验证
**源代码**
```
<?php
/**
 * Created by PhpStorm.
 * User: Administrator
 * Date: 2017/3/16 0016
 * Time: 16:01
 */
namespace Server\Controller;
use Think\Controller;

class ServerController extends Controller{
    const TOKEN = 'API';

    //响应前台的请求
    public function respond(){
        //验证身份
        $timeStamp = $_GET['t'];
        $randomStr = $_GET['r'];
        $signature = $_GET['s'];
        $str = $this -> arithmetic($timeStamp,$randomStr);
        if($str != $signature){
            echo "-1";
            exit;
        }
        //模拟数据
        $arr['name'] = 'api';
        $arr['age'] = 15;
        $arr['address'] = 'zz';
        $arr['ip'] = "192.168.0.1";
        echo json_encode($arr);
    }

    /**
     * @param $timeStamp 时间戳
     * @param $randomStr 随机字符串
     * @return string 返回签名
     */
    public function arithmetic($timeStamp,$randomStr){
        $arr['timeStamp'] = $timeStamp;
        $arr['randomStr'] = $randomStr;
        $arr['token'] = self::TOKEN;
        //按照首字母大小写顺序排序
        sort($arr,SORT_STRING);
        //拼接成字符串
        $str = implode($arr);
        //进行加密
        $signature = sha1($str);
        $signature = md5($signature);
        //转换成大写
        $signature = strtoupper($signature);
        return $signature;
    }
}
```

**结果**
string(57) "{"name":"api","age":15,"address":"zz","ip":"192.168.0.1"}"

**总结**
这种方法只是其中的一种方法，其实还有很多方法都是可以进行安全验证的。
