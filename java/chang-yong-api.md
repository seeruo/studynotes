# 常用API

## `equals`

Object.equals\(\)用于比较对象内存地址。

因此类里面常常重写此方法，用于自定比较规则。

重写示例：

```java
public class Test {

    public int a;
    public int b;

    public boolean equals(Object anObj) {
        if (this == anObj) {
            return true;
        }
        if (anObj instanceof Test) {
            Test obj = (Test) anObj;
            return obj.a == a && obj.b == b;
        }
        return false;
    }

}
```

## `StringBuffer`

比`String`节省内存。

> 优先使用`StringBuilder`，线程不安全，但是效率更高。

## Date

* `System.currentTimeMillis()`获取当前时间戳（毫秒）

### 格式化时间戳

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
sdf.format(new Date());
```

格式化字母语义：

| 字母 | 日期或时间元素 | 示例 |
| --- | --- | --- |
| `G` | Era 标志符 | `AD` |
| `y` | 年 | `1996`; `96` |
| `M` | 年中的月份 | `July`; `Jul`; `07` |
| `w` | 年中的周数 | `27` |
| `W` | 月份中的周数 | `2` |
| `D` | 年中的天数 | `189` |
| `d` | 月份中的天数 | `10` |
| `F` | 月份中的星期 | `2` |
| `E` | 星期中的天数 | `Tuesday`; `Tue` |
| `a` | Am/pm 标记 | `PM` |
| `H` | 一天中的小时数（0-23） | `0` |
| `k` | 一天中的小时数（1-24） | `24` |
| `K` | am/pm 中的小时数（0-11） | `0` |
| `h` | am/pm 中的小时数（1-12） | `12` |
| `m` | 小时中的分钟数 | `30` |
| `s` | 分钟中的秒数 | `55` |
| `S` | 毫秒数 | `978` |
| `z` | 时区 | `Pacific Standard Time`; `PST`; `GMT-08:00` |
| `Z` | 时区 | `-0800` |

### 解析字符串为日期

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm");
try {
    sdf.parse("2015-01-01 8:00");
} catch (Exception e) {
    System.err.println(e.getMessage());
}
```

