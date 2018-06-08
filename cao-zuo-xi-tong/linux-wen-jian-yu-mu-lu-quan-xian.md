# \[Linux\] 文件与目录权限

## 文件属性

```bash
-rw-r-xr-x.    1   root    root      4479     10月 31 2014   wgetrc
[   权限   ] [连结][拥有者] [群组]   [文件大小]  [  修改日期  ] [文件名称]
```

## 权限说明

### 权限值

| 权限名 | 全民 | 文件 | 目录 |
| :---: | :---: | :---: | :---: |
| **-** | none | 无 | 无 |
| **r** | read | 读取 | 读取目录结构列表 |
| **w** | write | 写入 | 删除、重命名、新建、移动该目录或目录下的文件 |
| **x** | execute | 执行 | 成为工作目录（允许进入目录） |

> **注意：**
>
> * 目录和文件的权限意义并不一样！
> * web网站的目录必须（或者只）要拥有`x`权限，文件必须（或者只）要拥有`r`权限。
> * 文件是否可以删除、重命名、移动取决于上级目录是否有`w`权限。

### 权限属性

```bash
   -   r   w   -   r   -   x   r   -   x    .
  [1] [2] [3] [4] [5] [6] [7] [8] [9] [10] [11]
```

* \[1\]：文档属性
  * _d_：目录
  * _-_：文件
  * _l_：连接档
  * _b_：装置文件里面的可供储存的接口设备\(可随机存取装置\)
  * _c_：装置文件里面的串行端口设备，例如键盘、鼠标\(一次性读取装置\)
* \[2-4\]：拥有者权限
* \[5-7\]：群组权限
* \[8-10\]：其他人权限
* \[11\]：ACL类型 \(不明待定\)  

  **权限分值**

  | 分值 | _0_ | _1_ | _2_ | _3_ | _4_ | _5_ | _6_ | _7_ |
  | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
  | 权限 | - | x | w | wx | r | rx | rw | rwx |
  | 说明 | 无权限 | **执行** | **写入** | 写入和执行 | **读取** | 读取和执行 | 读取和写入 | 所有权限 |

## 相关命令

### ls

> 即`list`的意思，用于显示文件和相关属性。

#### 参数说明：

* `-a`，`--all`：显示所有项目
* `-l`：显示文件完整信息
* `--full-time`：显示完整时间

  **常用命令：**

* `ll`：即ls -l，显示文件完整信息
* `ls -al`：显示所有文件（含隐藏文件）的文件完整信息

### chgrp,chown,chmod

#### chgrp

> 即`change group`的意思，用于改变文件所属群组。

用法：chgrp 群组名 文件名 例子：

```bash
chgrp root /home/test/text.txt
#改变text.txt文件的归属组为root组
```

#### chown

> 即`change owner`的意思，用于改变文件拥有者。

用法：chown 账号名\[:群组名\] 文件名 例子：

```bash
chown root /home/test/text.txt
#改变text.txt文件的拥有者为root
chown root:root /home/test/text.txt
#改变text.txt文件的拥有者为root,群组为root组
```

**注意**：由于复制行为\(cp\)会复制执行者的属性与权限，所以当复制文件之后，需要修改文件的拥有者于群组，便于用户使用。

#### chmod

> 即`change mode`的意思，用于改变文件的存取模式（改变权限）。

用法：chmod 权限值 文件名 例子：

```bash
chmode 770 /home/test/text.txt
chmode u=rwx,g=rwx,o=- /home/test/text.txt
#上面两个效果一样，均为改变text.txt文件的权限为拥有者和群组都有所有权限，其他人没有权限
```

#### 常用参数

> `-R`或`--recursive`，表示递归

例子：

```bash
chgrp -R root /home/test/
#递归改变text目录及下面所有内容的归属组
chown -R root /home/test/
#递归改变text目录及下面所有内容的拥有者
chmode 770 /home/test/text.txt
#递归改变text目录及下面所有内容的权限
```
