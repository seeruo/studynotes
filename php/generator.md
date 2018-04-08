# 协程二：迭代生成器



> 生成器提供了一种更容易的方法来实现简单的对象迭代，相比较定义类实现 Iterator 接口的方式，性能开销和复杂性大大降低。
>
> `yield`的理解：yield关键字暂停当前方法，且返回值给上一级处理。当上一级处理完毕继续遍历时，继续冲上次暂停的地方继续执行。



以改写`range`函数为例。使用`yield`关键字创建生成器接口来迭代对象，比使用`range`保存数组再遍历，内存对比：

**生成器：**

```php
<?php
/**
 * 生成器
 *
 * @author tangbo<admin@tbphp.net>
 */

namespace App;

use Generator;

class RangeGenerator
{
    private $len = 0;

    public function __construct(int $len)
    {
        $this->len = $len;
    }

    public function range(): Generator
    {
        for ($i = 0; $i <= $this->len; $i++) {
            yield $i;
        }
    }
}
```

测试：

```php
<?php
use App\RangeGenerator;
use App\RangeIterator;
use PHPUnit\Framework\TestCase;

class RangeGeneratorTest extends TestCase
{

    private $size = 50000;

    /**
     * 测试原始range方法
     */
    public function testOldrange(): void
    {
        $this->result(memory_get_usage(), range(0, $this->size));
    }

    /**
     * 测试yield关键字生成器方法
     */
    public function testRange(): void
    {
        $this->result(memory_get_usage(), (new RangeGenerator($this->size))->range());
    }

    private function result($start, $range): void
    {
        foreach ($range as $v) {
            echo $v;
        }
        echo PHP_EOL;
        echo '内存：' . (memory_get_usage() - $start) . PHP_EOL;
    }
}
```

经过对比得知，生成器方式来迭代内存小很多，特别是长度越长越明显。原因是使用`range`函数是在遍历之前，就先把目标长度的值保存为一个数组，数组的大小和目标长度成正比。但是生成器是在遍历时，再生成需要的数值。

我们可以使用迭代器来自己实现这一过程：

```php
<?php
/**
 * Description
 *
 * @author tangbo<admin@tbphp.net>
 */

namespace App;

use Iterator;

class RangeIterator implements Iterator
{

    private $size;
    private $key;

    public function __construct($size)
    {
        $this->size = $size;
    }

    public function current()
    {
        return $this->key;
    }

    public function next()
    {
        $this->key++;
    }

    public function key()
    {
        return $this->key;
    }

    public function valid()
    {
        return $this->key <= $this->size;
    }

    public function rewind()
    {
        $this->key = 0;
    }
}
```

在之前的测试类里面加入新的测试代码：

```php
/**
 * 测试迭代器
 */
public function testIrange(): void
{
    $this->result(memory_get_usage(), new RangeIterator($this->size));
}
```

对比该结果，使用迭代器跟生成器内存差不多。不过使用`yield`关键字的生成器使用更简单，性能也更高，推荐使用。

## return

使用yield同时也可以使用return返回值。

返回值需要调用getReturn方法获取，并且要在所有yield遍完之后才能获取返回值，否则会报错。