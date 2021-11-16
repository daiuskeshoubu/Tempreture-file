#<?php
/** jmpWeatherInit.php
 * 気象庁週間天気予報：予報地点情報ファイル作成
 *
 * @copyright	(c)studio pahoo
 * @author		パパぱふぅ
 * @動作環境	PHP 5/7/8
 * @参考URL		https://www.pahoo.org/e-soul/webtech/php06/php06-73-01.shtm
 *
*/
// 初期化処理 ================================================================
define('INTERNAL_ENCODING', 'UTF-8');
mb_internal_encoding(INTERNAL_ENCODING);
mb_regex_encoding(INTERNAL_ENCODING);
define('MYSELF', basename($_SERVER['SCRIPT_NAME']));
define('REFERENCE', 'https://www.pahoo.org/e-soul/webtech/php06/php06-73-01.shtm');

//プログラム・タイトル
define('TITLE', '気象庁天気予報：予報地点情報ファイル作成');

//リファラチェック＋リリースフラグの設定
if (isset($_SERVER['HTTP_HOST']) && ($_SERVER['HTTP_HOST'] == 'localhost')) {
	define('FLAG_RELEASE', FALSE);
	define('REFER_ON', '');
	ini_set('display_errors', 1);
	ini_set('error_reporting', E_ALL);
} else {
	//リリース・フラグ（公開時にはTRUEにすること）
	define('FLAG_RELEASE', TRUE);
	//リファラ・チェック（直リン防止用；空文字ならチェックしない）
	define('REFER_ON', 'www.pahoo.org');
}

//地域観測所一覧（アメダス）
//https://www.jma.go.jp/jma/kishou/know/amedas/kaisetsu.html から取得
define('FILE_AMEDAS', 'ame_master20210319.csv');

//表示幅（単位：ピクセル）
define('WIDTH',  600);

//気象情報クラス：include_pathが通ったディレクトリに配置
require_once('pahooWeather.php');

//都道府県コード
$TablePref = array(
//都道府県コード  都道府県名 地方コード 地方名
'01' => array('北海道',   '01', '北海道地方'),
'02' => array('青森県',   '02', '東北地方'),
'03' => array('岩手県',   '02', '東北地方'),
'04' => array('宮城県',   '02', '東北地方'),
'05' => array('秋田県',   '02', '東北地方'),
'06' => array('山形県',   '02', '東北地方'),
'07' => array('福島県',   '02', '東北地方'),
'08' => array('茨城県',   '03', '関東地方'),
'09' => array('栃木県',   '03', '関東地方'),
'10' => array('群馬県',   '03', '関東地方'),
'11' => array('埼玉県',   '03', '関東地方'),
'12' => array('千葉県',   '03', '関東地方'),
'13' => array('東京都',   '03', '関東地方'),
'14' => array('神奈川県', '03', '関東地方'),
'15' => array('新潟県',   '04', '中部地方'),
'16' => array('富山県',   '04', '中部地方'),
'17' => array('石川県',   '04', '中部地方'),
'18' => array('福井県',   '04', '中部地方'),
'19' => array('山梨県',   '04', '中部地方'),
'20' => array('長野県',   '04', '中部地方'),
'21' => array('岐阜県',   '04', '中部地方'),
'22' => array('静岡県',   '04', '中部地方'),
'23' => array('愛知県',   '04', '中部地方'),
'24' => array('三重県',   '05', '関西地方'),
'25' => array('滋賀県',   '05', '関西地方'),
'26' => array('京都府',   '05', '関西地方'),
'27' => array('大阪府',   '05', '関西地方'),
'28' => array('兵庫県',   '05', '関西地方'),
'29' => array('奈良県',   '05', '関西地方'),
'30' => array('和歌山県', '05', '関西地方'),
'31' => array('鳥取県',   '06', '中国地方'),
'32' => array('島根県',   '06', '中国地方'),
'33' => array('岡山県',   '06', '中国地方'),
'34' => array('広島県',   '06', '中国地方'),
'35' => array('山口県',   '06', '中国地方'),
'36' => array('徳島県',   '07', '四国地方'),
'37' => array('香川県',   '07', '四国地方'),
'38' => array('愛媛県',   '07', '四国地方'),
'39' => array('高知県',   '07', '四国地方'),
'40' => array('福岡県',   '08', '九州地方'),
'41' => array('佐賀県',   '08', '九州地方'),
'42' => array('長崎県',   '08', '九州地方'),
'43' => array('熊本県',   '08', '九州地方'),
'44' => array('大分県',   '08', '九州地方'),
'45' => array('宮崎県',   '08', '九州地方'),
'46' => array('鹿児島県', '08', '九州地方'),
'47' => array('沖縄県',   '08', '九州地方'),
);

