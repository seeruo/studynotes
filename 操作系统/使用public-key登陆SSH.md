# 使用public-key登陆SSH



- 打开配置文件：`/etc/ssh/sshd_config`
- 修改配置项：

```bash
#允许root用户使用ssh登陆
PermitRootLogin yes
#允许使用RSA算法验证
RSAAuthentication yes
#允许使用公共key
PubkeyAuthentication yes
#授权的公共key配置文件
AuthorizedKeysFile      .ssh/authorized_keys
```

- 重启sshd服务

```bash
service sshd restart
```

- 把允许登陆的公共key写入`~/.ssh/authorized_keys`文件里，一行一个。

> 如果登陆失败，检查`SELinux`和防火墙22端口。

