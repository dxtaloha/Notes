在vscode远程连接服务器时，如何让vscode获得root权限

1、在 /etc/ssh/sshd_config 文件中添加以下行：

PermitRootLogin yes

2、重新启动 SSH 服务，使更改生效：

systemctl restart ssh

3、在vscode登录信息中以 root 用户身份进行登录