//府県天気予報地点テーブル
//https://www.jma.go.jp/bosai/forecast/#area_type=offices&area_code={areaCode}
$TableStation = array(
'北海道'   => array(
	'宗谷地方' => array('稚内'),
	'上川地方' => array('旭川'),
	'留萌地方' => array('留萌'),
	'網走地方' => array('網走'),
	'北見地方' => array('北見'),
	'紋別地方' => array('紋別'),
	'十勝地方' => array('帯広'),
	'根室地方' => array('根室'),
	'釧路地方' => array('釧路'),
	'胆振地方' => array('室蘭'),
	'日高地方' => array('浦河'),
	'石狩地方' => array('札幌'),
	'空知地方' => array('岩見沢'),
	'後志地方' => array('倶知安'),
	'渡島地方' => array('函館'),
	'檜山地方' => array('江差'),
),
'青森県'   => array(
	'津軽地方'     => array('青森', '深浦', '弘前'),
	'下北地方'     => array('むつ'),
	'三八上北地方' => array('八戸'),
),
'岩手県'   => array(
	'内陸'     => array('盛岡', '二戸', '一関'),
	'沿岸北部' => array('宮古'),
	'沿岸南部' => array('大船渡'),
),
'宮城県'   => array(
	'東部' => array('仙台', '石巻', '古川'),
	'西部' => array('白石'),
),
'秋田県'   => array(
	'沿岸' => array('秋田'),
	'内陸' => array('鷹巣', '横手'),
),
'山形県'   => array(
	'村山' => array('山形'),
	'置賜' => array('米沢'),
	'庄内' => array('酒田'),
	'最上' => array('新庄'),
),
'福島県'   => array(
	'中通り' => array('福島',   '白河', '郡山'),
	'浜通り' => array('小名浜', '相馬'),
	'会津'   => array('若松',   '田島'),
),
'茨城県'   => array(
	'北部' => array('水戸'),
	'南部' => array('土浦'),
),
'栃木県'   => array(
	'南部' => array('宇都宮'),
	'北部' => array('大田原'),
),
'群馬県'   => array(
	'南部' => array('前橋'),
	'北部' => array('みなかみ'),
),
'埼玉県'   => array(
	'南部'     => array('さいたま'),
	'北部'     => array('熊谷'),
	'秩父地方' => array('秩父'),
),
'千葉県'   => array(
	'北西部' => array('千葉'),
	'北東部' => array('銚子'),
	'南部'   => array('館山'),
),
'東京都'   => array(
	'東京地方'     => array('東京'),
	'伊豆諸島北部' => array('大島'),
	'伊豆諸島南部' => array('八丈島'),
	'小笠原諸島'   => array('父島'),
),
'神奈川県' => array(
	'東部' => array('横浜'),
	'西部' => array('小田原'),
),
'新潟県'=> array(
	'下越' => array('新潟', '津川'),
	'中越' => array('長岡', '湯沢'),
	'上越' => array('高田'),
	'佐渡' => array('相川'),
),
'富山県'=> array(
	'東部' => array('富山'),
	'西部' => array('伏木'),
),
'石川県'=> array(
	'加賀' => array('金沢'),
	'能登' => array('輪島'),
),
'福井県'=> array(
	'嶺北' => array('福井', '大野'),
	'嶺南' => array('敦賀'),
),
'山梨県'=> array(
	'中・西部' => array('甲府'),
	'東部・富士五湖' => array('河口湖'),
),
'長野県'=> array(
	'北部' => array('長野'),
	'中部' => array('松本', '諏訪', '軽井沢'),
	'南部' => array('飯田'),
),
'岐阜県'=> array(
	'美濃地方' => array('岐阜'),
	'飛騨地方' => array('高山'),
),
'静岡県'=> array(
	'中部' => array('静岡'),
	'伊豆' => array('網代', '石廊崎'),
	'東部' => array('三島'),
	'西部' => array('浜松', '御前崎'),
),
'愛知県'=> array(
	'西部' => array('名古屋'),
	'東部' => array('豊橋'),
),
'三重県'=> array(
	'北中部' => array('津', '四日市', '上野'),
	'南部' => array('尾鷲'),
),
'滋賀県'=> array(
	'南部' => array('大津'),
	'北部' => array('彦根'),
),
'京都府'=> array(
	'南部' => array('京都'),
	'北部' => array('舞鶴'),
),
'大阪府'=> array(
	'大阪府' => array('大阪'),
),
'兵庫県'=> array(
	'南部' => array('神戸', '洲本', '姫路'),
	'北部' => array('豊岡'),
),
'奈良県'=> array(
	'北部' => array('奈良'),
	'南部' => array('風屋'),
),
'和歌山県'=> array(
	'北部' => array('和歌山'),
	'南部' => array('潮岬'),
),
'鳥取県'=> array(
	'東部'     => array('鳥取'),
	'中・西部' => array('米子'),
),
'島根県'=> array(
	'東部' => array('松江'),
	'西部' => array('浜田'),
	'隠岐' => array('西郷'),
),
'岡山県'=> array(
	'南部' => array('岡山'),
	'北部' => array('津山'),
),
'広島県'=> array(
	'南部' => array('広島', '呉', '福山'),
	'北部' => array('庄原'),
),
'山口県'=> array(
	'西部' => array('下関'),
	'中部' => array('山口'),
	'東部' => array('柳井'),
	'北部' => array('萩'),
),
'徳島県'=> array(
	'北部' => array('徳島', '池田'),
	'南部' => array('日和佐'),
),
'香川県'=> array(
	'香川県' => array('高松'),
),
'愛媛県'=> array(
	'中予' => array('松山'),
	'東予' => array('新居浜'),
	'南予' => array('宇和島'),
),
'高知県'=> array(
	'中部' => array('高知'),
	'東部' => array('室戸岬'),
	'西部' => array('清水'),
),
'福岡県'=> array(
	'福岡地方'   => array('福岡'),
	'北九州地方' => array('八幡'),
	'筑豊地方'   => array('飯塚'),
	'筑後地方'   => array('久留米'),
),
'佐賀県'=> array(
	'南部' => array('佐賀'),
	'北部' => array('伊万里'),
),
'長崎県'=> array(
	'南部'       => array('長崎'),
	'北部'       => array('佐世保'),
	'壱岐・対馬' => array('厳原'),
	'五島'       => array('福江'),
),
'熊本県'=> array(
	'熊本地方'       => array('熊本'),
	'阿蘇地方'       => array('阿蘇乙姫'),
	'天草・芦北地方' => array('牛深'),
	'球磨地方'       => array('人吉'),
),
'大分県'=> array(
	'中部' => array('大分'),
	'北部' => array('中津'),
	'西部' => array('日田'),
	'南部' => array('佐伯'),
),
'宮崎県'=> array(
	'南部平野部' => array('宮崎', '油津'),
	'北部平野部' => array('延岡'),
	'南部山沿い' => array('都城'),
	'北部山沿い' => array('高千穂'),
),
'鹿児島県'=> array(
	'薩摩地方' => array('鹿児島', '阿久根', '枕崎'),
	'大隅地方' => array('鹿屋'),
	'種子島・屋久島地方' => array('種子島'),
	'奄美地方' => array('名瀬', '沖永良部'),
),
'沖縄県'=> array(
	'本島中南部'   => array('那覇'),
	'本島北部'     => array('名護'),
	'久米島'       => array('久米島'),
	'大東島地方'   => array('南大東'),
	'宮古島地方'   => array('宮古島'),
	'石垣島地方'   => array('石垣島'),
	'与那国島地方' => array('与那国島'),
),
);

