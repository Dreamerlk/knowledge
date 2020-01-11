```
<?php
/**
 * @Author: likunlun
 * @Date:   2019-04-29 14:21:50
 * 使用说明：
 * 第一个参数：gcv4.http - 代表项目名称
 * 第二个参数：test - 代表发测试   prod - 代表发生产
 * 第三个参数：prod.gz.01 - 代表要发布的set，为空则发布所有节点
 * 第四个参数： d0f3c4xxxx - 代表要发布的版本  为空则发布最新版本
 * php deploy.php gcv4.http prod 1 就是发布所有set分组为01的节点
 * php deploy.php gcv4.http prod prod.gz.01 就是发布所有set名为prod,set区域为gz,set分组为01的节点
 * php deploy.php gcv4.http prod  就是发布所有节点
 * php deploy.php gcv4.http prod '' d0f3c4xxxx 就是发布所有节点到指定版本
 */

$project = [
    'gcv4.http'              => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.http.git',
        'module_name' => 'http',
    ],
    'gcv4.globalsetservice'  => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.globalsetservice.git',
        'module_name' => 'globalsetService',
    ],
    'gcv4.product'           => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.product.git',
        'module_name' => 'ProductServer',
    ],
    'gcv4.member.rpc'        => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.member.rpc.git',
        'module_name' => 'memberrpc',
    ],
    'gcv4.customer.rpc'      => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.customer.rpc.git',
        'module_name' => 'CustomerServer',
    ],
    'gcv4.warning.rpc'       => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.warning.rpc.git',
        'module_name' => 'warningService',
    ],
    'gcv4.article.service'   => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.article.service.git',
        'module_name' => 'articleServer',
    ],
    'gcv4.finance.rpc'       => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.finance.rpc.git',
        'module_name' => 'financeService',
    ],
    'gcv4.googlereport.rpc'  => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.googlereport.rpc.git',
        'module_name' => 'adsreportService',
    ],
    'gcv4.googleaccount.rpc' => [
        'application' => 'gc',
        'gitlab'      => 'ssh://git@code.xizhi.com:2828/gongchang/gcv4.googleaccount.rpc.git',
        'module_name' => 'googleaccountServer',
    ],
];

$config = [
    'test' => [
        'kongurl'  => 'http://192.168.8.39:8001',
        'loginurl' => 'http://192.168.8.21:3001',
        'endpoint' => 'http://192.168.8.21:3000',
        'username' => 'root',
        'password' => '123456',
        'node'     => ['01' => '192.168.8.22', '02' => '192.168.8.23'],
    ],
    'prod' => [
        'kongurl'  => 'http://10.101.30.102:8001',
        'loginurl' => 'http://10.101.40.101:30456',
        'endpoint' => 'http://10.101.40.101:30455',
        'username' => 'root',
        'password' => '123456',
    ],
];

$projectName = array_keys($project);
//项目路径
$deployProject = strtolower($argv[1]);

//发布环境
if (!in_array($argv[2], ['test', 'prod'])) {
    exit("发布环境 集合不对");
}

//set集合
if (isset($argv[3])) {
    if (is_numeric($argv[3])) {
        $set = sprintf("%02d", $argv[3]);
    } else {
        $set = $argv[3];
    }
}

if (empty($deployProject)) {
    exit("项目不对");
}

if (!in_array($deployProject, $projectName)) {
    exit("发布的项目名称不正确");
}
$gitlab = $project[$deployProject]['gitlab'];
$a      = glob($deployProject);
if (empty($a)) {
    $clonecmd = "git clone $gitlab";
    if (!empty($argv[4])) {
        $version = $argv[4];
        $clonecmd .= " && cd $deployProject && git checkout $version";
    }

    @exec($clonecmd);
} else {
    @exec("cd $deployProject && git reset --hard HEAD && git pull");
}
//项目代码更新 代码 初始化
$cmd = "
cd $deployProject &&
cd src &&
pwd &&
composer up --no-dev &&
chmod u+x ../scripts/tars2php.sh  &&
../scripts/tars2php.sh
";
@exec($cmd, $output);

$application = $project[$deployProject]['application'];
$moduleName  = $project[$deployProject]['module_name'];

if ($argv[2] == 'test') {
    $config = $config['test'];
}
if ($argv[2] == 'prod') {
    $config = $config['prod'];
}

//上传tars包
$command = "cd $deployProject && git rev-parse --short HEAD && git rev-list --format=%B --max-count=1 HEAD ";
@exec($command, $comment);

//登录，获取ticker
$login  = new \TarsLogin($config['loginurl'], $config['username'], $config['password']);
$ticker = $login->call();
if (empty($ticker)) {
    exit('tars登录失败');
}
$endpoint = $config['endpoint'];
$username = $config['username'];
$cookie   = "ticket=$ticker; uid=$username";
$fields   = [
    'application' => $application,
    'module_name' => $moduleName,
    'comment'     => @($comment[1] . $comment[2]) ?? '',
    'task_id'     => time(),
];
echo `pwd`;
$files = glob("$deployProject/src/*.tar.gz");
if (!$files) {
    $command = "cd $deployProject && pwd && cd src && composer deploy";
    @exec($command, $comment);
    $files = glob("$deployProject/src/*.tar.gz");
    if (!$files) {
        echo "没有找到需要上传的包" . PHP_EOL;
        exit(2);
    }
}
$files = array("suse" => $files[0]);

$upload = new \TarsUpload($endpoint, $cookie);
$upload->setFields($fields);
$upload->setFiles($files);
$patch = $upload->call();
@unlink($files['suse']);

//删除本地拉去的代码
@exec("rm -rf $deployProject");

//nodeip根据发布项目时指定的app、module、set获取
$task     = new \TarsTask($endpoint, $cookie, $application, $moduleName);
$serverid = "1$application.";
if (!empty($set)) {
    $set = array_reverse(explode('.', $set));
    $i   = 2;
    foreach ($set as $v) {
        $serverid .= $i . $v . '.';
        $i++;
    }
}
$serverid .= "5$moduleName";
$serverlists = $task->getServerList($serverid);
if (empty($serverlists['data']) || $serverlists['ret_code'] != 200) {
    $msg = isset($serverlists['err_msg']) ? $serverlists['err_msg'] : '';
    echo "获取应用需要发布的节点失败" . $serverid . $msg . PHP_EOL;
    exit(0);
}

//获取Kong的配置信息
$kongName = 'gc.http';
$kong     = new Kong($config['kongurl']);
$rs       = $kong->getUpstream($kongName);
if (!$rs['error'] && isset($rs['result']['message'])
    && ($rs['result']['message'] == 'Not found')) {
    echo "upstream not found" . PHP_EOL;
    exit(1);
}
$targets = $kong->getUpstreamTargets($kongName);
//依次发布各节点
foreach ($serverlists['data'] as $k => $v) {
    $nodeip = $v['node_name'];
    var_dump($nodeip);
    //取出当前节点的target
    $isupstream = false;
    $isdeploy   = false;
    foreach ($targets as $target) {
        if (strpos($target['target'], $nodeip) !== false) {
            if ($isdeploy) {
                break;
            }
            $isupstream = true;
            //先停掉Upstream
            $param = [
                'target' => $target['target'],
                'weight' => 0,
            ];
            $rs = $kong->updateUpstreamTargets($kongName, $param);
            if ($rs['error']) {
                echo "set target weight=0 error" . PHP_EOL;
                var_dump($rs);
                exit(1);
            }
            echo "停止upstream $nodeip target成功" . PHP_EOL;

            $endpoint = $config['endpoint'];
            //开始发布task
            $rs = $task->addTask($v, $patch['data']['id'], '');
            if (!$rs['status']) {
                echo "添加任务出错，" . $rs['msg'] . PHP_EOL;
                var_dump($rs);
                exit(3);
            }
            if ($rs['data']['ret_code'] != 200) {
                echo "任务状态码出错，" . $rs['data']['ret_code'] . $rs['data']['err_msg'] . PHP_EOL;
                var_dump($rs);
                exit(4);
            }
            $taskno = $rs['data']['data'];
            echo "添加任务成功, taskno:" . $taskno . PHP_EOL;

            //检测task状态
            $finish = false;
            $sleep  = 0;
            while (!$finish) {
                $rs = $task->check($taskno);
                if ($rs['status'] && ($rs['data']['ret_code'] == 200) && ($rs['data']['data']['status'] == 2)) {
                    echo "任务发布成功，taskno:" . $taskno . PHP_EOL;
                    $finish = true;
                } elseif ($rs['status'] && ($rs['data']['ret_code'] == 200) && ($rs['data']['data']['status'] == 3)) {
                    echo "任务状态出错，taskno:" . $taskno . PHP_EOL;
                    var_dump($rs);
                    $finish = true;
                } else {
                    if ($sleep > 40) {
                        echo "任务超过2分钟，失败:" . $taskno . PHP_EOL;
                        exit(6);
                    }
                    echo "检查任务状态，taskno:" . $taskno . PHP_EOL;
                    sleep(3);
                    $sleep++;
                }
            }

            if ($finish) {
                $isdeploy = true;
                //恢复业务
                $param = [
                    'target' => $target['target'],
                    'weight' => 99,
                ];
                $rs = $kong->updateUpstreamTargets($kongName, $param);
                if ($rs['error']) {
                    echo "set target weight=100 error" . PHP_EOL;
                    var_dump($rs);
                    exit(1);
                }
                echo "开启upstream $nodeip target成功" . PHP_EOL;
            } else {
                echo "tars task error, exit" . PHP_EOL;
                exit(6);
            }
        }
    }

    var_dump($v['application'] . '.' . $v['server_name'] . ':' . $v['set_name'] . '.' . $v['set_area'] . '.' . $v['set_group']);

    if (!$isupstream) {
        //添加upstream
        $param = [
            'target' => $nodeip . ':10080',
            'weight' => 99,
            'tags'   => [],
        ];
        $rs = $kong->createUpstreamTargets($kongName, $param);
        var_dump($rs);
    }
}

class TarsLogin
{

    protected $url;
    protected $uri = "/api/login";
    protected $username;
    protected $password;

    public function __construct(string $endpoint, string $username, string $password)
    {
        $this->url      = $endpoint . $this->uri;
        $this->username = $username;
        $this->password = $password;
    }

    public function call()
    {
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $this->url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_POSTFIELDS     => json_encode(['uid' => $this->username, 'password' => $this->password]),
            CURLOPT_HTTPHEADER     => array('Content-Type:application/json'),
            CURLOPT_POST           => 1,
            CURLOPT_ENCODING       => 'gzip, deflate',
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        $rs = [
            'response' => $response,
            //'info' => $info,
            'error'    => $err,
        ];

        if ($rs['error']) {
            echo "登录出错" . PHP_EOL;
            var_dump($rs['error']);
            exit(1);
        }
        $rs['response'] = json_decode($rs['response'], true);
        if ($rs['response']['ret_code'] != 200) {
            echo "登录出错，ret_code != 200" . PHP_EOL;
            var_dump($rs['response']);
            exit(1);
        }
        $ticker = $rs['response']['data']['ticket'];
        echo "登录成功, ticker:" . $ticker . PHP_EOL;

        return $ticker;
    }
}

class TarsUpload
{

    protected $url;
    protected $uri = "/pages/server/api/upload_patch_package";
    protected $fields;
    protected $files;
    protected $cookie;

    public function __construct(string $endpoint, string $cookie)
    {
        $this->url    = $endpoint . $this->uri;
        $this->cookie = $cookie;
    }

    public function setFields(array $fields)
    {
        $this->fields = $fields;
    }

    public function setFiles(array $files)
    {
        foreach ($files as $k => $f) {
            $this->files[$k] = file_get_contents($f);
        }
    }

    public static function buildDataFiles($boundary, $fields, $files)
    {
        $data = '';
        $eol  = "\r\n";

        $delimiter = '-------------' . $boundary;
        foreach ($fields as $name => $content) {
            $data .= "--" . $delimiter . $eol
                . 'Content-Disposition: form-data; name="' . $name . "\"" . $eol . $eol
                . $content . $eol;
        }
        foreach ($files as $name => $content) {
            $data .= "--" . $delimiter . $eol
                . 'Content-Disposition: form-data; name="' . $name . '"; filename="' . $name . '"' . $eol
                . 'Content-Transfer-Encoding: binary' . $eol
            ;

            $data .= $eol;
            $data .= $content . $eol;
        }
        $data .= "--" . $delimiter . "--" . $eol;

        return $data;
    }

    public function call()
    {
        $curl      = curl_init();
        $boundary  = uniqid();
        $delimiter = '-------------' . $boundary;
        $post_data = self::buildDataFiles($boundary, $this->fields, $this->files);
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $this->url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 60,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_CUSTOMREQUEST  => "POST",
            CURLOPT_POST           => 1,
            CURLOPT_POSTFIELDS     => $post_data,
            CURLOPT_HTTPHEADER     => array(
                "Content-Type: multipart/form-data; boundary=" . $delimiter,
                "Content-Length: " . strlen($post_data),
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        $rs = [
            'response' => $response,
            //'info' => $info,
            'error'    => $err,
        ];

        if ($rs['error']) {
            echo "上传包出错" . PHP_EOL;
            var_dump($rs['error']);
            exit(2);
        }
        $patch = json_decode($rs['response'], true);
        if ($patch['data']['id'] < 1) {
            echo "上传包出错，patch id获取失败" . PHP_EOL;
            var_dump($rs);
            exit(2);
        }
        echo "上传包成功, patchid:" . $patch['data']['id'] . PHP_EOL;
        return $patch;
    }
}

class TarsTask
{

    protected $url;
    protected $uri = "/pages/server/api/add_task";
    protected $checkurl;
    protected $checkuri = "/pages/server/api/task?task_no=";
    protected $nodeurl;
    protected $nodeuri = "/pages/server/api/server_list?tree_node_id=";
    protected $adapterurl;
    protected $adapteruri = "/pages/server/api/adapter_conf_list?id=";
    protected $fields;
    protected $files;
    protected $cookie;
    protected $node;
    protected $application;
    protected $moduleName;

    public function __construct(string $endpoint, string $cookie, string $application, string $moduleName)
    {
        $this->url         = $endpoint . $this->uri;
        $this->checkurl    = $endpoint . $this->checkuri;
        $this->nodeurl     = $endpoint . $this->nodeuri;
        $this->adapterurl  = $endpoint . $this->adapteruri;
        $this->cookie      = $cookie;
        $this->application = $application;
        $this->module_name = $moduleName;
        $this->getNodeInfo();
    }

    public function getNodeInfo()
    {
        $url  = $this->nodeurl . '1' . $this->application . '.5' . $this->module_name;
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        if (!$err) {
            $this->node = json_decode($response, true);
            return $this->node;
        } else {
            var_dump($err);
            $this->node = [];
            return [];
        }
    }

    public function getServerList($nodeid)
    {
        $url  = $this->nodeurl . $nodeid;
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        if (!$err) {
            return json_decode($response, true);
        } else {
            var_dump($err);
            return [];
        }
    }

    public function getSetInfo($set1, $set2, $set3, $set4, $set5)
    {
        $url = $this->nodeurl . '1' . $set1 . '.2' . $set2
            . '.3' . $set3 . '.4' . $set4 . '.5' . $set5;
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        if (!$err) {
            return json_decode($response, true);
        } else {
            var_dump($err);
            return [];
        }
    }

    public function getAdapterConf($id)
    {
        $url  = $this->adapterurl . $id;
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        if (!$err) {
            return json_decode($response, true);
        } else {
            var_dump($err);
            return [];
        }
    }

    public function addTask($node, $patch_id, $update_text)
    {
        $items = [];
        if ($node['id'] > 0) {
            $item = [
                'server_id'  => $node['id'],
                'command'    => 'patch_tars',
                'parameters' => [
                    'patch_id'    => $patch_id,
                    'bak_flag'    => false,
                    'update_text' => $update_text,
                ],
            ];
            $items[] = $item;
        }
        if ($items) {
            $data = [
                'serial' => true,
                'items'  => $items,
            ];
            $rs = $this->_addTaskCall($data);
            if (!$rs['error']) {
                $data = [
                    'status' => true,
                    'msg'    => '添加任务成功',
                    'data'   => json_decode($rs['response'], true),
                ];
            }
        }

        return $data;
    }

    public function check($task_no)
    {
        $url  = $this->checkurl . $task_no;
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        if (!$err) {
            $data = [
                'status' => true,
                'data'   => json_decode($response, true),
            ];
        } else {
            var_dump($err);
            $data = [
                'status' => false,
                'msg'    => $err,
            ];
        }
        return $data;
    }

    private function _addTaskCall($data)
    {
        $curl = curl_init();
        curl_setopt_array($curl, array(
            CURLOPT_URL            => $this->url,
            CURLOPT_RETURNTRANSFER => 1,
            CURLOPT_MAXREDIRS      => 10,
            CURLOPT_TIMEOUT        => 30,
            //CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_CUSTOMREQUEST  => "POST",
            CURLOPT_POST           => 1,
            CURLOPT_ENCODING       => 'gzip, deflate',
            CURLOPT_POSTFIELDS     => json_encode($data),
            CURLOPT_HTTPHEADER     => array(
                "Cookie: " . $this->cookie,
                "X-Requested-With: XMLHttpRequest",
                "Content-Type: application/json",
            ),
        ));
        $response = curl_exec($curl);
        $info     = curl_getinfo($curl);
        $err      = curl_error($curl);
        curl_close($curl);
        $data = [
            'response' => $response,
            //'info' => $info,
            'error'    => $err,
        ];
        return $data;
    }
}
class TarsUtils
{

    public static function getEndpointInfo($endpoint)
    {
        $parts     = explode('-', $endpoint);
        $sHost     = '';
        $sProtocol = '';
        $iPort     = '';
        $iTimeout  = '';
        $bIp       = '';
        foreach ($parts as $part) {
            if (strstr($part, 'tcp')) {
                $sProtocol = 'tcp';
            } elseif (strstr($part, 'udp')) {
                $sProtocol = 'udp';
            } elseif (strpos($part, 'h') !== false) {
                $sHost = trim($part, " h\t\r");
            } elseif (strpos($part, 'b') !== false) {
                $bIp = trim($part, " b\t\r");
            } elseif (strpos($part, 'p') !== false) {
                $iPort = trim($part, " p\t\r");
            } elseif (strpos($part, 't') !== false) {
                $iTimeout = trim($part, " t\t\r");
            }
        }

        return [
            'sHost'     => $sHost,
            'sProtocol' => $sProtocol,
            'iPort'     => $iPort,
            'iTimeout'  => $iTimeout,
            'bIp'       => $bIp,
            'sIp'       => $sHost,
        ];
    }
}

class Kong
{

    protected $url;

    public function __construct($url)
    {
        $this->url = $url;
    }

    public function getUpstream($name)
    {
        $url = $this->url . '/upstreams/' . $name;
        return $this->_curl($url);
    }

    public function getUpstreamTargets($name)
    {
        $url     = $this->url . '/upstreams/' . $name . '/targets';
        $targets = $this->_curl($url);

        if ($targets['error']) {
            echo "get upstream targets error" . PHP_EOL;
            var_dump($targets);
            exit(1);

        }
        $targets = $targets['result']['data'];
        if (count($targets) <= 1) {
            //upstream targets必须大于1
            //echo "get upstream targets count <= 1, exit" . PHP_EOL;
            //exit(1);
        }
        return $targets;

    }

    public function createUpstream($name, array $param = [])
    {
        $url           = $this->url . '/upstreams/';
        $param['name'] = $name;
        return $this->_curl($url, $param);
    }

    public function createUpstreamTargets($name, array $param)
    {
        $url = $this->url . '/upstreams/' . $name . '/targets';
        return $this->_curl($url, $param);
    }

    public function updateUpstreamTargets($name, array $param)
    {
        $url = $this->url . '/upstreams/' . $name . '/targets';
        return $this->_curl($url, $param);
    }

    private function _curl($url, array $param = [])
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        if ($param) {
            curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($param));
            curl_setopt($ch, CURLOPT_POST, 1);
        }

        $headers   = array();
        $headers[] = "Content-Type: application/x-www-form-urlencoded";
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

        $result = curl_exec($ch);
        $err    = curl_error($ch);
        curl_close($ch);

        return [
            'result' => json_decode($result, true) ?? [],
            'error'  => $err,
        ];

    }
}

```
