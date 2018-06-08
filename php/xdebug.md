# Xdebug

## 安装

```bash
brew install php71-xdebug
```

## 配置

配置php.ini文件：

```text
[xdebug]
xdebug.idekey=PHPSTORM
xdebug.profiler_append=0
xdebug.profiler_enable=1
xdebug.profiler_enable_trigger=0
xdebug.remote_autostart=no
xdebug.remote_enable=1
xdebug.remote_host=127.0.0.1
xdebug.remote_port=9000
```

## PhpStorm配置