/**
 * 共通HTMLヘッダ
 * @global string $HtmlHeader
*/
$encode = INTERNAL_ENCODING;
$title  = TITLE;
$width  = WIDTH;
$HtmlHeader =<<< EOT
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="{$encode}">
<title>{$title}</title>
<meta name="author" content="studio pahoo" />
<meta name="copyright" content="studio pahoo" />
<meta name="ROBOTS" content="NOINDEX,NOFOLLOW" />
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache">
<style>
/* 予報地点一覧表 */
table {
	width: {$width}px;
	border-collapse: collapse;
	margin-top: 10px;
}
tr, td, th {
	border: 1px gray solid;
	padding: 4px;
	text-align: center;
	font-size: small;
}
.index {
	background-color: gainsboro;
}
</style>
</head>

EOT;

/**
 * 共通HTMLフッタ
 * @global string $HtmlFooter
*/
$HtmlFooter =<<< EOT
</html>

EOT;

// サブルーチン ==============================================================
/**
 * エラー処理ハンドラ
*/
function myErrorHandler ($errno, $errmsg, $filename, $linenum) {
	echo 'Sory, system error occured !';
	exit(1);
}
error_reporting(E_ALL);
if (FLAG_RELEASE)	$old_error_handler = set_error_handler('myErrorHandler');

