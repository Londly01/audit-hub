

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
