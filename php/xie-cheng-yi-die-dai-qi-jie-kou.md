# 协程一：迭代器接口

> 实现迭代器接口，以实现自定义对象迭代。

## 代码

员工类：

```php
<?php
/**
 * 员工类
 *
 * @author tangbo<admin@tbphp.net>
 */

namespace App;

class Employee
{
    private $number;
    private $name;
    private $long;

    /**
     * Employee constructor.
     *
     * @param $name
     * @param $long
     * @param $work
     */
    public function __construct($number, $name, $long)
    {
        $this->number = $number;
        $this->name   = $name;
        $this->long   = $long;
    }

    public function getValue(): string
    {
        return '工号：' . $this->number . ' 姓名：' . $this->name . ' 年限：' . $this->long;
    }

}
```

**员工迭代器**：

```php
<?php
/**
 * 员工迭代器
 *
 * @author tangbo<admin@tbphp.net>
 */

namespace App;

use function array_push;
use Iterator;

class EmployeeIterator implements Iterator
{

    private $employees;
    private $key = 0;

    public function __construct()
    {
        $this->employees = [];
        $this->key       = 0;
    }

    public function push(Employee $e): int
    {
        return array_push($this->employees, $e);
    }

    public function current(): string
    {
        return $this->employees[$this->key]->getValue();
    }

    public function next(): void
    {
        $this->key++;
    }

    public function key(): int
    {
        return $this->key;
    }

    public function valid(): bool
    {
        return isset($this->employees[$this->key]);
    }

    public function rewind(): void
    {
        $this->key = 0;
    }
}
```

测试类：

```php
<?php
use App\Employee;
use App\EmployeeIterator;
use PHPUnit\Framework\TestCase;

class EmployeeIteratorTest extends TestCase
{
    private $obj;

    protected function setUp(): void
    {
        $this->obj = new EmployeeIterator();
        $this->obj->push(new Employee('001', 'tb', 5));
        $this->obj->push(new Employee('002', 'zx', 3));
        $this->obj->push(new Employee('003', 'tx', 7));
    }

    public function testIterator(): void
    {
        foreach ($this->obj as $k => $v) {
            echo '第：' . ($k + 1) . '行：' . $v . PHP_EOL;
        }
        self::assertInstanceOf(Iterator::class, $this->obj);
    }

}
```

