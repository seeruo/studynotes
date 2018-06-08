# IO

## `File`

操作文件，兼容任何系统。

### 静态变量

```java
File.File.pathSeparator; // 路径分隔符  ':'
File.separator; // 目录名称分隔符  '/'
```

### 过滤文件

```java
// 显示java后缀的文件
File[] fs = new File("/www/java").listFiles(name -> name.getName().endsWith(".java"));
if (fs != null) {
    for (File f : fs) {
        System.out.println(f);
    }
}
```

## `OutputStream`

字节输出流，写文件示例：

```java
try {
    FileOutputStream fos = new FileOutputStream("a.txt");
    fos.write("Hello World!".getBytes());
    fos.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

## `InputStream`

字节输入流，读文件示例：

```java
try {
    FileInputStream fis = new FileInputStream("a.txt");
    String ret = new String(fis.readAllBytes());
    fis.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

## `FileWriter`

字符输出流，仅限文本。

```java
try {
    FileWriter fw = new FileWriter("a.txt");
    fw.write("Hello World!");
    fw.flush(); // 尽量每次写之后都flush一次
    fw.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

## `FileReader`

字符输入流，仅限文本。

```java
private static String readAllChars(String file, int step) {
    try {
        FileReader fr = new FileReader(file);
        char[] ch = new char[step];
        int length;
        StringBuilder str = new StringBuilder();
        while ((length = fr.read(ch)) != -1) {
            str.append(new String(ch, 0, length));
        }
        return str.toString();
    } catch (IOException e) {
        return "N/A";
    }
}
```

## 编码转换

```java
new InputStreamReader(new FileInputStream("a.txt"), "gbk"); // 读
new OutputStreamWriter(new FileOutputStream("a.txt"), "gbk"); // 写
```

## 缓冲流

```java
new BufferedOutputStream(new FileOutputStream("a.txt")); // 字节流，写
new BufferedInputStream(new FileInputStream("a.txt")); // 字节流，读
new BufferedWriter(new FileWriter("a.txt")); // 字符流，写
new BufferedReader(new FileReader("a.txt")); // 字符流，读
```

## `Properties`

读写数据：

```java
Properties pp = new Properties();
FileReader fr = new FileReader("a.properties");
pp.load(fr);
fr.close();
pp.setProperty("QQ", "332487886");
pp.setProperty("邮箱", "332487886@qq.com");
FileWriter fw = new FileWriter("a.properties");
pp.store(fw, "update info");
fw.close();
```

## 对象序列化

序列化目标对象必须实现`Serializable`接口。

静态不能序列化。

```java
// 序列化
ObjectOutputStream oo = new ObjectOutputStream(new FileOutputStream("a.txt"));
oo.writeObject(new Card("tb", 1));
oo.close();
// 反序列化
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("a.txt"));
Card p = (Card) ois.readObject();
System.out.println(p);
ois.close();
```

### `transient` 瞬态关键字

阻止成员变量序列化

## 打印流

> `PrintStream` `PrintWriter`
>
> 不浮躁数据源，只负责数据目的。
>
> 为其他输出流添加功能。

```java
PrintWriter pw = new PrintWriter("a.txt");
pw.println("Hello World!");
pw.close();
```

### 打印流开启自动刷新

**条件**

1. 输出对象必须是流对象。\(`OutputStream`,`Writer`\)。
2. 必须调用`println`,`printf`,`format`方法。

```java
new PrintWriter(new FileWriter("a.txt"), true);
```

