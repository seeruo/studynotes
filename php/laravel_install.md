# [Laravel] 安装与部署



## Composer安装

```bash
composer create-project --prefer-dist laravel/laravel blog
```



## 配置站点

配置nginx站点，指向`/public/index.php`文件。

配置成功，则可以访问站点看到laravel欢迎页面。

nginx站点配置需要添加以下代码来隐藏index.php文件：

```conf
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```



## 配置文件

可以参考config目录下的配置修改。建议直接修改.env文件。

修改初始化配置：数据库、缓存、会话控制。

.env文件为环境配置文件，也可以创建一个.env.testing文件，以便phpunit使用。

使用env('ENV_NAME', 'default')方法获取.env文件配置的值，但是这个方法仅限于配置文件中使用。

判断环境：

```php
if (App::environment(['local', 'staging'])) {
    // 当前是local和staging环境
}
```



## 停机维护

```bash
# 停机
php artisan down
# 开机
php artisan up
```



## 生产部署

1. 优化自动加载：

   ```bash
   composer install --optimize-autoloader
   ```

2. 优化配置缓存：

   ```bash
   php artisan config:cache
   ```

   使用此命令类使配置文件缓存为一个文件，以加快速度。

3. 优化路由加载

   ```bash
   php artisan route:cache
   ```

   ​