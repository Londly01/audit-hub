## 0x00 验证码复用代码审计

某次审代码审的第一个小漏洞思路如下：

登录框代码如下：

```
<?php
require_once('../system/inc.php');
if(isset($_POST['submit'])){
    if ($_SESSION['verifycode'] != $_POST['verifycode']) {
        alert_href('验证码错误','cms_login.php');
    }
    null_back($_POST['a_name'],'请输入用户名');
    null_back($_POST['a_password'],'请输入密码');
    null_back($_POST['verifycode'],'请输入验证码');
    $a_name = $_POST['a_name'];
    $a_password = $_POST['a_password'];
    $sql = 'select * from xtcms_manager where m_name = "'.$a_name.'" and m_password = "'.md5($a_password).'"';
    $result = mysql_query($sql);  
    if(!! $row = mysql_fetch_array($result)){  //获取第一行数据
        setcookie('admin_name',$row['m_name']); 
        setcookie('admin_password',$row['m_password']);
        header('location:cms_welcome.php');
    }else{
        alert_href('用户名或密码错误','cms_login.php');
    }
}
```
看看怎么校验验证码的：

```
 if ($_SESSION['verifycode'] != $_POST['verifycode']) {
        alert_href('验证码错误','cms_login.php');
    }
```

校验方式是上传的POST数据和verifycode进行比较，不等于就跳转，提示验证码错误。

搭建环境以后进行登录，查看到前端JS验证码的位置如下：

![图片](https://github.com/Londly01/audit-hub/assets/118274389/d12c2892-b76c-4b2e-9a18-ee824df70371)


/system/verifycode.php

对这个路径进行代码审计

```
<?php
session_start();
$image = imagecreate(50, 34);
$bcolor = imagecolorallocate($image, 0, 0, 0);
$fcolor = imagecolorallocate($image, 255, 255, 255);
$str = '0123456789';
$rand_str = '';
for ($i = 0; $i < 4; $i++){
	$k = mt_rand(1, strlen($str));
	$rand_str .= $str[$k - 1];
}
$_SESSION['verifycode'] = $rand_str;
imagefill($image, 0, 0, $bcolor);
imagestring($image, 7, 7, 10, $rand_str, $fcolor);
header('content-type:image/png');
imagepng($image);
?>

```

这段代码意思是用0-9中的任意四个数字作为验证码，也就是说js引用该文件来产生验证码。
绕过思路就是Burp抓包对用户名和密码进行爆破，因为burp不解析JS文件，绕过前端校验

