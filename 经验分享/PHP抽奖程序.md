```
$arr = array(
          array('id' => 1, 'name' => '2积分', 'v' => 60000, 'gift' => 2),
          array('id' => 2, 'name' => '9.4折vip折扣券(限VIP基础版）', 'v' => 10000),
          );
ps：id为奖品id,name为奖品名字，V为奖品概率，gift为奖品值

//抽奖
function get_rand($proArr)
{
    $result = array();
    foreach ($proArr as $key => $val) {
        $arr[$key] = $val['v'];
    }
    // 概率数组的总概率
    $proSum = array_sum($arr);
    asort($arr);
    // 概率数组循环
    foreach ($arr as $k => $v) {
        $randNum = mt_rand(1, $proSum);
        if ($randNum <= $v) {
            $result = $proArr[$k];
            break;
        } else {
            $proSum -= $v;
        }
    }
    return $result;
}
```
