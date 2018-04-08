# 协程三：协程的实现原理



## 接收

上面说到yield的生成器中只说了返回值的使用方式，yield除了返回之外还可以接收值：

```php
function run(): Generator
{
    while (true) {
        echo yield . '.' . PHP_EOL;
    }
}

$run = run();
$run->send('tb');
$run->send('zx');
$run->send('tx');
$run->send('zz');
```



## 同时进行接收和返回

```php
function run(): Generator
{
    $i = 0;
    while (true) {
        sleep(2);
        echo (yield $i++) . '.' . PHP_EOL;
    }
}

$run = run();
$run->send('tb');
echo '第：' . $run->current() . '次' . PHP_EOL;
$run->send('zx');
echo '第：' . $run->current() . '次' . PHP_EOL;
$run->send('tx');
echo '第：' . $run->current() . '次' . PHP_EOL;
$run->send('zz');
echo '第：' . $run->current() . '次' . PHP_EOL;
```

双向通行，就是协程的基础。