//PHP5判定
if (! isphp5over()) {
	echo 'Error > Please let me run on PHP5 or more...';
	exit(1);
}

//リファラ・チェック
if (REFER_ON != '') {
	if (isset($_SERVER['HTTP_REFERER'])) {
		$url = parse_url($_SERVER['HTTP_REFERER']);
		$res = ($url['host'] == REFER_ON) ? TRUE : FALSE;
	} else {
		$res = FALSE;
	}
} else {
	$res = TRUE;
}
if (! $res) {
	echo 'Please refer to ' . REFER_ON . ' !';
	exit(1);
}

/**
 * PHP5以上かどうか検査する
 * @return	bool TRUE：PHP5以上／FALSE:PHP5未満
*/
function isphp5over() {
	$version = explode('.', phpversion());

	return $version[0] >= 5 ? TRUE : FALSE;
}

/**
 * 無効な証明書サイトからXML取得できるようにする
 * @param	なし
 * @return	なし
*/
function unknown_certificate() {
	$context = array(
		'ssl' => array(
			'verify_peer' => FALSE,
			'verify_peer_name' => FALSE,
		)
	);
	libxml_set_streams_context(stream_context_create($context));
}

/**
 * $TableStationの予報地点を数える
 * @param	なし
 * @return	なし
*/
function countTableStation() {
	global $TableStation;
	$cnt = 0;

	foreach ($TableStation as $pref=>$arr1) {
		foreach ($arr1 as $areaName=>$arr2) {
			foreach ($arr2 as $stationName) {
				$cnt++;
			}
		}
	}
	return $cnt;
}

/**
 * 気象庁防災情報XMLから各地点の最新の府県週間天気予報情報URLを取得
 * @param	object $pwt    pahooWeatherオブジェクト
 * @param	string $code   データコード
 * @param	array  $items  府県週間天気予報情報URLを格納する配列
 *					[page]['url']  URL
 *					[page]['dt']   日時
 * @param	string $errmsg エラーメッセージを格納
 * @return	int 格納したURLの数／FALSE：エラー発生
*/
function getWeatherUrls($pwt, $code, &$items, &$errmsg) {
	//マッチするURLパターン
	$pat1 = sprintf('/http\:\/\/www\.data\.jma\.go\.jp\/developer\/xml\/data\/([0-9\_]+)%s\_([0-9]+)\.xml/ui', $code);

	//気象庁防災情報XML：長期フィード - 定時配信
	$errmsg = '';
	unknown_certificate();
	$xml = @simplexml_load_file($pwt::FEED_REGULAR_L);
	//レスポンス・チェック
	if (! isset($xml->entry)) {
		$errmsg = '気象庁防災情報XMLにアクセスできません';
		return FALSE;
	}

	//フィードを解析
	$items = array();
	$cnt = 0;
	foreach ($xml->entry as $entry) {
		if (preg_match($pat1, (string)$entry->id, $arr) > 0) {
			$dt   = (string)$arr[1];
			$page = (string)$arr[2];
			//登録済み
			if (isset($items[$page])) {
				//より新しければ要素を上書き
				if ($dt > $items[$page]['dt']) {
					$items[$page]['dt']  = (string)$dt;
					$items[$page]['url'] = (string)$entry->id;
				}
			//未登録
			} else {
				$items[$page]['dt']  = (string)$dt;
				$items[$page]['url'] = (string)$entry->id;
				$cnt++;
			}
		}
	}
	//ソート
	ksort($items);

	return $cnt;
}

