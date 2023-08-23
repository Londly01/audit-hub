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
