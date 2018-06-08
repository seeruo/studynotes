# git

## 全局配置

```bash
#全局配置格式
git config --global 命令 参数
user.name        #配置用户名
user.email        #配置邮箱
core.autocrlf    #禁用自动转换(true|false)--win平台下需要配置，避免add操作时替换换行符
color.ui        #git提示显示颜色(true|false)
```

### 别名配置

```bash
#新建别名
git config --global alias.[别名] [命令名]
#删除别名
git config --global --unset alias.[别名]
#查看别名
git config --get-regexp alias
```

### 个人别名配置记录

```bash
git config --global alias.cf "config --global"
git cf alias.s "status -s"
git cf alias.a add
git cf alias.b branch
git cf alias.t tag
git cf alias.l "log --pretty=oneline"
git cf alias.c "commit -am"
git cf alias.co checkout
git cf alias.df diff
git cf alias.map "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

## 操作

```bash
#把这个目录变成Git可以管理的仓库-在要管理的目录下使用init
git init
#把需要监控的文件添加到git库
git add index.php
#提交
git commit -m "新增index.php文件"
#添加暂存并提交
git commit -am "新增index.php文件"
```

> `git commit -a`中，`-a`参数的意思是自动执行`add`操作，这样不用`add`就可以自动添加所有有改动的文件，当然，**新建的文件仍需要执行**`add`**操作**。

## 查看

```bash
#查看状态
git status
#查看变化(不写文件名即为查看所有变化的文件)
git diff index.php
#与上个版本比较变化
git diff HEAD^ -- index.php
#查看现在时间线的历史记录(pretty为显示模式)
git log --pretty=oneline
#图形方式查看分支情况
git log --graph --pretty=oneline --abbrev-commit
#查看完整记录
git reflog
#查看远程库信息
git remote -v
```

> `diff`是把现在的文件和暂存文件做比较，如果已经`add`了，就不会显示。
>
> ## 返回历史
>
> ```bash
> git reset --hard [commitID]
> ```
>
> commitID:
>
> * `HEAD^` 上一个
> * `HEAD^^`  上上一个
> * `HEAD~10` 前第10个
>
> commitID除了使用`HEAD`表示以外，还可以使用commit id标识，不需要写完整。这样能回到后面的版本。

## 撤销修改

```bash
#删除暂存区文件
git reset HEAD index.php
#撤销工作区修改
git checkout -- index.php
```

> 如果已经进行了`add`操作，就需要先删除暂存区的文件，再撤销修改

## 删除文件

```bash
git rm index.php
```

> 删除的文件，同样可以执行撤销、回到历史版本等操作。

## 远程操作

#### github `SSH KEY` 验证

1. 创建`SSH KEY`

   ```bash
   ssh-keygen -t rsa -C "admin@tbphp.net"
   ```

2. 登录github，打开个人设置-&gt;SSH keys，把`id_rsa.pub`文件的内容添加进去。

#### 关联远程库

```bash
git remote add origin git@github.com:tbphp/test.git
```

#### 推送本地到服务器

```bash
git push -u origin master
```

> `-u`参数在第一次使用的时候关联远程服务器的`origin`主机下的`master`分支，以后直接使用`git push`即可。

#### 从服务器克隆到本地

```bash
#克隆到当前目录的/test/文件夹下
git clone git@github.com:tbphp/test.git
#指定克隆目录，而不是test文件夹
git clone git@github.com:tbphp/test.git  E:/www/git/
```

> 克隆的时候，会自动创建项目文件夹，所以不用手动创建

## 分支

### 创建分支

```bash
#创建test1分支
git branch test1
#切换到test1分支
git checkout test1
```

上面两行代码可以缩短为下面：

```bash
#创建并切换到test1分支
git checkout -b test1
```

> `git branch`命令如果后面不接参数，为**显示当前分支**，如果后面接分支名，则为**创建分支**。

### 合并分支

```bash
#把test1分支合并到当前分支
git merge --no-ff test1
```

> `--no-ff`参数使用普通模式合并，这样能看到合并的历史记录。

### 删除分支

```bash
#删除test1分支
git branch -d test1
```

### 分支策略

1. `master`分支应该是非常稳定的，仅用来发布新版本，平时不能在上面干活；
2. 干活都在`dev`分支上，`dev`分支是不稳定的，把dev分支合并到master上，在master分支发布；
3. 其他人都有自己的分支，频繁的往`dev`上合并；
4. 修复bug时，创建新的bug分支进行修复，然后合并，最后删除bug分支；
5. 每一个新功能，都添加一个feature分支；
6. but分支和feature分支通常作为本地库自己私有，是否需要删除，看个人需要。以及是否需要`push`看是否需要其他人一起完成。

### `stash`

> 没有写完的代码，需要临时切换分支，却又不想提交未完成的代码，就需要用到临时存储功能。

```bash
#临时存储分支
git stash
#查看分支
git stash list
#提取存储的分支，并删除临时文件
git stash pop
```

> **注意：**经测试，如果新文件没有添加到暂存区，是不会被`stash`命令存储的，所以新建的文件，必须先执行`git add newfilename`。

## 多人协作

1. 首先，可以试图用`git push origin branch-name`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin branch-name`推送就能成功！
5. 如果git pull提示_“no tracking information”_，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream branch-name origin/branch-name`。

## 标签管理

```bash
#创建标签
git tag v1.0
#给历史版本创建标签
git tag v0.9 commitID
#给标签添加说明
git tag -a v1.1 -m "v1.1版本的说明"
#查看标签列表
git tag
#查看某个标签信息
git show tagname
#删除标签
git tag -d tagname
#推送某个tag
git push origin tagname
#推送全部未发布tag
git push origin --tags
```

## 忽略文件

创建`.gitignore`文件，每一行写一个需要忽略的文件或目录

> **注意：**如果已经提交过的文件，添加忽略不生效，必须在暂存区删除要忽略的文件，再提交。如下：

```bash
#忽略a.html文件
git rm -r --cached a.html
git commit -m "忽略a.html文件"
```

> 如果忽略的文件多的话，可以清空暂存区，在重新添加。如下：

```bash
git rm -r --cached .
git add .
git commit -m "配置忽略文件"
```