/**
 * 1つの府県週間天気予報情報から予報地点情報を取得
 * @param	string $url    府県週間天気予報情報URL
 * @param	array  $items  予報地点情報を格納する配列
 * @param	string $errmsg エラーメッセージを格納
 * @return	int 格納した予報地点の数／FALSE：エラー発生
*/
function getWeatherSpotsSub($url, &$items, &$errmsg) {
	$errmsg = '';
	unknown_certificate();
	$xml = @simplexml_load_file($url);
	//レスポンス・チェック
	if (! isset($xml->Body->MeteorologicalInfos)) {
		$errmsg = '府県週間天気予報情報を取得できません';
		return FALSE;
	}

	$flag1 = $flag2 = FALSE;
	foreach ($xml->Body->MeteorologicalInfos as $MeteorologicalInfos) {
		//地方名
		if (!$flag1 && $MeteorologicalInfos['type'] == '区域予報') {
			$flag1 = TRUE;
			$cnt = 0;
			foreach ($MeteorologicalInfos->TimeSeriesInfo->Item as $item) {
				$items[$cnt]['areaName'] = (string)$item->Area->Name;
				$items[$cnt]['areaCode'] = (string)$item->Area->Code;
				$cnt++;
			}
		//予報地点名
		} else if (!$flag2 && $MeteorologicalInfos['type'] == '地点予報') {
			$flag2 = TRUE;
			$cnt = 0;
			foreach ($MeteorologicalInfos->TimeSeriesInfo->Item as $item) {
				$items[$cnt]['stationName'] = (string)$item->Station->Name;
				$items[$cnt]['stationCode'] = (string)$item->Station->Code;
				$cnt++;
			}
		}
	}
	//余分な予報地点を削除
/**
	foreach ($items as $key=>$item) {
		if ($key >= $cnt) {
			unset($items[$key]['stationName']);
			unset($items[$key]['stationCode']);
		}
	}
**/

	return $cnt;
}

/**
 * 地域観測所一覧を配列に格納
 * @param	string $fname  地域観測所一覧ファイル名
 * @param	array  $items  地域観測所一覧を格納する配列
 * @param	string $errmsg エラーメッセージを格納
 * @return	int 格納した観測所の数／FALSE：エラー発生
*/
function getAmedasSpots($fname, &$items, &$errmsg) {
	$errmsg = '';
	$infp = fopen($fname, 'r');
	if ($infp == FALSE) {
		$errmsg = '地域観測所一覧ファイル "' . $fname . '" が見つかりません';
		return FALSE;
	}

	//一覧読み込み
	$cnt = 0;
	while (! feof($infp)) {
		$ss = fgets($infp, 2000);
		$ss = mb_convert_encoding($ss, INTERNAL_ENCODING, 'SJIS');
		$arr = preg_split('/,/ui', $ss);
		if (isset($arr[9]) && is_numeric($arr[9])) {
			$items[$cnt]['areaName']    = (string)$arr[0];
			$items[$cnt]['stationCode'] = (string)$arr[1];
			$items[$cnt]['stationName'] = (string)$arr[3];
			$items[$cnt]['location']    = (string)$arr[5];
			$items[$cnt]['latitude']    = (float)$arr[6] + (float)$arr[7] / 60;
			$items[$cnt]['longitude']   = (float)$arr[8] + (float)$arr[9] / 60;
			$cnt++;
		}
	}
	return $cnt;
}

