---
description: '[Doctrine Migrations]数据库迁移组件的深入解析四：集成diff方式迁移组件'
---

# 数据迁移组件四：集成diff方式迁移组件

## 场景及优势

熟悉Symfony框架之后，深刻感受到框架集成的ORM组件Doctrine2的强大之处，其中附带的数据迁移也十分方便。Doctrine2是使用Doctrine DBAL组件把代码里面的表结构与实际数据库中的表结构进行对比的方式进行数据迁移。这种方式比之前版本管理的方式更加精准也更方便。

Symfony框架是自身ORM组件支持，但是很多项目并没有使用其中的ORM功能，或者有自己的ORM组件，又该如何集成diff方式的迁移呢？下面我们就来完成这个任务。

## 源码解析

在研究组件源码时，发现一个类：`Doctrine\DBAL\Migrations\Provider\SchemaDiffProvider`，其中`getSqlDiffToMigrate`方法就是我们编写diff脚本的关键。

```php
/**
 * @param Schema $fromSchema
 * @param Schema $toSchema
 * @return string[]
 */
public function getSqlDiffToMigrate(Schema $fromSchema, Schema $toSchema)
{
    return $fromSchema->getMigrateToSql($toSchema, $this->platform);
}
```

该方法通过传入的两个入参（fromSchema和toSchema），来对比旧数据结构和新数据结构的差异，最后返回相关的sql语句。那么我们只要拿到现有数据库中的表结构，以及代码里面的表结构，就能实现diff方式的数据迁移了。

获取现有数据库的表结构可以通过上面的`createFromSchema`方法直接获取。

然后我们需要把数据结构写入代码，再从代码中获取实时的表结构，最后diff。

## 编写diff脚本

首先在项目创建diff文件，并赋予执行权限。之后是根据上面的思路来编写脚本，以下是完整代码示例：

```php
#!/usr/bin/env php
<?php
require_once 'vendor/autoload.php';

use Doctrine\DBAL\DBALException;
use Doctrine\DBAL\DriverManager;
use Doctrine\DBAL\Migrations\Provider\SchemaDiffProvider;
use Doctrine\DBAL\Schema\MySqlSchemaManager;
use Doctrine\DBAL\Schema\Schema;

// 读取数据库配置信息
$db_config = include 'config/db.php';
$db_params = [
    'driver' => 'pdo_mysql',
    'host' => $db_config['host'],
    'port' => $db_config['port'],
    'dbname' => $db_config['dbname'],
    'user' => $db_config['user'],
    'password' => $db_config['password'],
];
try {
    $connection = DriverManager::getConnection($db_params);
} catch (DBALException $e) {
    echo $e->getMessage() . PHP_EOL;
    exit;
}

// 获取数据库表结构
$schema_manager = new MySqlSchemaManager($connection);
$sdp = new SchemaDiffProvider($schema_manager, $schema_manager->getDatabasePlatform());
$from_schema = $sdp->createFromSchema();

// 获取写在脚本里面的表结构
$to_schema = new Schema();
$schema_class_list = glob(__DIR__ . '/db/tables/*.php');
foreach ($schema_class_list as $file) {
    require_once $file;
    $class = '\\db\\tables\\' . basename($file, '.php');
    if (class_exists($class)) {
        $cls = new $class;
        if (method_exists($cls, 'up')) {
            $cls->up($to_schema);
        }
    }
}

// 进行对比，输出结果
$diff = $sdp->getSqlDiffToMigrate($from_schema, $to_schema);
if ($diff) {
    foreach ($diff as $sql) {
        echo $sql . ';' . PHP_EOL;
    }
}
```

## diff方式的迁移使用

之后，所有的数据表都需要在根目录下db/tables目录里面建立对应的表结构脚本（任意文件名本），下面，我们以test\_user表举例：

创建db/tables/testUser.php文件，编写代码：

```php
<?php
namespace db\tables;

use Doctrine\DBAL\Schema\Schema;

class testUser
{
    public function up(Schema $schema): void
    {
        $table = $schema->createTable('test_user');
        $table->addColumn('id', 'integer')->setUnsigned(true)->setAutoincrement(true);
        $table->addColumn('name', 'string')->setLength(20)->setComment('用户名');
        $table->addColumn('age', 'integer')->setUnsigned(true)->setDefault(0)->setComment('年龄');
        $table->addColumn('sex', 'string')->setLength(2)->setDefault('')->setComment('性别');
        $table->addColumn('is_del', 'boolean')->setDefault(false)->setComment('是否删除');
        $table->setPrimaryKey(['id'])->addIndex(['is_del']);
    }
}
```

然后在根目录执行`./diff`命令，就会看到输出的sql信息。

## 添加黑名单过滤

到这一步diff方式的迁移已经能正常使用了，但是我们发现，如果是在已有的项目中途加入迁移功能时，使用diff命令会把已经存在的表删除掉，另外，也有时候会有其他表不需要管理的，我们就需要过滤掉这些表。现在我们来添加黑名单功能。

在进行对比之前，加入以下代码：

```php
// 过滤黑名单
$black_list = ['migration_versions', 'test1', 'test2', 'test3'];
foreach ($black_list as $black_table) {
    $from_schema->hasTable($black_table) && $from_schema->dropTable($black_table);
    $to_schema->hasTable($black_table) && $to_schema->dropTable($black_table);
}
```

再执行diff命令时，就会过滤掉`$black_list`变量数组里面的表，当然这个黑名单变量完全可以用其他配置文件来代替，可以任意方式的自定义。

## 脚本增强

最好再对diff脚本进一步的增强，让它可以直接执行sql。

修改最后的对比代码为以下内容：

```php
// 进行对比，输出结果
$diff = $sdp->getSqlDiffToMigrate($from_schema, $to_schema);
if (empty($diff)) {
    echo 'No need to update the schema.' . PHP_EOL;
} elseif ($diff) {
    $statement = '';
    foreach ($diff as $sql) {
        $statement .= $sql;
        echo $sql . ';' . PHP_EOL;
    }
    echo 'Do you want to execute these SQLs? (Y/n)';
    $flag = trim(fgets(STDIN));
    if ($flag === 'Y') {
        $connection->beginTransaction();
        try {
            $connection->executeUpdate($statement);
            $connection->commit();
        } catch (\Exception $e) {
            try {
                $connection->rollBack();
            } catch (ConnectionException $e) {
                echo $e->getMessage();
                exit;
            }
            echo $e->getMessage() . PHP_EOL;
            exit;
        }
        echo 'SQL executed successfully' . PHP_EOL;
    }
}
```

现在，执行diff命令时就会提示是否需要执行sql（默认不执行）。

## 结语

现在，diff方式的数据迁移就已经比较完美的集成与项目中了，但是diff方式也有缺陷，就是不能对数据进行管理，比如有时候菜单或者或者权限就需要数据也能同步。所以两种方式要根据自己的项目情况去选择。

此系列的所有代码都可以在文章最后的代码库链接中找到对应的代码，并且每次commit对应每篇文章，方便读者对应文章和代码。

现在此系列就已经完结，希望这个系列的文章能对你使用数据迁移组件有所帮助，感谢您的阅读，也希望提出您宝贵的意见。



{% hint style="info" %}
在[我的代码库](https://github.com/tbphp/studyMigrations/tree/b3b2d565b66eaf25fdfcf1cc1293f39bd0c249d4)可以查看这篇文章的详细代码，欢迎star。
{% endhint %}

