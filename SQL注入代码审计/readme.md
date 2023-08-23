## 0x00 SQL注入代码审计1

这是一套比较简单的代码，存好SQL注入的函数很多，比如，增删改查，执行函数等。找到这些函数后，看看有没有引号闭合和过滤，看看能不能绕过，下面是2典型的记录一下

Seay工具审出可能存在SQL注入漏洞位置如下：

![图片](https://github.com/Londly01/audit-hub/assets/118274389/3affb98e-609d-4922-8cdf-78b9934335f8)

源码如下：

```
<?php 
include('system/inc.php');//载入全局配置文件
error_reporting(0);//关闭错误报告
$result = mysql_query('select * from xtcms_vod where d_id = '.$_GET['play'].' ');
if (!!$row = mysql_fetch_array($result)) {
	$d_id = $row['d_id'];
	$d_name = $row['d_name'];
	$d_jifen = $row['d_jifen'];
	$d_user = $row['d_user'];
	$d_parent = $row['d_parent'];
	$d_picture = $row['d_picture'];
	$d_content = $row['d_content'];
	$d_scontent = $row['d_scontent'];
	$d_seoname = $row['d_seoname'];
	$d_keywords = $row['d_keywords'];
	$d_description = $row['d_description'];
	$d_player = $row['d_player'];
	$d_title = ($d_seoname == '') ? $d_name .' - '.$xtcms_name : $d_seoname.' - '.$d_name.' - '.$xtcms_name ;
} else {
	die ('您访问的详情不存在');
}
$result1 = mysql_query('select * from xtcms_vod_class where c_id='.$d_parent.' order by c_id asc');
while ($row1 = mysql_fetch_array($result1)){
$c_hide=$row1['c_hide'];
}
if($c_hide>0){
if(!isset($_SESSION['user_name'])){
		alert_href('请注册会员登录后观看',''.$xtcms_domain.'ucenter');
	};
    $result = mysql_query('select * from xtcms_user where u_name="'.$_SESSION['user_name'].'"');//查询会员积分
     if($row = mysql_fetch_array($result)){
	 $u_group=$row['u_group'];//到期时间
     }
 if($u_group<=1){//如果会员组
 alert_href('对不起,您不能观看会员视频，请升级会员！',''.$xtcms_domain.'ucenter/mingxi.php');
 } 
}
include('system/shoufei.php');
if($d_jifen>0){//积分大于0,普通会员收费
	if(!isset($_SESSION['user_name'])){
		alert_href('请注册会员登录后观看',''.$xtcms_domain.'ucenter');
	};
    $result = mysql_query('select * from xtcms_user where u_name="'.$_SESSION['user_name'].'"');//查询会员积分
     if($row = mysql_fetch_array($result)){
     $u_points=$row['u_points'];//会员积分
     $u_plays=$row['u_plays'];//会员观看记录
     $u_end=$row['u_end'];//到期时间
	 $u_group=$row['u_group'];//到期时间
     }	

	     if($u_group<=1){//如果会员组
     if($d_jifen>$u_points){
	 alert_href('对不起,您的积分不够，无法观看收费数据，请推荐本站给您的好友、赚取更多积分',''.$xtcms_domain.'ucenter/yaoqing.php');
    }  else{

    if (strpos(",".$u_plays,$d_id) > 0){ 

	}	
	//有观看记录不扣点
else{

   $uplays = ",".$u_plays.$d_id;
   $uplays = str_replace(",,",",",$uplays);
   $_data['u_points'] =$u_points-$d_jifen;
   $_data['u_plays'] =$uplays;
   $sql = 'update xtcms_user set '.arrtoupdate($_data).' where u_name="'.$_SESSION['user_name'].'"';
if (mysql_query($sql)) {

alert_href('您成功支付'.$d_jifen.'积分,请重新打开视频观看!',''.$xtcms_domain.'bplay.php?play='.$d_id.'');
}
}
	
}
}
}
if($d_user>0){	
if(!isset($_SESSION['user_name'])){
		alert_href('请注册会员登录后观看',''.$xtcms_domain.'ucenter');
	};
    $result = mysql_query('select * from xtcms_user where u_name="'.$_SESSION['user_name'].'"');//查询会员积分
     if($row = mysql_fetch_array($result)){
     $u_points=$row['u_points'];//会员积分
     $u_plays=$row['u_plays'];//会员观看记录
     $u_end=$row['u_end'];//到期时间
	 $u_group=$row['u_group'];//到期时间
     }		 
if($u_group<$d_user){
	alert_href('您的会员组不支持观看此视频!',''.$xtcms_domain.'ucenter/mingxi.php');
}
}
function get_play($t0){
	$result = mysql_query('select * from xtcms_player where id ='.$t0.'');
	if (!!$row = mysql_fetch_array($result)){
return $row['n_url'];
	}else{
		return $t0;
	};
}
$result = mysql_query('select * from xtcms_vod where d_id ='.$d_id.'');
	if (!!$row = mysql_fetch_array($result)){
$d_scontent=explode("\r\n",$row['d_scontent']);
//print_r($d_scontent);
for($i=0;$i<count($d_scontent);$i++)
{	$d_scontent[$i]=explode('$',$d_scontent[$i]);
		}
$playdizhi=get_play($row['d_player']).$d_scontent[0][1];
	}else{
		return '';
	};
	
include('template/'.$xtcms_bdyun.'/bplay.php');
?>

```
关键语句报的SQL注入为：
```
$result = mysql_query('select * from xtcms_vod where d_id = '.$_GET['play'].' ');

```
这个GET传入的play参数是没有单引号或者双引号包裹的，肯定存在SQL注入漏洞，所以Seay工具报了这个错误，此时我们在跟进一下全局文件中include的文件,也就是system/inc.php，查看一下这个文件

```
<?php
require_once('conn.php');
require_once('library.php');
require_once('function.php');
require_once('config.php');
?>

```
查看library.php文件
```
if (!defined('PCFINAL')) {
	exit('Request Error!');
}
if (!get_magic_quotes_gpc()) {
	if (!empty($_GET)) {
		$_GET = addslashes_deep($_GET);
	}
	if (!empty($_POST)) {
		$_POST = addslashes_deep($_POST);
	}
	$_COOKIE = addslashes_deep($_COOKIE);
	$_REQUEST = addslashes_deep($_REQUEST);
}
function addslashes_deep($_var_0)
{
	if (empty($_var_0)) {
		return $_var_0;
	} else {
		return is_array($_var_0) ? array_map('addslashes_deep', $_var_0) : addslashes($_var_0);
	}
```

magic_quotes_gpc函数在php中的做用是判断解析用户提示的数据所以传入post、get、cookie过来的数据增长转义字符“\”，但是现在做这个已经没有意义了，此处还是存在SQL注入漏洞

## 0x00 SQL注入代码审计2

     seay审计出其他路径存在SQL注入漏洞，看一下代码

     <?php include('../system/inc.php');
error_reporting(0);
$op=$_GET['op'];
if(!isset($_SESSION['user_name'])){
  alert_href('请登陆后进入','login.php?op=login');
 };
 //退出
if ($op == 'out'){ 
unset($_SESSION['user_name']);
unset($_SESSION['user_group']);
if (! empty ( $_COOKIE ['user_name'] ) || ! empty ( $_COOKIE ['user_password'] ))   
    {  
        setcookie ( "user_name", null, time () - 3600 * 24 * 365 );  
        setcookie ( "user_password", null, time () - 3600 * 24 * 365 );  
    }  
header('location:login.php?op=login');
}
//支付
if ( isset($_POST['paysave']) ) {
if ($_POST['pay']==1){

//判定会员组别
$result = mysql_query('select * from xtcms_user where u_name="'.$_SESSION['user_name'].'"');
if($row = mysql_fetch_array($result)){

$u_points=$row['u_points'];
$u_group=$row['u_group'];
$send = $row['u_end'];

//获取会员卡信息
$card= mysql_query('select * from xtcms_userka where id="'.$_POST['cardid'].'"');
if($row2 = mysql_fetch_array($card)){
$day=$row2['day'];//天数
$userid=$row2['userid'];//会员组
$jifen=$row2['jifen'];//积分
}
//判定会员组
if ($row['u_group']>$userid){ 
alert_href('您现在所属会员组的权限制大于等于目标会员组权限值，不需要升级!','mingxi.php');
}


此处存在SQL注入点可能为  

```
$result = mysql_query('select * from xtcms_user where u_name="'.$_SESSION['user_name'].'"');
```

传入的参数进行了""闭合，看看通过闭合双引号进行绕过限制，继续跟全局过滤代码：

```
if (!get_magic_quotes_gpc()) {
	if (!empty($_GET)) {
		$_GET = addslashes_deep($_GET);
	}
	if (!empty($_POST)) {
		$_POST = addslashes_deep($_POST);
	}
	$_COOKIE = addslashes_deep($_COOKIE);
	$_REQUEST = addslashes_deep($_REQUEST);
}
function addslashes_deep($_var_0)
{
	if (empty($_var_0)) {
		return $_var_0;
	} else {
		return is_array($_var_0) ? array_map('addslashes_deep', $_var_0) : addslashes($_var_0);
```

其中addslashes_deep负责对用户提交的数据进行转义(如',",\)，所以没法闭合双引号，此处不存在SQL注入