/**
 * 予報地点情報に地方名を代入：VPFD51用
 * @param	array  $spot  予報地点情報
 * @return	なし
*/
function addAreaName(&$spot) {
	global $TableStation;

	foreach ($TableStation as $pref=>$arr1) {
		foreach ($arr1 as $areaName=>$arr2) {
			foreach ($arr2 as $stationName) {
				if (($spot['prefName'] == $pref) && ($spot['stationName'] == $stationName)) {
					$spot['areaName'] = (string)$areaName;
					return;
				}
			}
		}
	}
}

/**
 * 予報地点情報に場所・緯度・経度を代入：VPFD51用
 * @param	array  $amedas 地域観測所一覧
 * @param	array  $spot   予報地点情報
 * @return	なし
*/
function addLocation($amedas, &$spot) {
	//地域観測所を探索
	foreach ($amedas as $item) {
		if ($spot['stationCode'] == $item['stationCode']) {
			$spot['location']  = $item['location'];
			$spot['latitude']  = $item['latitude'];
			$spot['longitude'] = $item['longitude'];
			return;
		}
	}
}

/**
 * 気象庁防災情報XMLから予報地点情報を取得
 * @param	object $pwt    pahooWeatherオブジェクト
 * @param	array  $amedas 地域観測所一覧
 * @param	array  $spots  予報地点情報を格納する配列
 * @param	string $errmsg エラーメッセージを格納
 * @return	int 格納した予報地点の数／FALSE：エラー発生
*/
function getWeatherSpots($pwt, $amedas, &$spots, &$errmsg) {
	global $TablePref;
	$cnt = 0;

	//各地点の最新の府県週間天気予報情報URLを取得
	$code = 'VPFW50';
	$urlLists = array();
	$errmsg = '';
	getWeatherUrls($pwt, $code, $urlLists, $errmsg);
	if ($errmsg != '')		return FALSE;

	//府県週間天気：予報地点情報を取得
	foreach ($urlLists as $page=>$list) {
		$items = array();
		getWeatherSpotsSub($list['url'], $items, $errmsg);
		if ($errmsg != '')		return FALSE;
		foreach ($items as $item) {
			$cd = (string)substr($page, 0, 2);	//都道府県コード
			$spots[$cnt]['code']        = (string)$code;
			$spots[$cnt]['page']        = (string)$page;
			$spots[$cnt]['regionCode']  = (string)$TablePref[$cd][1];
			$spots[$cnt]['regionName']  = (string)$TablePref[$cd][2];
			$spots[$cnt]['prefCode']    = (string)$cd;
			$spots[$cnt]['prefName']    = (string)$TablePref[$cd][0];
			$spots[$cnt]['areaCode']    = (string)$item['areaCode'];
			$spots[$cnt]['areaName']    = (string)$item['areaName'];
			$spots[$cnt]['stationCode'] = (string)$item['stationCode'];
			$spots[$cnt]['stationName'] = (string)$item['stationName'];
			$spots[$cnt]['delete']      = FALSE;
			addLocation($amedas, $spots[$cnt]);
			$cnt++;
		}
	}

	//各地点の最新の府県天気予報情報URLを取得
	$code = 'VPFD51';
	$urlLists = array();
	$errmsg = '';
	getWeatherUrls($pwt, $code, $urlLists, $errmsg);
	if ($errmsg != '')		return FALSE;

	//府県天気：予報地点情報を取得
	foreach ($urlLists as $page=>$list) {
		$items = array();
		getWeatherSpotsSub($list['url'], $items, $errmsg);
		if ($errmsg != '')		return FALSE;
		foreach ($items as $item) {
			$cd = (string)substr($page, 0, 2);	//都道府県コード
			$spots[$cnt]['code']        = (string)$code;
			$spots[$cnt]['page']        = (string)$page;
			$spots[$cnt]['regionCode']  = (string)$TablePref[$cd][1];
			$spots[$cnt]['regionName']  = (string)$TablePref[$cd][2];
			$spots[$cnt]['prefCode']    = (string)$cd;
			$spots[$cnt]['prefName']    = (string)$TablePref[$cd][0];
			$spots[$cnt]['areaCode']    = (string)'';
			$spots[$cnt]['areaName']    = (string)'';
			$spots[$cnt]['stationCode'] = (string)$item['stationCode'];
			$spots[$cnt]['stationName'] = (string)$item['stationName'];
			$spots[$cnt]['delete']      = FALSE;
			addAreaName($spots[$cnt]);
			//areaNameからareaCodeを逆引きする
			foreach ($items as $item2) {
				$pat = '/' . $item2['areaName'] . '/ui';
				if (preg_match($pat, $spots[$cnt]['areaName']) > 0) {
					$spots[$cnt]['areaCode'] = $item2['areaCode'];
					break;
				}
			}
			addLocation($amedas, $spots[$cnt]);
			$cnt++;
		}
	}

	//エラー・チェック
	if ($cnt == 0) {
		$errmsg = '気象庁防災情報XMLから週間天気予報情報を取得できません';
		return FALSE;
	}

	//重複情報に削除フラグを立てる
	for ($i = 0; $i < $cnt; $i++) {
		for ($j = $i + 1; $j < $cnt; $j++) {
			if (($spots[$i]['code'] == $spots[$j]['code']) && ($spots[$i]['stationCode'] == $spots[$j]['stationCode'])) {
				$spots[$j]['delete'] = TRUE;
			}
		}
	}

	return $cnt;
}

