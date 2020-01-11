MPDF类库下载地址：https://download.csdn.net/download/babayao/10592355
```
       require_once 'vendor/autoload.php';  //引入类库文件
       $config = [
			'mode' => 'utf-8',
			'format' => 'WEIXIN',
			'default_font_size' => 0,
			'default_font' => 'simhei',
			'mgl' => 52.5,
			'mgr' => 52.5,
			'mgt' => 64,
			'mgb' => 23,
			'mgh' => 38,
			'mgf' => 23,
			'orientation' => 'P'];
		$mpdf = new \Mpdf\Mpdf($config);
        //添加页眉和页脚到pdf中
		$mpdf->SetHTMLHeader($header);
		$mpdf->SetHTMLFooter($footer);
		$style = file_get_contents('https://XXX.com/mpdf.css');
		// $t0    = microtime(1);
		$mpdf->WriteHTML($style, 1);
		$mpdf->WriteHTML($page1);
		$mpdf->AddPage();
		$mpdf->WriteHTML($page2 . $page3 . $page4);
		$fileName = $purchaseid . '.pdf';
		//输出pdf文件
		$mpdf->Output('pdf/' . $fileName, 'I'); //'I'表示在线展示 'D'则显示下载窗口 'F'保存本地文件
```
