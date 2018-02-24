# API



## `equals`

Object.equals()用于比较对象内存地址。

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

>   优先使用`StringBuilder`，线程不安全，但是效率更高。