/**
 * UNIXタイムスタンプからRFC3339 UTC タイムスタンプを返す
 * @param	int $unix UNIXタイムスタンプ，Unix epoch(1970年1月1日 00:00:00 GMT))からの通算秒
 * @return	RFC3339 UTC
*/
function unix2utc($unix) {
	return (gmdate("Y-m-d", $unix) . "T" . gmdate("H:i:s", $unix) . "Z");
}

/**
 * RFC3339 UTC タイムスタンプをローカルのタイムスタンプに変換する
 * @param	string $utc RFC3339 UTCタイムスタンプ
 * @param	int tz_hour UTCと地方時の差（時間）
 * @param	int tz_min UTCと地方時の差（分）（省略可能）
 * @return	string ローカルのタイムスタンプ(RF3339表記)
 *                 NULL=エラー（入力文字列が規定外など）
*/
function utc2local($utc, $tz_hour) {
	$tz_hour = func_get_arg(1);
	//可変長変数を使う
	$tz_min = (@func_num_args() >= 3) ? func_get_arg(2) : (0);

	if (preg_match("/[0-9]+-[0-9]+-[0-9]+T[0-9]+:[0-9]+:[0-9]+Z/", $utc) == 1) {
		list($year, $month, $day, $hour, $minuite, $second) = sscanf($utc, "%d-%d-%dT%d:%d:%dZ");
	} else if (preg_match("/[0-9]+-[0-9]+-[0-9]+T[0-9]+:[0-9]+Z/", $utc) == 1) {
		list($year, $month, $day, $hour, $minuite) = sscanf($utc, "%d-%d-%dT%d:%dZ");
		$second = 0;
	} else if (preg_match("/[0-9]+-[0-9]+-[0-9]+T[0-9]+Z/", $utc) == 1) {
		list($year, $month, $day, $hour, $minuite) = sscanf($utc, "%d-%d-%dT%dZ");
		$minuite = 0;	$second = 0;
	} else if (preg_match("/[0-9]+-[0-9]+-[0-9]+Z/", $utc) == 1) {
		list($year, $month, $day, $hour, $minuite) = sscanf($utc, "%d-%d-%dZ");
		$hour = 0;	$minuite = 0;	$second = 0;
	} else {
		return NULL;	//UTC書式エラー
	}

	$tt = mktime($hour + $tz_hour, $minuite + $tz_min, $second, $month, $day, $year);

	$s = date("Y-m-d", $tt) . "T" . date("H:i:s", $tt);
	$f = ($tz_hour < 0) ? "-" : "+";

	return sprintf("%s%s%02d:%02d", $s, $f, abs($tz_hour), $tz_min);
}

