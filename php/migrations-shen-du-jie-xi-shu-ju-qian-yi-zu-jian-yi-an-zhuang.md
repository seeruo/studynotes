---
description: '[Doctrine Migrations]数据库迁移组件的深入解析一：安装与使用'
---

# 数据迁移组件一：安装与使用

## 场景分析

团队开发中，每个开发人员对于数据库都修改都必须手动记录，上线时需要人工整理，运维成本极高。而且在多个开发者之间数据结构同步也是很大的问题。`Doctrine Migrations`组件把数据库变更加入到代码中和代码一起进行版本管理，很好的解决了上述问题。

`Doctrine Migrations`是基于`Doctrine DBAL`组件的数据迁移组件。集成于Laravel，Symfony等主流框架。大概可以分为两种方式进行迁移，即版本管理方式和diff方式。

**版本管理**：把数据库变更写入到代码中，来进行版本管理。Laravel框架就是版本管理的模式，迁移组件默认的命令行就是支持这种模式。

**diff**：把现有数据库结构和代码里面的数据库结构来做对比，执行差异的sql以保证一致性。一般需要ORM的支持，Symfony框架就是使用`Doctrine2`ORM工具加上`Doctrine DBAL`来进行diff方式的数据迁移。

此系列文章不讨论现有框架中数据迁移组件的使用，而是着重于探讨如何单独使用迁移组件以及如何把数据迁移组件**集成到自己的项目、个性化定制**。

## 安装

composer安装

```bash
composer require doctrine/migrations ~1.8.0
```

{% hint style="info" %}
本系列使用的是目前最新的1.8.1版本
{% endhint %}

## 配置

安装之后不能直接使用，还需要进行组件配置以及数据库配置：

1. 根目录下建立migrations.php文件，配置组件：

   ```php
   return [
       'name' => 'Doctrine Migrations', // 组件显示名称
       'migrations_namespace' => 'db\migrations', // 迁移类的命名空间
       'table_name' => 'migration_versions', // 迁移组件的表名
       'migrations_directory' => 'db/migrations', // 迁移类的文件夹
   ];
   ```

   详情的配置参数参见[官方文档](https://www.doctrine-project.org/projects/doctrine-migrations/en/latest/reference/configuration.html#migrations-configuration)。 

2. 根目录下简历migrations-db.php文件，配置数据库信息：

   ```php
   return [
       'driver' => 'pdo_mysql',
       'host' => 'localhost',
       'port' => 3306,
       'user' => 'root',
       'password' => '1236',
       'dbname' => 'migrations',
   ];
   ```

到此，组件需要的配置已经完成。

## 使用

1. 运行vendor目录下面的命令生成版本迁移类文件。

   ```bash
   ./vendor/bin/doctrine-migrations migrations:generate
   ```

   在迁移类目录/db/migrations下面生成Version20180608155932.php类文件，此文件即是用于写入迁移sql的类，Version后面的数字则是当前版本号。

2. 重写up方法，用于执行迁移sql：

   ```php
   public function up(Schema $schema): void
   {
       $table = $schema->createTable('test1');
       $table->addColumn('id', 'integer')->setUnsigned(true)->setAutoincrement(true);
       $table->addColumn('name', 'string')->setDefault('')->setLength(20);
       $table->setPrimaryKey(['id']);
   } 
   ```

   这个脚本是生成一个test1的表。

3. 重写down方法，用于版本回退，撤销up方法的迁移操作：

   ```php
   public function down(Schema $schema): void
   {
       if ($schema->hasTable('test1')) {
           $schema->dropTable('test1');
       }
   }
   ```

4. 执行迁移命令：

   ```bash
   ./vendor/bin/doctrine-migrations migrations:migrate
   ```

这样就成功的创建了一个版本的迁移数据。迁移类脚本可以使用$schema来操作数据库实体，也可以使用$this-&gt;addSql\(\)方法来直接写入相关的sql，详细参加官方文档对于脚本编写的[详细说明](https://www.doctrine-project.org/projects/doctrine-migrations/en/1.7/reference/migration_classes.html)，此处不过多展开。

## 版本管理

既然迁移使用版本管理，那么多个版本之间可以来回切换。下面是相关的版本切换命令：

```bash
./vendor/bin/doctrine-migrations migrations:migrate 20180608161758
```

这条命令是恢复到20180608161758版本，除了直接输入具体版本号之外，还有更加方便的版本别名：

* **first**:回退到初始版本
* **prev**:回到上一个版本
* **next**:迁移到下一个版本
* **latest**:迁移到最新版本（和不加版本号效果一样）

## 常用命令

```bash
# 生成迁移脚本
php migration.php migrations:generate
# 执行迁移到最新版本
php migration.php migrations:migrate
# --dry-run是空转参数，只显示操作结果，不执行修改
php migration.php migrations:migrate --dry-run
# 不执行操作，只写入文件，对于生产环境需要手动验证并执行的场景有用
php migration.php migrations:migrate --write-sql=file.sql
# 查看详细信息
php migration.php status
```

## 结语

到此，就学会了migrations组件的基本使用，但是还有如下几个问题需要我们解决：

1. 一般项目中都会有数据库配置文件，如何让组件公用项目中的配置文件？
2. 现在迁移组件所需要的配置文件只能在根目录，如何让配置文件更合理的归档？
3. 命令行又长又难记，如何简单使用命令行？

[下一章](migrations-shu-ju-qian-yi-zu-jian-er-zi-ding-yi-ji-cheng-xiang-mu.md)，我们会解决以上问题，并且让组件更个性化的融入到我们自己的项目中去。



{% hint style="info" %}
在[我的代码库](https://github.com/tbphp/studyMigrations/tree/725659ad104700a2d4d4fbbcf26b5a6c5498de83)可以查看这篇文章的详细代码，欢迎star。
{% endhint %}

