类库使用fpdf和fpdi(处理PDF，例如合并PDF和PDF水印处理)
除此之外也可以插入HTML代码来生成PDF文件。
类库下载链接：https://download.csdn.net/download/babayao/10592274
```
                        $pagesize = 50;	//每页条数
						$num = $num > 5000 ? 5000 : $num;	//总条数
						$pagenum = ceil($num / $pagesize);  //总页数


						$file = iconv("UTF-8","gbk",'2018年全网'.$head_title.'-'.$wds['wd'].'.pdf');
						$pdf = new PDF_Chinese('P','pt','A3');
						$pdf->AddGBFont('simhei', iconv("UTF-8", "GB2312//IGNORE",'黑体'));

						$pdf->SetLeftMargin(28);
						$pdf->SetRightMargin(28);
						$pdf -> SetTextColor(0,0,0);

						$pdf->SetFont('simhei', '', 10);
						$pdf->AddPage();
						$pdf->Cell (640, 20, iconv("UTF-8","gbk",'2018年全网'.$head_title.'  ------  '.$wds['wd']));
						$pdf->Cell (100, 20, iconv("UTF-8","gbk",'让目标用户更精准，转化率更高'));
						$pdf -> Ln(10);
						$pdf->Cell (740, 20, '---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------');
						$pdf -> Ln();
						$pdf -> Write (5, iconv("UTF-8","gbk",'本数据生成时间为' . date('Y年m月d日H点i分').'，您所看到的内容为截止该时间点此关键词的数据快照，共计' . $num . '条。'));
						$pdf -> Ln(300);
						$pdf->SetFont('simhei', 'B', 44);
						$pdf -> Cell(740,30,iconv("UTF-8","gbk",'2018年全网'.$head_title), 0, 0, 'C');
						$pdf -> Ln(80);
						$pdf->SetFont('simhei', '', 32);
						$pdf -> Cell(740,30,iconv("UTF-8","gbk",$wds['wd']), 0, 0, 'C');

						$pdf -> Ln(650);
						$pdf -> SetFillColor(0,0,0);
						$pdf->SetFont('simhei', 'I',8); //设置页脚字体
						$pdf->Cell (750, 20, iconv("UTF-8","gbk",'2018年全网'.$head_title.'t.gongchang.com/keyword'));
						$pdf->Cell (50, 20,iconv("UTF-8","gbk", '共' . $pagenum . '页'));
						$pdf -> SetFillColor(196,221,225);

						for($p = 1;$p <= $pagenum;$p++) {
							$pdf->SetFont('simhei', '', 10);
							$pdf->AddPage();
							$pdf->Cell (640, 20, iconv("UTF-8","gbk",'2018年全网'.$head_title.'  ------  '.$wds['wd']));
							$pdf->Cell (100, 20, iconv("UTF-8","gbk",'让目标用户更精准，转化率更高'));
							$pdf -> Ln(10);
							$pdf->Cell (740, 20, '---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------');
							$pdf -> Ln(20);
							$pdf -> SetFillColor(196,221,225);
							$pdf -> Cell(160,20,iconv("UTF-8", "gbk",'关键词'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'搜索结果'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'竞价公司数量'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'长尾词数量'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'百度指数'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'百度PC检索量'), 1, 0, 'C');
							$pdf -> Cell(90,20,iconv("UTF-8", "gbk",'百度移动检索量'), 1, 0, 'C');
							$pdf -> Cell(93,20,iconv("UTF-8", "gbk",'竞价竞争激烈程度'), 1, 0, 'C');
							$pdf -> Ln();


							$offset = ($p-1) * $pagesize;	//当前页
							$result = $this->db->query("SELECT * FROM destoon_miningword where {$condition} ORDER BY id ASC LIMIT $offset, $pagesize");
							$a = 1;
							while ($r = $this->db->fetch_array($result)) {
								if($a % 2 == 1) {
									$pdf -> SetFillColor(212,227,232);
								} else {
									$pdf -> SetFillColor(196,221,225);
								}
								$pdf -> Cell(160,20,iconv("UTF-8", "gbk",$r['word']), 1, 0, 'L', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['collect_count']), 1, 0, 'C', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['bidword_company_count']), 1, 0, 'C', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['long_keyword_count']), 1, 0, 'C', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['baidu_index']), 1, 0, 'C', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['bidword_kwc']), 1, 0, 'C', TRUE);
								$pdf -> Cell(90,20,iconv("UTF-8", "gbk",$r['bidword_pcpv']), 1, 0, 'C', TRUE);
								$pdf -> Cell(93,20,iconv("UTF-8", "gbk",$r['bidword_wisepv']), 1, 0, 'C', TRUE);
								$pdf -> Ln();
								$a = $a + 1;
							}
							$pdf -> Ln(10);
							$pdf -> SetFillColor(0,0,0);
							$pdf->SetFont('simhei', 'I',8); //设置页脚字体
							$pdf->Cell (750, 20, iconv("UTF-8","gbk",'2018年全网'.$head_title.'t.gongchang.com/keyword'));
							$pdf->Cell (50, 20,iconv("UTF-8","gbk", $p .'/' . $pagenum));
						}
						$pdf->Output('../file/miningword/' . $file,'F');

						$ipdf = new FPDI();
						$pageCount = $ipdf->setSourceFile('../file/miningword/' . $file);
						for ($pageNo = 1; $pageNo <= $pageCount; $pageNo++)
						{
							$templateId = $ipdf->importPage($pageNo);
							$size = $ipdf->getTemplateSize($templateId);
							if ($size['w'] > $size['h']) {
								$ipdf->AddPage('L', array($size['w'], $size['h']));
							} else {
								$ipdf->AddPage('P', array($size['w'], $size['h']));
							}
							$ipdf->useTemplate($templateId);
							$ipdf->image(DT_ROOT . "/api/fpdf/shuiyin.png", 0, 0, 300, 500);
						}
						$ipdf->Output('../file/miningword/' . $file,'F');
```