/**
 * データファイル作成
 * @param	object $pwt    pahooWeatherオブジェクト
 * @param	array  $items  予報地点情報
 * @param	string $fname  出力ファイル名
 * @param	string $errmsg エラーメッセージを格納
 * @return	bool TRUE/FALSE
*/
function putXmlFile($pwt, $items, $fname, &$errmsg) {
	//XMLのルートタグとXML宣言
	$root = '<?xml version="1.0" encoding="UTF-8" ?><jmaweatherspots></jmaweatherspots>';
	//XML作成開始
	$xml = new SimpleXMLElement($root);

	//作成時刻
	$utc = unix2utc(time());			//内蔵時計の日時をUTCに変換する
	$local = utc2local($utc, 9);		//JST = UTC + 09:00
	$date  = $xml->addChild('date', $local);

	//バージョン
	$date  = $xml->addChild('version', $pwt::FILE_VERSION);

	//地点情報
	foreach($items as $item) {
		if ($item['delete'] == FALSE) {
			$spot = $xml->addChild('spot');
			foreach ($item as $key=>$val) {
				if ($key != 'delete') {
					$spot->addChild($key, $val);
				}
			}
		}
	}

	if (($outfp = @fopen($fname, 'w')) != FALSE) {
		//XMLを整形出力
		$dom = new DOMDocument('1.0');
		$dom->loadXML($xml->asXML() );
		$dom->formatOutput = TRUE;
		$res = @fwrite($outfp, $dom->saveXML()) == FALSE ? FALSE : TRUE;
		fclose($outfp);
		$dom = NULL;
	} else {
		$errmsg = $fname . ' の書き込みに失敗しました';
		$res = FALSE;
	}

	return $res;
}

/**
 * HTML BODYを作成する
 * @param	object $pwt pahooWeatherオブジェクト
 * @return	string HTML BODY
*/
function makeCommonBody($pwt) {
	//オブジェクト生成
	$pwt = new pahooWeather();

	$title   = TITLE;
	$refere  = REFERENCE;
	$myself  = MYSELF;
	$version = '<span style="font-size:small;">' . date('Y/m/d版', filemtime(__FILE__)) . '</span>';
	$width   = WIDTH;

	if (FLAG_RELEASE) {
		$msg = '';
	} else {
		$phpver = phpversion();
		$msg =<<< EOT
PHPver : {$phpver}<br />
<dl>

EOT;
	}

	//府県天気予報地点の数
	$cts = countTableStation();

	//HTML本体
	$body =<<< EOT
<body>
<h2>{$title} {$version}</h2>

<div style="border-style:solid; border-width:1px; margin:20px 0px 0px 0px; padding:5px; width:{$width}px; font-size:small; overflow-wrap:break-word; word-break:break-all;">
※参考サイト：<a href="{$refere}">{$refere}</a>
<p>{$msg}</p>
</div>

<p>TableStation地点数：{$cts}</p>

<table>
<tr class="index">
<th>No.</th>
<th>コード</th>
<th>予報地点</th>
<th>緯度</th>
<th>経度</th>
</tr>

EOT;

	//地域観測所一覧の取得
	$errmsg = '';
	$amedas = array();
	getAmedasSpots(FILE_AMEDAS, $amedas, $errmsg);

	//予報地点情報の取得
	if ($errmsg == '') {
		$fname = $pwt::FILE_JMASPOTS;
		$spots = array();
		getWeatherSpots($pwt, $amedas, $spots, $errmsg);
		if ($errmsg == '') {
			$cnt = 1;
			foreach ($spots as $key=>$spot) {
				if ($spot['delete'] == FALSE) {
					$body .=<<< EOT
<tr>
<td>{$cnt}</td>
<td>{$spot['code']}</td>
<td>{$spot['stationName']}</td>
<td>{$spot['latitude']}</td>
<td>{$spot['longitude']}</td>
</tr>

EOT;
					$cnt++;
				}
			}
			if ($errmsg == '') {
				putXmlFile($pwt, $spots, $fname, $errmsg);
			}
		}
	}
	$body .=<<< EOT
</table>

EOT;

	//エラー
	if ($errmsg != '') {
		$body .=<<< EOT
<p style="color:red;">
エラー：{$errmsg}．
</p>

EOT;
	} else {
		$body .=<<< EOT
<p style="color:blue;">
データファイル "<span style="font-weight:bold;">{$fname}</span>" の作成に成功しました．
</p>

EOT;
	}
	$body .=<<< EOT
</body>

EOT;
	return $body;
}
