~~~
第一种方式：
使用方法：
$arr = ['name'=>'test','name2'=>['name22'=>'test1']];//数组维度可以随意
$xml = arrayToXml($arr);
function arrayToXml($arr)
{
    foreach ($arr as $key => $val) {
         //处理xml中为同一级内容且key名字相同的情况 
        //例：<contactPerson>****</contactPerson><contactPerson>###</contactPerson>
        $key = explode("_", $key);
        $key = $key['0'];
        if (is_array($val)) {
            $xml .= "<" . $key . ">" . arrayToXml($val) . "</" . $key . ">";
        } else {
            //处理特殊符号
            $val = str_ireplace(['<', '>', '&', "'", '"', ''], ["&lt;", "&gt;", "&amp;", "&apos;", "&quot;", ''], $val);
            $reg = '/[^\x{0009}\x{000a}\x{000d}\x{0020}-\x{D7FF}\x{E000}-\x{FFFD}]+/u'; 过滤掉类似<0x0b><0x1e>特殊符号
            $val = preg_replace($reg, '', $val);
            $xml .= "<" . $key . ">" . $val . "</" . $key . ">";
        }
    }
    return $xml;
}
输出结果：

<name>test</name>
<name2>
    <name22>test1</name22>
</name2>

第二种方式：
使用方法：
$xml = new SimpleXMLElement('<root/>');
echo $xml->asXML();//echo $xml->asXML('test.xml');

function array_to_XML($obj, $array)
{
    foreach ($array as $k => $v) {
        if (is_array($v)) {
            $node = $obj->addChild($k);
            array_to_XML($node, $v);
        } else {
            $obj->addChild($k, $v);
        }
    }
}

格式化xml，为了方便阅读：
function formateFile($xmlFile)
{
    $dom                     = new DOMDocument('1.0');
    $dom->preserveWhiteSpace = false;
    $dom->formatOutput       = true;
    $dl                      = @$dom->load($xmlFile);
    $dom->save($xmlFile);
}
~~~
