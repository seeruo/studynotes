# 面向对象



## 类与对象

类是抽象定义的类，对象是把类具体实例化之后。

即new出来的就是对象。



## 成员属性

所有属性必须私有化，并且提供set和get方法，例：

```java
public class Test {
    private int attr;

    public void setAttr(int a)
    {
        attr = a;
    }

    public int getAttr()
    {
        return attr;
    }
}
```



## 成员属性常量

常量使用`final`关键字，且不可被改变。

```java
public class Test {
    public final int a = 1;
}
```





## `this`关键字

实例化对象的变量可以用this.变量调用，静态变量直接用类名.变量名调用。



## 继承

### 变量

变量没有重写，子类和父类变量同时存在，只是调用变量的规则是取最近。局部变量->子类成员属性->父类成员属性。

子类里面调用父类变量使用super关键字：

```java
public class Father {
    int v = 1;
}

public class Son extends Father {
    int v = 2;

    public void print() {
        System.out.println(v); // 输出Son.v
        System.out.println(super.v); // 输出Father.v
    }
}
```

### 方法

子类方法可以重写父类方法，调用规则和变量一致，同样调用父类方法用super关键字。

**方法重写时，子类的方法权限必须大于等于父类方法！**



## 抽象类

**`abstract` 关键字**

>   定义抽象类，规定需要申明的方法，继承抽象类时，必须实现抽象类的所定义的抽象方法;
>
>   抽象类不能直接使用，不能new，只能被继承。

定义抽象类：

```java
public abstract class Base {
    public abstract void work();
    public abstract int sleep();
}
```

继承抽象类：

```java
public abstract class Worker extends Base {
    public void work() {
        // do work
    }
    
    public int sleep() {
        return 1;
    }
}
```

### 抽象类的特殊使用方法

>   抽象类可以不申明抽象方法。
>
>   可以定义普通方法，以保证子类继承之后直接可用。
>
>   也可以定义静态方法，以直接调用。
>
>   这样做的目的是为了可以使用，但是不能new对象。



## 接口

**`interface` 关键字**

**`implements` 关键字**

>   **更严格的抽象类**，只能定义抽象方法，没有类似'抽象类的特殊使用方法'。
>
>   接口成员变量只能定义常量。

定义接口：

```java
public interface Pay {
    
    public static final int PAY_TYPE = 1;
	
	public abstract void check();
	
	public abstract void done();
}
```

实现接口：

```java
public class AliPay implements Pay {

	public void check() {

	}

	public void done() {

	}
}
```

### 接口里面的修饰符

接口里面的属性或者方法的修饰符都可以省略，且写与不写都没区别。

接口里面的成员变量，有没有修饰符都是公共静态常量。

接口里面的方法，有没有修饰符都是公共抽象方法。

例如:

```java
public interface Pay {

	int PAY_TYPE = 1;

	void check();

	void done();
}
```

和：

```java
public interface Pay {
    
    public static final int PAY_TYPE = 1;
	
	public abstract void check();
	
	public abstract void done();
}
```

是一样的。

但是实现接口的类里面，必须加上修饰符，不能省略。

**但是编码规范建议写上修饰符，避免混淆。**

### 接口的多实现、多继承

类可以多实现接口

```java
public class Ali implements Pay,Work {
    
}
```

子接口也可以多继承父接口

```java
public interface A extends B,C {
    
}
```



## 多态

使用父类型，实例化子类，实现多态性。

自动类型提升。

父类 a = new 子类();

a.fun();// **此处调用的是子类重写的方法，不是父类**

a.var;// 此处调用的是父类的var变量，跟方法不同

父类方法必须存在，如果子类有重写，则调用子类重写方法。如果子类有，但是父类不存在，则报错。

**如果是静态方法，则调用父方法**

### 向下转型

多态默认向上转型，如果要向下转型，则需要强制转换，例：

```java
P a = new S();
S b = (S)a; // 强制向下转换
```



## `instanceof` 关键字

用于比较引用数据类型

子类 a = new 子类();

a instanceof 父类 // true

a instanceof 子类 // true



## 构造方法

### 格式

方法名和类名一致，不写返回值。

权限 方法(参数) {

}

```java
public class Post {
    public int a;
    public Post(int a) {
        this.a = a;
    }
}
```

### this super

构造方法里面使用this调用其他构造方法必须在第一行:

```java
public class Post {

    public int a;

    public Post() {
        // this之前不能有其他代码
        this(123);
    }

    public Post(int a) {
        this.a = a;
    }
}
```

super调用父类的构造方法，痛this一样，必须在第一行：

```java
public class S extends P {

    public int a;

    public S() {
        // this之前不能有其他代码
        super(123);
    }
}
```

>   **注意**：继承时，子类所有构造方法里面都有一行隐式的super()代码，所以都会调用父类的空构造方法。



## 访问修饰符

|                | public | protected | default | private |
|:------------:| :----: | :-------: | :-----: | :-----: |
|     当前类     |   √    |     √     |    √    |    √    |
|     当前包     |   √    |     √     |    √    |    -    |
|  不同包的子类   |   √    |     √     |    -    |    -    |
| 其他包的其他类  |   √    |     -     |    -    |    -    |

>   注意：default为不写修饰符。

## 可变参方法
用于方法参数数量不确定，参数必须是同一类型，且传入的多个变量在方法里面识别为数组。例：
```java
private static void testFun(int... b) {
    for (int i : b) {
        System.out.println(i);
    }
}

// 调用
testFun(1, 2, 3, 4, 5);
```

还可以和其他参数一起，但是可变参数必须在最后，且只能有一个。例：
```java
private static void testFun(String a, System b, int... b) {
    for (int i : b) {
        System.out.println(i);
    }
}
```


## 反射

获取Class对象：

```java
// 方法一：
Class cls = new User("tb", 12, "男").getClass();
// 方法二：
Class cls = User.class;
// 方法三
Class cls = Class.forName("study.fanshe.User");
```

> **注意**：方法二虽然简单，但不会把类加载到内存里，导致静态代码块不能执行，所以根据情况使用，常用第三种方法。

反射的方式执行构造方法实例化对象：

```java
Class<User> cls = User.class;
Constructor<User> ct = cls.getConstructor(String.class, int.class, String.class);
User user = ct.newInstance("tb", 12, "男");
```

泛型擦除：

通过反射，可以绕开编译之前的泛型约束。