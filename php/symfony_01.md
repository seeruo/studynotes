# Symfony的安装与配置



`symfony`安装器

```bash
sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
```

创建项目

```bash
composer create-project symfony/framework-standard-edition project-name
```

nginx配置

```nginx
server {
    listen      80;
    server_name symfony.com;
    root        /www/studycodes/Symfony/web;

    location / {
        # 正式环境app_dev.php改为app.php
        try_files $uri /app_dev.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_pass    127.0.0.1:9072;
        fastcgi_index   index.php;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include         fastcgi_params;
    }
}
```

