# git服务器搭建

## Windows环境

使用`gitblit`搭建windowns环境的git服务器

### 准备工作

1. 下载`gitblit`，官网：[gitblit.com](http://gitblit.com/)。
2. 下载`java`，并安装。官网：[java.com](https://www.java.com)。

### 基本配置

修改配置文件：`gitblit\data\gitblit.properties`

```bash
#git库集中存放的根目录
git.repositoriesFolder = E:\git
#web管理端口
server.httpPort = 8080
#web管理IP
server.httpBindInterface = 192.168.1.100
```

### 安装服务

修改安装文件：`gitblit\installService.cmd`

```bash
#根据系统环境修改为x86,amd64,或ia32
SET ARCH=x86
#设置CD变量的值为gitblit的安装位置
SET CD=D:\gitblit
#修改StopParams参数
–StopParams="..." ^ 改为 –StopParams=””
```

安装好后，访问[http://127.0.0.1:8080进行管理，默认用户名和密码都是：\`admin\`。](http://127.0.0.1:8080进行管理，默认用户名和密码都是：`admin`。)

> **注意：**执行安装时，必须以管理员身份运行。 64位的系统，ARCH本应该设置为amd64，但是安装时出错，改成x86就行了，暂时不知道原因。

### 自动部署

新建`autodeploy.groovy`文件，并写入以下代码：

```groovy
import com.gitblit.GitBlit
import com.gitblit.Keys
import com.gitblit.models.RepositoryModel
import com.gitblit.models.TeamModel
import com.gitblit.models.UserModel
import com.gitblit.utils.JGitUtils
import com.gitblit.utils.StringUtils
import java.text.SimpleDateFormat
import org.eclipse.jgit.api.CloneCommand
import org.eclipse.jgit.api.PullCommand
import org.eclipse.jgit.api.Git
import org.eclipse.jgit.lib.Repository
import org.eclipse.jgit.lib.RepositoryBuilder
import org.eclipse.jgit.internal.storage.file.FileRepository
import org.eclipse.jgit.lib.Config
import org.eclipse.jgit.revwalk.RevCommit
import org.eclipse.jgit.transport.ReceiveCommand
import org.eclipse.jgit.transport.ReceiveCommand.Result
import org.eclipse.jgit.util.FileUtils
import org.slf4j.Logger

logger.info("autodeploy hook triggered by ${user.username} for ${repository.name}")

// 目标根目录(不能以'/'结尾)
def rootFolder = 'E:/www'
// 项目名
def repoName = repository.name
// 目标部署目录
def repoFolder=StringUtils.stripDotGit(repoName)
// 项目路径
def destinationFolder = new File(rootFolder, repoFolder)

// 判断项目路径是否存在，存在就pull，不存在就clone
if (destinationFolder.exists()) {
    // 目标项目已存在，执行pull操作

    logger.info("Pulling ${repoName} to ${destinationFolder}")
    // 目标git目录
    def gitpath = new File( rootFolder + '/' + repoFolder + '/.git' )
    FileRepository repo = new FileRepository(gitpath)
    Git git = new Git(repo)
    PullCommand pullCmd = git.pull()
    pullCmd.call()
    git.repository.close()

    clientLogger.info("项目部署成功：${repoName} >> ${destinationFolder}")
} else {
    // 目标项目不存在，执行clone操作

    def srcUrl = 'file://' + new File(gitblit.getRepositoriesFolder(), repoName).absolutePath
    logger.info("Cloning ${srcUrl} to ${destinationFolder}")
    def bare = false
    def cloneBranch = 'refs/heads/master'
    def includeSubmodules = true
    CloneCommand cmd = Git.cloneRepository()
    cmd.setBare(bare)
    cmd.setBranch(cloneBranch)
    cmd.setCloneSubmodules(includeSubmodules)
    cmd.setURI(srcUrl)
    cmd.setDirectory(destinationFolder)
    Git git = cmd.call()
    git.repository.close()
    clientLogger.info("项目克隆成功：${repoName} >> ${destinationFolder}")
}
```

服务器端，不要手动创建项目或者clone，必须通过自动部署脚本自动`clone`，这样以后才能`pull`，不然容易出现`UnknownHostKey`的错误提示。

## Linux环境

