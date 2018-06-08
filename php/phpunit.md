# PHPUnit

## PHPUnit

### 安装

全局安装：

```bash
brew install phpunit
```

项目安装：

```bash
composer require --dev phpunit/phpunit
```

推荐项目安装，这样可以使用composer的自动加载。

#### phpstorm配置

确保是项目安装，配置composer.json的自动加载，添加需要测试的命名空间：

```javascript
{
    "autoload": {
        "psr-4": {
          "App\\": "App"
        }
      }
}
```

php-&gt;Test Frameworks 添加phpunit。

php-&gt;设置phpunit的composer自动加载路径。

如果是全局安装，需要指定自动加载的php文件。

### 编写

```php
class UserTest extends TestCase
{
    public function testInfo()
    {
        $info = 1;
        self::assertEquals($info, 1);
        return $info;
    }
    /**
     * @depends testInfo
     */
    public function testGetInfo($info)
    {
        self::assertEquals((new User())->getInfo($info), 1);
    }
}
```

@depends 为声明依赖，依赖申明传递的变量为引用传递，如果需要副本而非引用，则使用 @depends clone

### 数据供给器

使用`@dataProvider`声明数据供给方法，批量提交测试数据，例如：

```php
class UserTest extends TestCase
{

    /**
     * @dataProvider testData
     */
    public function testInfo($a, $b, $c)
    {
        self::assertEquals($c, $a + $b);
    }

    public function testGetInfo($info)
    {
        return [
            [1, 2, 3],
            [0, 0, 0],
            [-1, 1, 0],
        ];
    }
}
```

## 钩子

1. `setUp`：重写此方法可以在每次执行测试之前运行，常用来初始化数据，比如初始化公共对象、初始化数据库连接、初始化文件handle等，这样一次操作，避免多次初始化。
2. `tearDown`：重写此方法可以在每次测试完毕之后运行，常用来关闭资源，比如mysql、文件、socket等。

例：

```php
class UserTest extends TestCase
{

    private $obj;

    protected function setUp()
    {
        $this->obj = new User;
    }

    /**
     * @dataProvider testData
     */
    public function testAge($a, $b, $c)
    {
        self::assertEquals($this->obj->age(1), 1);
    }
}
```

> 注意：一个类有多个测试方法时，避免每次初始化需要测试的对象，所以就会用到setUp来进行公共的初始化。但是在测试类里面禁止重写构造方法，这会导致测试结果出错！

### 测试异常

如果判断测试案例会抛出异常，则使用异常测试：

```php
class ObjTest extends TestCase
{
    public function testObj()
    {
        // 此处是在断言后面的代码会抛出异常，且异常为ClassNotFoundException
        $this->expectException(ClassNotFoundException::class);
        new Abc();
    }
}
```

> 注意：异常断言需要在代码之前。

